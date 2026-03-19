# KPIs Databox

> Indicateurs de performance avec formules SQL complètes, validés par exécution sur la base.
> S'appuie sur les définitions du glossaire métier (glossaire.md).

## Vue d'ensemble

| KPI | Source glossaire | Table(s) principale(s) |
|-----|------------------|----------------------|
| Évolution CA facturé | CA facturé | `invoices` |
| Évolution CA commandé | CA commandé | `sales_orders` |
| Taux de conversion devis→commandes | Devis converti, Taux de conversion | `quotes` |
| Marge brute | Marge | `invoices`, `invoices_lines` |
| Panier moyen | Panier moyen | `sales_orders` |
| Top N clients par CA | CA facturé, Commercial | `invoices`, `accounts` |
| Délai moyen de paiement | Délai de paiement | `invoices`, `open_items` |

---

## Détails par KPI

### Évolution CA facturé

- **Description :** Suivi du chiffre d'affaires facturé HT dans le temps, avoirs déduits, avec gestion des périodes sans données
- **Source glossaire :** CA facturé
- **Paramètres :**
  - `date_debut` / `date_fin` : bornes de la période
  - `granularité` : `month` | `quarter` | `year`
- **Formule SQL :**
  ```sql
  SELECT
    periods.period,
    COALESCE(COUNT(DISTINCT i.invoices_id_databox), 0) AS nb_factures,
    COALESCE(SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign), 0) AS ca_facture_ht
  FROM generate_series(
    DATE_TRUNC('month', :date_debut::timestamp),
    DATE_TRUNC('month', :date_fin::timestamp),
    '1 month'::interval  -- adapter selon granularité
  ) AS periods(period)
  LEFT JOIN databox.invoices i
    ON DATE_TRUNC('month', i.invoices_invoice_date) = periods.period
  LEFT JOIN databox.id_mapping im
    ON i.invoices_id_databox = im.id_databox
    AND im.end_date IS NULL
  GROUP BY periods.period
  ORDER BY periods.period ASC;
  ```
- **Exemple de résultat (2024) :**

  | period | nb_factures | ca_facture_ht |
  |--------|-------------|---------------|
  | 2024-01 | 3 | 1 073,33 |
  | 2024-02 | 5 | 1 252,00 |
  | 2024-03 | 9 | 389,20 |
  | 2024-04 | 0 | 0,00 |

- **Règles Databox appliquées :** `end_date IS NULL`, `* invoices_sign`, `generate_series` pour les périodes vides
- **Variantes :** Remplacer `'1 month'` par `'3 months'` (trimestre) ou `'1 year'` (année), et adapter `DATE_TRUNC` en conséquence

### Évolution CA commandé

- **Description :** Suivi du chiffre d'affaires commandé HT dans le temps, hors commandes annulées
- **Source glossaire :** CA commandé
- **Paramètres :**
  - `date_debut` / `date_fin` : bornes de la période
  - `granularité` : `month` | `quarter` | `year`
- **Formule SQL :**
  ```sql
  SELECT
    periods.period,
    COALESCE(COUNT(DISTINCT so.sales_orders_id_databox), 0) AS nb_commandes,
    COALESCE(SUM(so.sales_orders_amount_lines_excl_vat), 0) AS ca_commande_ht
  FROM generate_series(
    DATE_TRUNC('quarter', :date_debut::timestamp),
    DATE_TRUNC('quarter', :date_fin::timestamp),
    '3 months'::interval  -- adapter selon granularité
  ) AS periods(period)
  LEFT JOIN databox.sales_orders so
    ON DATE_TRUNC('quarter', so.sales_orders_order_date) = periods.period
    AND so.sales_orders_order_status IN (1, 2)
  LEFT JOIN databox.id_mapping im
    ON so.sales_orders_id_databox = im.id_databox
    AND im.end_date IS NULL
  GROUP BY periods.period
  ORDER BY periods.period ASC;
  ```
- **Exemple de résultat (2024, trimestriel) :**

  | period | nb_commandes | ca_commande_ht |
  |--------|-------------|---------------|
  | 2024-Q1 | 72 | 20 811,34 |
  | 2024-Q2 | 76 | 37 042,90 |
  | 2024-Q3 | 72 | 44 858,45 |
  | 2024-Q4 | 57 | 18 469,87 |

- **Règles Databox appliquées :** `end_date IS NULL`, `order_status IN (1, 2)`, `generate_series`

