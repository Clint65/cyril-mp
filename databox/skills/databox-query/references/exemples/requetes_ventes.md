# Exemples NL→SQL — Domaine Ventes

> Requêtes validées par exécution sur la base Databox.
> Complète les exemples fondamentaux (requetes_fondamentales.md).

---

### Exemple 1 : CA facturé par trimestre

**Question :** "Quel est mon CA facturé par trimestre en 2024 ?"

**SQL :**
```sql
SELECT
  TO_CHAR(periods.period, 'YYYY-"Q"Q') AS trimestre,
  COALESCE(COUNT(DISTINCT i.invoices_id_databox), 0) AS nb_factures,
  COALESCE(SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign), 0) AS ca_facture_ht
FROM generate_series(
  '2024-01-01'::timestamp,
  '2024-10-01'::timestamp,
  '3 months'::interval
) AS periods(period)
LEFT JOIN databox.invoices i
  ON DATE_TRUNC('quarter', i.invoices_invoice_date) = periods.period
LEFT JOIN databox.id_mapping im
  ON i.invoices_id_databox = im.id_databox
  AND im.end_date IS NULL
GROUP BY periods.period
ORDER BY periods.period ASC;
```

**Points clés :**
- `generate_series` avec intervalle `'3 months'` pour les trimestres
- `DATE_TRUNC('quarter')` pour l'agrégation
- `TO_CHAR(..., 'YYYY-"Q"Q')` pour un affichage "2024-Q1"
- `invoices_sign` dans le SUM pour déduire les avoirs

**Résultat :**

| trimestre | nb_factures | ca_facture_ht |
|-----------|-------------|---------------|
| 2024-Q1 | 17 | 2 714,53 |
| 2024-Q2 | 4 | 8 751,20 |
| 2024-Q3 | 1 | 2 000,00 |
| 2024-Q4 | 13 | 625,68 |

---

### Exemple 2 : Comparaison CA année N / N-1

**Question :** "Compare mon CA facturé 2024 avec 2023"

**SQL :**
```sql
WITH ca_par_annee AS (
  SELECT
    EXTRACT(YEAR FROM i.invoices_invoice_date) AS annee,
    SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_net_ht
  FROM databox.invoices i
  INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND i.invoices_invoice_date >= '2023-01-01'
    AND i.invoices_invoice_date < '2025-01-01'
  GROUP BY EXTRACT(YEAR FROM i.invoices_invoice_date)
)
SELECT
  n1.annee AS annee_n_1,
  n1.ca_net_ht AS ca_n_1,
  n.annee AS annee_n,
  n.ca_net_ht AS ca_n,
  ROUND(100.0 * (n.ca_net_ht - n1.ca_net_ht) / NULLIF(ABS(n1.ca_net_ht), 0), 2) AS evolution_pct
FROM ca_par_annee n
JOIN ca_par_annee n1 ON n.annee = n1.annee + 1;
```

**Points clés :**
- CTE `ca_par_annee` pour calculer le CA par année
- Auto-jointure pour mettre N et N-1 côté à côte
- `ABS()` dans le dénominateur pour gérer un CA N-1 négatif (cas avoirs > factures)
- Le `% d'évolution` est calculé comme `(N - N-1) / |N-1| * 100`

**Résultat :**

| annee_n_1 | ca_n_1 | annee_n | ca_n | evolution_pct |
|-----------|--------|---------|------|---------------|
| 2023 | -3 277,10 | 2024 | 14 091,41 | +530,00% |

---

### Exemple 3 : CA facturé par commercial

**Question :** "Quel est le CA facturé par commercial en 2024 ?"

**SQL :**
```sql
SELECT
  COALESCE(sr.sales_reps_name, '(non assigné)') AS commercial,
  COUNT(DISTINCT i.invoices_id_databox) AS nb_factures,
  SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_net_ht
FROM databox.invoices i
INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping rep_im
  ON i.invoices_idcor_sales_rep = rep_im.id_correspondence
  AND rep_im.end_date IS NULL
LEFT JOIN databox.sales_reps sr ON rep_im.id_databox = sr.sales_reps_id_databox
WHERE doc_im.end_date IS NULL
  AND i.invoices_invoice_date >= '2024-01-01'
  AND i.invoices_invoice_date < '2025-01-01'
GROUP BY sr.sales_reps_name
ORDER BY ca_net_ht DESC;
```

**Points clés :**
- Résolution commercial via `invoices_idcor_sales_rep` → `id_mapping` (alias `rep_im`) → `sales_reps`
- `COALESCE` pour afficher "(non assigné)" quand pas de commercial lié
- Deux jointures id_mapping avec aliases distincts (`doc_im`, `rep_im`)

**Résultat :**

| commercial | nb_factures | ca_net_ht |
|-----------|-------------|-----------|
| (non assigné) | 35 | 14 091,41 |

---

### Exemple 4 : Évolution CA commandé par mois

