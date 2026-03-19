# Glossaire métier Databox

> Source de vérité pour la traduction des termes business en SQL Databox.
> Chaque définition a été validée par l'utilisateur.

## Termes

| Terme | Définition métier | Table(s) | Colonne(s) clé(s) | Filtres SQL obligatoires |
|-------|-------------------|----------|--------------------|-----------------------|
| CA facturé | Montant total HT facturé, avoirs déduits | `invoices` | `invoices_amount_lines_excl_vat` | `end_date IS NULL`, `* invoices_sign` |
| CA commandé | Montant total HT des commandes, hors annulées | `sales_orders` | `sales_orders_amount_lines_excl_vat` | `end_date IS NULL`, `order_status IN (1, 2)` |
| Marge | Marge brute en valeur absolue et en % du CA | `invoices_lines` / `sales_orders` | `invoices_lines_line_margin`, `sales_orders_total_margin` | `end_date IS NULL`, `* invoices_sign` (facturé) |
| Client actif | Client ayant commandé dans les 12 derniers mois | `sales_orders`, `accounts` | `sales_orders_order_date` | `end_date IS NULL`, `order_status IN (1, 2)` |
| Client inactif | Client avec historique mais sans commande depuis 12 mois, ou `accounts_is_active = false` | `sales_orders`, `accounts` | `accounts_is_active`, `sales_orders_order_date` | `end_date IS NULL` |
| Devis converti | Devis ayant donné lieu à une commande | `quotes` | `quotes_quote_status` | `end_date IS NULL`, `status IN ('2', '3')` |
| Avoir | Facture avec signe négatif | `invoices` | `invoices_sign` | `invoices_sign = -1` |
| Encours client | Montant restant dû par un client | `open_items` | `open_items_amount_in_company_currency`, `open_items_paid_amount_in_company_currency` | `end_date IS NULL`, `* open_items_sign` |
| Taux de conversion | Ratio devis convertis / total devis | `quotes` | `quotes_quote_status` | `end_date IS NULL`, commandes directes exclues |
| Panier moyen | Montant moyen par commande | `sales_orders` | `sales_orders_amount_lines_excl_vat` | `end_date IS NULL`, `order_status IN (1, 2)` |
| Délai de paiement | Jours entre facture et paiement | `invoices`, `open_items` | `invoices_invoice_date`, `open_items_payment_date` | `end_date IS NULL` |
| Commande en retard | Ligne de commande non livrée après la date demandée | `sales_orders_lines` | `asked_delivery_date`, `quantity_to_deliver` | `end_date IS NULL`, `quantity_to_deliver > 0` |
| Commercial | Représentant commercial lié aux documents | `sales_reps` | `sales_reps_rep_code`, `sales_reps_name` | Résolution via `idcor_sales_rep` + `id_mapping` |

## Détails par terme

### CA facturé
- **Définition :** Montant total HT des factures émises, avoirs déduits automatiquement via le signe comptable
- **Table(s) :** `invoices`
- **Colonne(s) :** `invoices_amount_lines_excl_vat`
- **Filtres obligatoires :** `end_date IS NULL`, multiplication par `invoices_sign` dans les agrégations
- **Exemple SQL :**
  ```sql
  SELECT SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_facture_ht
  FROM databox.invoices i
  INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND i.invoices_invoice_date >= '2025-01-01'
    AND i.invoices_invoice_date < '2026-01-01';
  ```
- **Ambiguïtés résolues :** Basé sur les factures (pas les commandes). Les avoirs (invoices_sign = -1) se déduisent automatiquement.

### CA commandé
- **Définition :** Montant total HT des commandes clients, hors commandes annulées (statut 0)
- **Table(s) :** `sales_orders`
- **Colonne(s) :** `sales_orders_amount_lines_excl_vat`
- **Filtres obligatoires :** `end_date IS NULL`, `sales_orders_order_status IN (1, 2)`
- **Exemple SQL :**
  ```sql
  SELECT SUM(so.sales_orders_amount_lines_excl_vat) AS ca_commande_ht
  FROM databox.sales_orders so
  INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND so.sales_orders_order_status IN (1, 2)
    AND so.sales_orders_order_date >= '2025-01-01'
    AND so.sales_orders_order_date < '2026-01-01';
  ```
- **Ambiguïtés résolues :** Statut 0 exclu (brouillon/annulé, 23 commandes). Statuts 1 (non soldée) et 2 (soldée) inclus.