### Taux de conversion devis→commandes

- **Description :** Pourcentage de devis ayant donné lieu à une commande, ventilable par commercial et par période
- **Source glossaire :** Devis converti, Taux de conversion
- **Paramètres :**
  - `date_debut` / `date_fin` : période des devis
  - `par_commercial` : ventilation optionnelle par commercial
- **Formule SQL (global) :**
  ```sql
  SELECT
    COUNT(*) AS total_devis,
    COUNT(*) FILTER (WHERE q.quotes_quote_status IN ('2', '3')) AS devis_convertis,
    ROUND(100.0 * COUNT(*) FILTER (WHERE q.quotes_quote_status IN ('2', '3'))
      / NULLIF(COUNT(*), 0), 2) AS taux_conversion_pct
  FROM databox.quotes q
  INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND q.quotes_quote_date >= :date_debut
    AND q.quotes_quote_date < :date_fin;
  ```
- **Formule SQL (par commercial) :**
  ```sql
  SELECT
    sr.sales_reps_name AS commercial,
    COUNT(*) AS total_devis,
    COUNT(*) FILTER (WHERE q.quotes_quote_status IN ('2', '3')) AS devis_convertis,
    ROUND(100.0 * COUNT(*) FILTER (WHERE q.quotes_quote_status IN ('2', '3'))
      / NULLIF(COUNT(*), 0), 2) AS taux_conversion_pct
  FROM databox.quotes q
  INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
  LEFT JOIN databox.id_mapping rep_im
    ON q.quotes_idcor_sales_rep = rep_im.id_correspondence
    AND rep_im.end_date IS NULL
  LEFT JOIN databox.sales_reps sr ON rep_im.id_databox = sr.sales_reps_id_databox
  WHERE im.end_date IS NULL
    AND q.quotes_quote_date >= :date_debut
    AND q.quotes_quote_date < :date_fin
  GROUP BY sr.sales_reps_name
  ORDER BY total_devis DESC;
  ```
- **Exemple de résultat (2024) :**

  | total_devis | devis_convertis | taux_conversion_pct |
  |-------------|-----------------|---------------------|
  | 27 | 13 | 48,15% |

- **Règles Databox appliquées :** `end_date IS NULL`, résolution idcor_sales_rep avec alias `rep_im`, commandes directes exclues

### Marge brute

- **Description :** Marge brute en valeur absolue et en pourcentage du CA, sur factures ou commandes
- **Source glossaire :** Marge
- **Paramètres :**
  - `date_debut` / `date_fin` : période
  - `contexte` : facturé (invoices_lines) | commandé (sales_orders)
- **Formule SQL (facturé) :**
  ```sql
  SELECT
    SUM(il.invoices_lines_line_margin * i.invoices_sign) AS marge_absolue,
    SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_facture_ht,
    ROUND(100.0 * SUM(il.invoices_lines_line_margin * i.invoices_sign)
      / NULLIF(SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign), 0), 2) AS marge_pct
  FROM databox.invoices i
  INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
  INNER JOIN databox.invoices_lines il ON i.invoices_id_databox = il.invoices_id_databox
  WHERE im.end_date IS NULL
    AND i.invoices_invoice_date >= :date_debut
    AND i.invoices_invoice_date < :date_fin;
  ```
- **Formule SQL (commandé) :**
  ```sql
  SELECT
    SUM(so.sales_orders_total_margin) AS marge_absolue,
    SUM(so.sales_orders_amount_lines_excl_vat) AS ca_commande_ht,
    ROUND(100.0 * SUM(so.sales_orders_total_margin)
      / NULLIF(SUM(so.sales_orders_amount_lines_excl_vat), 0), 2) AS marge_pct
  FROM databox.sales_orders so
  INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND so.sales_orders_order_status IN (1, 2)
    AND so.sales_orders_order_date >= :date_debut
    AND so.sales_orders_order_date < :date_fin;
  ```
- **Exemple de résultat (facturé, 2024) :**

  | marge_absolue | ca_facture_ht | marge_pct |
  |---------------|---------------|-----------|
  | 9 403,28 | 48 466,61 | 19,40% |

- **Règles Databox appliquées :** `end_date IS NULL`, `* invoices_sign` (facturé), `order_status IN (1, 2)` (commandé)

### Panier moyen

- **Description :** Montant moyen HT par commande client
- **Source glossaire :** Panier moyen
- **Paramètres :**
  - `date_debut` / `date_fin` : période