**Question :** "Évolution de mon CA commandé par mois en 2024"

**SQL :**
```sql
SELECT
  TO_CHAR(periods.period, 'YYYY-MM') AS mois,
  COALESCE(COUNT(DISTINCT so.sales_orders_id_databox), 0) AS nb_commandes,
  COALESCE(SUM(so.sales_orders_amount_lines_excl_vat), 0) AS ca_commande_ht
FROM generate_series(
  '2024-01-01'::timestamp,
  '2024-12-01'::timestamp,
  '1 month'::interval
) AS periods(period)
LEFT JOIN databox.sales_orders so
  ON DATE_TRUNC('month', so.sales_orders_order_date) = periods.period
  AND so.sales_orders_order_status IN (1, 2)
LEFT JOIN databox.id_mapping im
  ON so.sales_orders_id_databox = im.id_databox
  AND im.end_date IS NULL
GROUP BY periods.period
ORDER BY periods.period ASC;
```

**Points clés :**
- Même pattern generate_series que pour le CA facturé
- `order_status IN (1, 2)` pour exclure les commandes annulées (statut 0)
- Pas de `invoices_sign` sur les commandes (pas d'avoirs dans sales_orders)

---

### Exemple 5 : Taux de conversion devis→commandes global

**Question :** "Quel est mon taux de conversion devis/commandes en 2024 ?"

**SQL :**
```sql
SELECT
  COUNT(*) AS total_devis,
  COUNT(*) FILTER (WHERE q.quotes_quote_status IN ('2', '3')) AS devis_convertis,
  ROUND(100.0 * COUNT(*) FILTER (WHERE q.quotes_quote_status IN ('2', '3'))
    / NULLIF(COUNT(*), 0), 2) AS taux_conversion_pct
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE im.end_date IS NULL
  AND q.quotes_quote_date >= '2024-01-01'
  AND q.quotes_quote_date < '2025-01-01';
```

**Points clés :**
- `quotes_quote_status IN ('2', '3')` : partiellement commandé + totalement commandé = converti
- `COUNT FILTER` pour compter conditionnellement sans sous-requête
- Commandes directes (sans devis) exclues — on ne compte que les devis émis

**Résultat :**

| total_devis | devis_convertis | taux_conversion_pct |
|-------------|-----------------|---------------------|
| 27 | 13 | 48,15% |

---

### Exemple 6 : Taux de conversion par mois

**Question :** "Comment évolue mon taux de conversion mois par mois en 2024 ?"

**SQL :**
```sql
SELECT
  TO_CHAR(periods.period, 'YYYY-MM') AS mois,
  COALESCE(COUNT(q.quotes_id_databox), 0) AS total_devis,
  COALESCE(COUNT(q.quotes_id_databox) FILTER (WHERE q.quotes_quote_status IN ('2', '3')), 0) AS convertis,
  CASE WHEN COUNT(q.quotes_id_databox) > 0
    THEN ROUND(100.0 * COUNT(q.quotes_id_databox) FILTER (WHERE q.quotes_quote_status IN ('2', '3'))
      / COUNT(q.quotes_id_databox), 2)
    ELSE 0 END AS taux_pct
FROM generate_series('2024-01-01'::timestamp, '2024-12-01'::timestamp, '1 month'::interval) AS periods(period)
LEFT JOIN databox.quotes q
  ON DATE_TRUNC('month', q.quotes_quote_date) = periods.period
LEFT JOIN databox.id_mapping im
  ON q.quotes_id_databox = im.id_databox
  AND im.end_date IS NULL
GROUP BY periods.period
ORDER BY periods.period;
```

**Points clés :**
- generate_series pour inclure les mois sans devis
- `CASE WHEN` pour éviter la division par zéro sur les mois vides

**Résultat (extrait) :**

| mois | total_devis | convertis | taux_pct |
|------|-------------|-----------|----------|
| 2024-03 | 8 | 4 | 50,00% |
| 2024-05 | 20 | 18 | 90,00% |
| 2024-11 | 2 | 2 | 100,00% |

---

### Exemple 7 : Montant des devis convertis vs perdus

**Question :** "Quel est le montant des devis perdus vs convertis en 2024 ?"

**SQL :**
```sql
SELECT
  CASE
    WHEN q.quotes_quote_status = '1' THEN 'Non commandé (ouvert)'
    WHEN q.quotes_quote_status = '2' THEN 'Partiellement commandé'
    WHEN q.quotes_quote_status = '3' THEN 'Totalement commandé'
  END AS statut,
  COUNT(*) AS nb_devis,
  SUM(q.quotes_amount_lines_excl_vat) AS montant_ht
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE im.end_date IS NULL
  AND q.quotes_quote_date >= '2024-01-01'
  AND q.quotes_quote_date < '2025-01-01'
GROUP BY q.quotes_quote_status
ORDER BY q.quotes_quote_status;
```

**Points clés :**
- CASE pour décoder les statuts en libellés lisibles (alternative au pattern value_list)
- Utile pour les statuts bien connus (devis) sans jointure supplémentaire

**Résultat :**

| statut | nb_devis | montant_ht |
|--------|----------|------------|
| Non commandé (ouvert) | 14 | 2 574,84 |
| Totalement commandé | 13 | 6 094,35 |

---

### Exemple 8 : Analyse Pareto — part du CA des top clients

**Question :** "Quelle part de mon CA font mes meilleurs clients en 2024 ?"

**SQL :**
```sql
WITH ca_client AS (
  SELECT
    a.accounts_company_name_1 AS client,
    SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_net_ht
  FROM databox.invoices i
  INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
  LEFT JOIN databox.id_mapping acc_im
    ON i.invoices_idcor_account = acc_im.id_correspondence AND acc_im.end_date IS NULL
  LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
  WHERE doc_im.end_date IS NULL
    AND i.invoices_invoice_date >= '2024-01-01' AND i.invoices_invoice_date < '2025-01-01'
  GROUP BY a.accounts_company_name_1
  HAVING SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) > 0
),
ranked AS (
  SELECT client, ca_net_ht,
    SUM(ca_net_ht) OVER (ORDER BY ca_net_ht DESC) AS cumul,
    SUM(ca_net_ht) OVER () AS total,
    ROW_NUMBER() OVER (ORDER BY ca_net_ht DESC) AS rang,
    COUNT(*) OVER () AS nb_clients
  FROM ca_client
)
SELECT client, ca_net_ht,
  ROUND(100.0 * cumul / total, 2) AS cumul_pct,
  rang, nb_clients
FROM ranked ORDER BY rang;
```

**Points clés :**
- CTE + window functions (`SUM OVER`, `ROW_NUMBER`) pour le cumul
- `HAVING > 0` pour exclure les clients à CA négatif (avoirs > factures)
- Le top 2 clients représente 71% du CA (Pareto)

**Résultat (extrait) :**

| client | ca_net_ht | cumul_pct | rang |
|--------|-----------|-----------|------|
| ALCAN | 7 005 | 47,84% | 1 |
| VELOLAND CHOLET | 3 443 | 71,36% | 2 |
| AERONEF | 2 050 | 85,36% | 3 |

---

### Exemple 9 : Clients sans commande récente

**Question :** "Quels clients n'ont pas commandé depuis 3 mois alors qu'ils commandaient avant ?"

**SQL :**
```sql
SELECT
  a.accounts_account_code,
  a.accounts_company_name_1,
  MAX(so.sales_orders_order_date) AS derniere_commande
FROM databox.accounts a
INNER JOIN databox.id_mapping acc_im ON a.accounts_id_databox = acc_im.id_databox
INNER JOIN databox.sales_orders so
  ON so.sales_orders_idcor_sold_to_account = acc_im.id_correspondence
INNER JOIN databox.id_mapping so_im ON so.sales_orders_id_databox = so_im.id_databox
WHERE acc_im.end_date IS NULL
  AND so_im.end_date IS NULL
  AND so.sales_orders_order_status IN (1, 2)
GROUP BY a.accounts_account_code, a.accounts_company_name_1
HAVING MAX(so.sales_orders_order_date) < NOW() - INTERVAL '3 months'
  AND MAX(so.sales_orders_order_date) >= NOW() - INTERVAL '15 months'
ORDER BY derniere_commande DESC;
```

**Points clés :**
- `HAVING` avec deux conditions : pas de commande récente (< 3 mois) mais commande dans les 15 derniers mois
- Jointure inverse : on part des accounts et on rejoint les commandes
- Le seuil de 3 mois est paramétrable dans la question

---

### Exemple 10 : Nouveaux clients sur une période

**Question :** "Combien de nouveaux clients en 2024 ?"

**SQL :**
```sql
SELECT
  a.accounts_account_code,
  a.accounts_company_name_1,
  MIN(so.sales_orders_order_date) AS premiere_commande
FROM databox.accounts a
INNER JOIN databox.id_mapping acc_im ON a.accounts_id_databox = acc_im.id_databox
INNER JOIN databox.sales_orders so
  ON so.sales_orders_idcor_sold_to_account = acc_im.id_correspondence
INNER JOIN databox.id_mapping so_im ON so.sales_orders_id_databox = so_im.id_databox
WHERE acc_im.end_date IS NULL
  AND so_im.end_date IS NULL
  AND so.sales_orders_order_status IN (1, 2)
GROUP BY a.accounts_account_code, a.accounts_company_name_1
HAVING MIN(so.sales_orders_order_date) >= '2024-01-01'
  AND MIN(so.sales_orders_order_date) < '2025-01-01'
ORDER BY premiere_commande;
```

**Points clés :**
- `MIN(order_date)` = date de première commande du client
- `HAVING MIN(...) >= '2024-01-01'` = première commande dans la période = nouveau client
- 12 nouveaux clients en 2024