### Marge
- **Définition :** Marge brute, exprimée en valeur absolue et en pourcentage du CA, selon le contexte facturé ou commandé
- **Table(s) :** `invoices` + `invoices_lines` (facturé) ou `sales_orders` (commandé)
- **Colonne(s) :** `invoices_lines_line_margin` (par ligne), `sales_orders_total_margin` (en-tête commande), `quotes_total_margin` (en-tête devis)
- **Filtres obligatoires :** `end_date IS NULL`, `* invoices_sign` pour la marge facturée
- **Exemple SQL (marge facturée) :**
  ```sql
  SELECT
    SUM(il.invoices_lines_line_margin * i.invoices_sign) AS marge_absolue,
    ROUND(100.0 * SUM(il.invoices_lines_line_margin * i.invoices_sign)
      / NULLIF(SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign), 0), 2) AS marge_pct
  FROM databox.invoices i
  INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
  INNER JOIN databox.invoices_lines il ON i.invoices_id_databox = il.invoices_id_databox
  WHERE im.end_date IS NULL;
  ```
- **Ambiguïtés résolues :** Les deux métriques (absolue et %) sont pertinentes. Contexte facturé ou commandé selon la question.

### Client actif
- **Définition :** Compte client ayant au moins une commande (statut 1 ou 2) dans les 12 derniers mois
- **Table(s) :** `sales_orders`, `accounts`
- **Colonne(s) :** `sales_orders_order_date`, `sales_orders_idcor_sold_to_account`
- **Filtres obligatoires :** `end_date IS NULL`, `sales_orders_order_status IN (1, 2)`, date dans les 12 derniers mois
- **Exemple SQL :**
  ```sql
  SELECT DISTINCT a.accounts_account_code, a.accounts_company_name_1
  FROM databox.sales_orders so
  INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
  LEFT JOIN databox.id_mapping acc_im
    ON so.sales_orders_idcor_sold_to_account = acc_im.id_correspondence
    AND acc_im.end_date IS NULL
  LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
  WHERE im.end_date IS NULL
    AND so.sales_orders_order_status IN (1, 2)
    AND so.sales_orders_order_date >= NOW() - INTERVAL '12 months';
  ```
- **Ambiguïtés résolues :** Basé sur les commandes (pas les factures). Seuil = 12 mois.

### Client inactif
- **Définition :** Client marqué inactif dans le système (`accounts_is_active = false`) OU client ayant un historique de commandes mais aucune commande dans les 12 derniers mois. `accounts_is_active = false` est prioritaire (surcharge la règle temporelle).
- **Table(s) :** `accounts`, `sales_orders`
- **Colonne(s) :** `accounts_is_active`, `sales_orders_order_date`
- **Filtres obligatoires :** `end_date IS NULL`
- **Exemple SQL :**
  ```sql
  SELECT a.accounts_account_code, a.accounts_company_name_1,
    CASE
      WHEN a.accounts_is_active = false THEN 'Désactivé'
      ELSE 'Inactif (pas de commande depuis 12 mois)'
    END AS raison
  FROM databox.accounts a
  INNER JOIN databox.id_mapping acc_im ON a.accounts_id_databox = acc_im.id_databox
  WHERE acc_im.end_date IS NULL
    AND (
      a.accounts_is_active = false
      OR (
        EXISTS (
          SELECT 1 FROM databox.sales_orders so
          INNER JOIN databox.id_mapping so_im ON so.sales_orders_id_databox = so_im.id_databox
          WHERE so.sales_orders_idcor_sold_to_account = acc_im.id_correspondence
            AND so_im.end_date IS NULL
            AND so.sales_orders_order_status IN (1, 2)
        )
        AND NOT EXISTS (
          SELECT 1 FROM databox.sales_orders so
          INNER JOIN databox.id_mapping so_im ON so.sales_orders_id_databox = so_im.id_databox
          WHERE so.sales_orders_idcor_sold_to_account = acc_im.id_correspondence
            AND so_im.end_date IS NULL
            AND so.sales_orders_order_status IN (1, 2)
            AND so.sales_orders_order_date >= NOW() - INTERVAL '12 months'
        )
      )
    );
  ```
