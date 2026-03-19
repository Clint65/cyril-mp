# Exemples NL→SQL — Domaine Facturation & Encours

> Requêtes validées par exécution sur la base Databox.

---

### Exemple 1 : Détail factures vs avoirs par client

**Question :** "Détail des factures et avoirs par client en 2024"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  COUNT(*) FILTER (WHERE i.invoices_sign = 1) AS nb_factures,
  COALESCE(SUM(i.invoices_amount_lines_excl_vat) FILTER (WHERE i.invoices_sign = 1), 0) AS montant_factures,
  COUNT(*) FILTER (WHERE i.invoices_sign = -1) AS nb_avoirs,
  COALESCE(SUM(i.invoices_amount_lines_excl_vat) FILTER (WHERE i.invoices_sign = -1), 0) AS montant_avoirs,
  SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_net_ht
FROM databox.invoices i
INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON i.invoices_idcor_account = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE doc_im.end_date IS NULL
  AND i.invoices_invoice_date >= '2024-01-01' AND i.invoices_invoice_date < '2025-01-01'
GROUP BY a.accounts_company_name_1
ORDER BY ca_net_ht DESC;
```

**Points clés :**
- `COUNT FILTER` et `SUM FILTER` pour séparer factures (sign=1) et avoirs (sign=-1)
- `ca_net_ht` = SUM avec invoices_sign pour le net

**Résultat (extrait) :**

| client | nb_factures | montant_factures | nb_avoirs | montant_avoirs | ca_net_ht |
|--------|-------------|-----------------|-----------|----------------|-----------|
| ALCAN | 1 | 7 005 | 0 | 0 | 7 005 |
| VELOLAND CHOLET | 6 | 4 456 | 4 | 1 013 | 3 443 |

---

### Exemple 2 : Encours client (open_items)

**Question :** "Quel est l'encours de chaque client ?"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  SUM((oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency)
    * oi.open_items_sign) AS encours
FROM databox.open_items oi
INNER JOIN databox.id_mapping oi_im ON oi.open_items_id_databox = oi_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON oi.open_items_idcor_partner = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE oi_im.end_date IS NULL
GROUP BY a.accounts_company_name_1
HAVING SUM((oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency)
  * oi.open_items_sign) > 0
ORDER BY encours DESC;
```

**Points clés :**
- Table `open_items` (PAS `customer_due_dates` — abandonnée)
- Montants en devise société (`_in_company_currency`) pour comparaison
- `open_items_sign` comme `invoices_sign`
- Résolution client via `open_items_idcor_partner`
- `HAVING > 0` pour n'afficher que les encours positifs

**Note :** Table `open_items` non alimentée dans le jeu de données actuel. Syntaxe validée.

---

### Exemple 3 : Balance âgée

**Question :** "Balance âgée de mes clients"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  SUM(CASE WHEN oi.open_items_payment_date >= NOW() - INTERVAL '30 days' THEN
    (oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency) * oi.open_items_sign
    ELSE 0 END) AS "0-30j",
  SUM(CASE WHEN oi.open_items_payment_date >= NOW() - INTERVAL '60 days'
    AND oi.open_items_payment_date < NOW() - INTERVAL '30 days' THEN
    (oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency) * oi.open_items_sign
    ELSE 0 END) AS "30-60j",
  SUM(CASE WHEN oi.open_items_payment_date >= NOW() - INTERVAL '90 days'
    AND oi.open_items_payment_date < NOW() - INTERVAL '60 days' THEN
    (oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency) * oi.open_items_sign
    ELSE 0 END) AS "60-90j",
  SUM(CASE WHEN oi.open_items_payment_date < NOW() - INTERVAL '90 days' THEN
    (oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency) * oi.open_items_sign
    ELSE 0 END) AS ">90j"
FROM databox.open_items oi
INNER JOIN databox.id_mapping oi_im ON oi.open_items_id_databox = oi_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON oi.open_items_idcor_partner = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE oi_im.end_date IS NULL
GROUP BY a.accounts_company_name_1
ORDER BY a.accounts_company_name_1;
```

**Points clés :**
- Ventilation par tranche d'ancienneté via `CASE WHEN` sur `open_items_payment_date`
- 4 tranches : 0-30j, 30-60j, 60-90j, >90j

**Note :** Syntaxe validée, à tester avec données réelles.

---

### Exemple 4 : Cycle complet d'un article (devis→commandes→factures)

**Question :** "Résumé de l'activité de l'article X sur devis, commandes et factures"

**SQL :**
```sql
-- Remplacer 'COR-ART-XXX' par l'id_correspondence de l'article
SELECT 'devis' AS type,
  COUNT(DISTINCT ql.quotes_id_databox) AS nb_docs,
  SUM(ql.quotes_lines_amount_excl_vat) AS total_ht
FROM databox.quotes_lines ql
INNER JOIN databox.quotes q ON ql.quotes_id_databox = q.quotes_id_databox
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE ql.quotes_lines_idcor_product = 'COR-ART-XXX'
  AND im.end_date IS NULL

UNION ALL

SELECT 'commandes',
  COUNT(DISTINCT sol.sales_orders_id_databox),
  SUM(sol.sales_orders_lines_net_price * sol.sales_orders_lines_ordered_quantity)
FROM databox.sales_orders_lines sol
INNER JOIN databox.sales_orders so ON sol.sales_orders_id_databox = so.sales_orders_id_databox
INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
WHERE sol.sales_orders_lines_idcor_product = 'COR-ART-XXX'
  AND im.end_date IS NULL

UNION ALL

SELECT 'factures',
  COUNT(DISTINCT il.invoices_id_databox),
  SUM(il.invoices_lines_amount_excl_vat)
FROM databox.invoices_lines il
INNER JOIN databox.invoices i ON il.invoices_id_databox = i.invoices_id_databox
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE il.invoices_lines_idcor_product = 'COR-ART-XXX'
  AND im.end_date IS NULL;
```

**Points clés :**
- Pattern `UNION ALL` pour croiser 3 tables de lignes
- Utilise `idcor_product` (id_correspondence) — pas besoin du cast TEXT ici
- Chaque bloc a son propre `INNER JOIN id_mapping` avec `end_date IS NULL`

---

### Exemple 5 : Top articles par CA facturé

**Question :** "Quels sont mes 10 articles les plus vendus en CA ?"

**SQL :**
```sql
SELECT
  art.articles_product_code,
  il.invoices_lines_product_designation,
  SUM(il.invoices_lines_amount_excl_vat * i.invoices_sign) AS ca_net_ht,
  SUM(il.invoices_lines_quantity) AS total_qty
FROM databox.invoices_lines il
INNER JOIN databox.invoices i ON il.invoices_id_databox = i.invoices_id_databox
INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
LEFT JOIN databox.articles art
  ON il.invoices_lines_id_product = art.articles_id_databox::text
WHERE doc_im.end_date IS NULL
  AND i.invoices_invoice_date >= '2024-01-01' AND i.invoices_invoice_date < '2025-01-01'
GROUP BY art.articles_product_code, il.invoices_lines_product_designation
ORDER BY ca_net_ht DESC
LIMIT 10;
```

**Points clés :**
- Cast `::text` obligatoire sur `articles_id_databox` pour la jointure
- `invoices_sign` dans le SUM pour déduire les avoirs au niveau article
- Jointure via `invoices_lines_id_product` (id_databox en text)