- **Formule SQL :**
  ```sql
  SELECT
    COUNT(DISTINCT so.sales_orders_id_databox) AS nb_commandes,
    SUM(so.sales_orders_amount_lines_excl_vat) AS ca_commande_ht,
    ROUND(SUM(so.sales_orders_amount_lines_excl_vat)
      / NULLIF(COUNT(DISTINCT so.sales_orders_id_databox), 0), 2) AS panier_moyen
  FROM databox.sales_orders so
  INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND so.sales_orders_order_status IN (1, 2)
    AND so.sales_orders_order_date >= :date_debut
    AND so.sales_orders_order_date < :date_fin;
  ```
- **Exemple de résultat (2024) :**

  | nb_commandes | ca_commande_ht | panier_moyen |
  |--------------|----------------|--------------|
  | 272 | 120 498,31 | 443,01 |

- **Règles Databox appliquées :** `end_date IS NULL`, `order_status IN (1, 2)`

### Top N clients par CA

- **Description :** Classement des N meilleurs clients par chiffre d'affaires facturé
- **Source glossaire :** CA facturé, Commercial
- **Paramètres :**
  - `date_debut` / `date_fin` : période
  - `N` : nombre de clients à afficher (défaut 10)
- **Formule SQL :**
  ```sql
  SELECT
    a.accounts_account_code,
    a.accounts_company_name_1,
    COUNT(DISTINCT i.invoices_id_databox) AS nb_factures,
    SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_facture_ht
  FROM databox.invoices i
  INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
  LEFT JOIN databox.id_mapping acc_im
    ON i.invoices_idcor_account = acc_im.id_correspondence
    AND acc_im.end_date IS NULL
  LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
  WHERE im.end_date IS NULL
    AND i.invoices_invoice_date >= :date_debut
    AND i.invoices_invoice_date < :date_fin
  GROUP BY a.accounts_account_code, a.accounts_company_name_1
  ORDER BY ca_facture_ht DESC
  LIMIT :N;
  ```
- **Exemple de résultat (top 5, 2024) :**

  | accounts_account_code | accounts_company_name_1 | nb_factures | ca_facture_ht |
  |----------------------|------------------------|-------------|---------------|
  | ALCAN | ALCAN | 1 | 7 005,00 |
  | 000032 | VELOLAND CHOLET | 10 | 3 442,53 |
  | AERONEF | AERONEF | 4 | 2 050,00 |
  | XNS003 | XNS Suisse | 1 | 1 696,20 |
  | 000034 | INTERCYCLES | 1 | 210,00 |

- **Règles Databox appliquées :** `end_date IS NULL`, `* invoices_sign`, résolution idcor_account avec alias `acc_im`
- **Note :** Le nom de colonne correct est `accounts_company_name_1` (avec underscore avant le chiffre)

### Délai moyen de paiement

- **Description :** Nombre moyen de jours entre l'émission de la facture et le paiement effectif
- **Source glossaire :** Délai de paiement
- **Paramètres :**
  - `date_debut` / `date_fin` : période des factures
- **Formule SQL (délai réel) :**
  ```sql
  SELECT
    ROUND(AVG(EXTRACT(DAY FROM oi.open_items_payment_date - i.invoices_invoice_date)), 1) AS delai_moyen_jours,
    COUNT(*) AS nb_factures_payees
  FROM databox.open_items oi
  INNER JOIN databox.id_mapping oi_im ON oi.open_items_id_databox = oi_im.id_databox
  INNER JOIN databox.invoices i
    ON oi.open_items_document_number = i.invoices_invoice_code
  INNER JOIN databox.id_mapping i_im ON i.invoices_id_databox = i_im.id_databox
  WHERE oi_im.end_date IS NULL
    AND i_im.end_date IS NULL
    AND oi.open_items_payment_date IS NOT NULL
    AND i.invoices_invoice_date >= :date_debut
    AND i.invoices_invoice_date < :date_fin;
  ```
- **Exemple de résultat :** Non disponible (table `open_items` non alimentée dans le jeu de données actuel)
- **Règles Databox appliquées :** `end_date IS NULL` sur les deux id_mapping (aliases `oi_im` et `i_im`), jointure via `document_number = invoice_code`
- **Note :** La jointure `open_items → invoices` via `document_number` devra être vérifiée lorsque les données open_items seront disponibles. Le lien pourrait nécessiter un ajustement (idcor ou autre clé).