- **Ambiguïtés résolues :** Un compte qui n'a jamais commandé n'est pas "inactif" (c'est un prospect). `accounts_is_active = false` surcharge la règle temporelle.

### Devis converti
- **Définition :** Devis ayant donné lieu à une commande, identifié par `quotes_quote_status IN ('2', '3')`
- **Table(s) :** `quotes`
- **Colonne(s) :** `quotes_quote_status`
- **Filtres obligatoires :** `end_date IS NULL`
- **Variante granulaire :** Taux de conversion par devis = ratio des lignes commandées / lignes totales (en nombre ou en montant), via jointure `quotes_lines` → `sales_orders_lines` (lien `sales_orders_lines_idcor_quote_line`)
- **Exemple SQL (binaire) :**
  ```sql
  SELECT
    COUNT(*) AS total_devis,
    COUNT(*) FILTER (WHERE q.quotes_quote_status IN ('2', '3')) AS devis_convertis,
    ROUND(100.0 * COUNT(*) FILTER (WHERE q.quotes_quote_status IN ('2', '3'))
      / NULLIF(COUNT(*), 0), 2) AS taux_conversion_pct
  FROM databox.quotes q
  INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND q.quotes_quote_date >= '2025-01-01'
    AND q.quotes_quote_date < '2026-01-01';
  ```
- **Ambiguïtés résolues :** Statuts '2' (partiellement commandé) et '3' (totalement commandé) comptent tous deux comme convertis. La variante granulaire mesure le % de conversion par devis.
- **Attention :** Les libellés value_list (`'2'`="Partiellement commandé", `'3'`="Totalement commandé") diffèrent de DB-SCHEMA.md. Les value_list sont la source de vérité.

### Avoir
- **Définition :** Facture avec signe comptable négatif (`invoices_sign = -1`). Se déduit automatiquement dans les agrégations quand on multiplie par `invoices_sign`.
- **Table(s) :** `invoices`
- **Colonne(s) :** `invoices_sign`
- **Filtres obligatoires :** `invoices_sign = -1` pour isoler les avoirs ; `* invoices_sign` dans les SUM pour déduction automatique
- **Exemple SQL :**
  ```sql
  -- Nombre et montant des avoirs sur une période
  SELECT COUNT(*) AS nb_avoirs,
    SUM(i.invoices_amount_lines_excl_vat) AS montant_avoirs_ht
  FROM databox.invoices i
  INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND i.invoices_sign = -1
    AND i.invoices_invoice_date >= '2025-01-01';
  ```
- **Ambiguïtés résolues :** Critère = `invoices_sign = -1`. Pourra être affiné avec `invoices_invoice_type` si besoin.

### Encours client
- **Définition :** Montant restant dû par un client (factures émises non encore intégralement payées)
- **Table(s) :** `open_items` (PAS `customer_due_dates` — table abandonnée)
- **Colonne(s) :** `open_items_amount_in_company_currency`, `open_items_paid_amount_in_company_currency`, `open_items_sign`
- **Filtres obligatoires :** `end_date IS NULL`
- **Exemple SQL :**
  ```sql
  SELECT
    oi.open_items_idcor_partner,
    a.accounts_company_name_1,
    SUM((oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency)
      * oi.open_items_sign) AS encours
  FROM databox.open_items oi
  INNER JOIN databox.id_mapping im ON oi.open_items_id_databox = im.id_databox
  LEFT JOIN databox.id_mapping acc_im
    ON oi.open_items_idcor_partner = acc_im.id_correspondence
    AND acc_im.end_date IS NULL
  LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
  WHERE im.end_date IS NULL
  GROUP BY oi.open_items_idcor_partner, a.accounts_company_name_1
  HAVING SUM((oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency)
    * oi.open_items_sign) > 0
  ORDER BY encours DESC;
  ```
- **Ambiguïtés résolues :** Utiliser `open_items` (pas `customer_due_dates`). Montants en devise société pour comparaison. Résolution client via `idcor_partner`.

### Taux de conversion
- **Définition :** Pourcentage de devis ayant donné lieu à une commande sur une période
- **Table(s) :** `quotes`
- **Colonne(s) :** `quotes_quote_status`, `quotes_quote_date`
- **Filtres obligatoires :** `end_date IS NULL`, commandes directes (sans devis) exclues
- **Formule :** `nb devis convertis (statut 2 ou 3) / nb total devis * 100`
- **Exemple SQL :** Voir terme "Devis converti" ci-dessus
- **Ambiguïtés résolues :** Le taux se calcule uniquement sur les devis émis. Les commandes directes sans devis source ne sont pas comptabilisées.

### Panier moyen
- **Définition :** Montant moyen HT par commande client
- **Table(s) :** `sales_orders`
- **Colonne(s) :** `sales_orders_amount_lines_excl_vat`
- **Filtres obligatoires :** `end_date IS NULL`, `sales_orders_order_status IN (1, 2)`
- **Formule :** `SUM(sales_orders_amount_lines_excl_vat) / COUNT(DISTINCT sales_orders_id_databox)`
- **Exemple SQL :**
  ```sql
  SELECT
    ROUND(SUM(so.sales_orders_amount_lines_excl_vat)
      / NULLIF(COUNT(DISTINCT so.sales_orders_id_databox), 0), 2) AS panier_moyen
  FROM databox.sales_orders so
  INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND so.sales_orders_order_status IN (1, 2)
    AND so.sales_orders_order_date >= '2025-01-01'
    AND so.sales_orders_order_date < '2026-01-01';
  ```
- **Ambiguïtés résolues :** Basé sur les commandes (pas les factures).

### Délai de paiement
- **Définition :** Nombre de jours entre l'émission de la facture et le paiement effectif (délai réel) ou conditions contractuelles du compte (délai contractuel)
- **Table(s) :** `invoices` + `open_items` (réel), `accounts` (contractuel)
- **Colonne(s) :** `invoices_invoice_date`, `open_items_payment_date` (réel), `accounts_payment_term` (contractuel)
- **Filtres obligatoires :** `end_date IS NULL`
- **Exemple SQL (délai réel moyen) :**
  ```sql
  SELECT
    AVG(EXTRACT(DAY FROM oi.open_items_payment_date - i.invoices_invoice_date)) AS delai_moyen_jours
  FROM databox.open_items oi
  INNER JOIN databox.id_mapping oi_im ON oi.open_items_id_databox = oi_im.id_databox
  INNER JOIN databox.invoices i
    ON oi.open_items_document_number = i.invoices_invoice_code
  INNER JOIN databox.id_mapping i_im ON i.invoices_id_databox = i_im.id_databox
  WHERE oi_im.end_date IS NULL
    AND i_im.end_date IS NULL
    AND oi.open_items_payment_date IS NOT NULL;
  ```
- **Ambiguïtés résolues :** Deux mesures : réel (facture→paiement via open_items) et contractuel (conditions du compte).

### Commande en retard
- **Définition :** Ligne de commande dont la date de livraison demandée est dépassée et dont la quantité restante à livrer est supérieure à zéro
- **Table(s) :** `sales_orders_lines`, `sales_orders`
- **Colonne(s) :** `sales_orders_lines_asked_delivery_date`, `sales_orders_lines_quantity_to_deliver`
- **Filtres obligatoires :** `end_date IS NULL`, `asked_delivery_date < NOW()`, `quantity_to_deliver > 0`
- **Exemple SQL :**
  ```sql
  SELECT so.sales_orders_order_code,
    sol.sales_orders_lines_product_designation,
    sol.sales_orders_lines_asked_delivery_date,
    sol.sales_orders_lines_quantity_to_deliver
  FROM databox.sales_orders_lines sol
  INNER JOIN databox.sales_orders so ON sol.sales_orders_id_databox = so.sales_orders_id_databox
  INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND sol.sales_orders_lines_asked_delivery_date < NOW()
    AND sol.sales_orders_lines_quantity_to_deliver > 0
  ORDER BY sol.sales_orders_lines_asked_delivery_date ASC;
  ```
- **Ambiguïtés résolues :** Granularité à la ligne de commande (pas à l'en-tête).

### Commercial
- **Définition :** Représentant commercial associé aux documents de vente (devis, commandes, factures)
- **Table(s) :** `sales_reps`
- **Colonne(s) :** `sales_reps_rep_code`, `sales_reps_name`
- **Résolution :** Via `idcor_sales_rep` dans les documents → `id_mapping` → `sales_reps`
- **Exemple SQL :**
  ```sql
  -- Résoudre le commercial d'une facture
  LEFT JOIN databox.id_mapping rep_im
    ON i.invoices_idcor_sales_rep = rep_im.id_correspondence
    AND rep_im.end_date IS NULL
  LEFT JOIN databox.sales_reps sr ON rep_im.id_databox = sr.sales_reps_id_databox
  ```
- **Ambiguïtés résolues :** Même pattern de résolution idcor que pour les comptes, avec alias distinct (`rep_im`).
