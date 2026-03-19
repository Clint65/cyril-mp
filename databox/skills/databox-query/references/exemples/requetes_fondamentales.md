# Exemples NL→SQL fondamentaux

> Paires question/SQL validées par exécution sur la base Databox.
> Chaque exemple illustre un pattern critique du schéma.

## Patterns couverts

| # | Pattern | Règle Databox |
|---|---------|---------------|
| 1 | Version courante | `INNER JOIN id_mapping` + `end_date IS NULL` |
| 2 | Résolution idcor | `idcor_account` → `id_mapping` → `accounts` avec alias |
| 3 | Jointure article (cast TEXT) | `id_product = articles_id_databox::text` |
| 4 | Agrégation temporelle | `generate_series` + `LEFT JOIN` pour périodes vides |
| 5 | invoices_sign | `SUM(montant * invoices_sign)` pour déduire les avoirs |
| 6 | Aliases multiples id_mapping | 3 jointures id_mapping avec aliases distincts |

---

## Exemples

### Exemple 1 : Version courante — Lister les clients actifs

**Question :** "Liste tous les clients actifs"

**SQL :**
```sql
SELECT DISTINCT
  a.accounts_account_code,
  a.accounts_company_name_1
FROM databox.sales_orders so
INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON so.sales_orders_idcor_sold_to_account = acc_im.id_correspondence
  AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE im.end_date IS NULL
  AND so.sales_orders_order_status IN (1, 2)
  AND so.sales_orders_order_date >= NOW() - INTERVAL '12 months'
ORDER BY a.accounts_company_name_1;
```

**Points clés :**
- `INNER JOIN id_mapping` + `end_date IS NULL` pour ne voir que les versions courantes
- Définition "client actif" du glossaire : commande dans les 12 derniers mois, statut IN (1, 2)
- Résolution du compte via `idcor_sold_to_account` → `id_mapping` (alias `acc_im`) → `accounts`

**Résultat (extrait) :**

| accounts_account_code | accounts_company_name_1 |
|----------------------|------------------------|
| 000033 | CULTURE VELO ST NAZAIRE |
| 000010 | HYPER U AGDE Leapwork |
| FR052 | OnicIndus |

---

### Exemple 2 : Résolution idcor — Factures d'un client

**Question :** "Quelles sont les factures du client VELOLAND CHOLET ?"

**SQL :**
```sql
SELECT
  i.invoices_invoice_code,
  i.invoices_invoice_date,
  i.invoices_amount_lines_excl_vat,
  i.invoices_sign,
  a.accounts_company_name_1
FROM databox.invoices i
INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON i.invoices_idcor_account = acc_im.id_correspondence
  AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE doc_im.end_date IS NULL
  AND a.accounts_company_name_1 = 'VELOLAND CHOLET'
ORDER BY i.invoices_invoice_date DESC;
```

**Points clés :**
- Résolution `invoices_idcor_account` → `id_mapping` (alias `acc_im`) → `accounts`
- Alias distinct `doc_im` pour le document et `acc_im` pour le compte
- Filtrage sur le nom du client après résolution

**Résultat (extrait) :**

| invoices_invoice_code | invoices_invoice_date | amount_excl_vat | sign |
|----------------------|----------------------|-----------------|------|
| PGC111211FACLI000001 | 2024-11-06 | 417,00 | +1 |
| PGC111203AVCLI000001 | 2024-03-12 | 236,80 | -1 |
| PGC111203FACLI000002 | 2024-03-08 | 637,00 | +1 |

---

### Exemple 3 : Jointure article avec cast TEXT

**Question :** "Quel article se vend le plus en quantité ?"

**SQL :**
```sql
SELECT
  art.articles_product_code,
  sol.sales_orders_lines_product_designation,
  SUM(sol.sales_orders_lines_ordered_quantity) AS total_qty
FROM databox.sales_orders_lines sol
INNER JOIN databox.sales_orders so ON sol.sales_orders_id_databox = so.sales_orders_id_databox
INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
LEFT JOIN databox.articles art
  ON sol.sales_orders_lines_id_product = art.articles_id_databox::text
WHERE im.end_date IS NULL
  AND so.sales_orders_order_status IN (1, 2)
GROUP BY art.articles_product_code, sol.sales_orders_lines_product_designation
ORDER BY total_qty DESC
LIMIT 5;
```

**Points clés :**
- **Cast obligatoire** : `sales_orders_lines_id_product = art.articles_id_databox::text`
- Sans le `::text`, la jointure serait silencieusement vide (types incompatibles)
- Le `end_date IS NULL` est sur l'en-tête commande (pas besoin sur les lignes, elles suivent l'en-tête)

**Résultat (extrait) :**

| articles_product_code | designation | total_qty |
|----------------------|-------------|-----------|
| 1453_DC_60 | MEMBRANE SILICONE CUITE | 30 000 000 109 |
| RAW921 | Caja para carrito | 200 000 |
| FIN303 | Sce pimentée Carrefour 250gx24 | 30 000 |

---

### Exemple 4 : Agrégation temporelle avec generate_series

**Question :** "Quel est mon CA facturé par mois en 2024 ?"

**SQL :**
```sql
SELECT
  TO_CHAR(periods.period, 'YYYY-MM') AS mois,
  COALESCE(COUNT(DISTINCT i.invoices_id_databox), 0) AS nb_factures,
  COALESCE(SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign), 0) AS ca_facture_ht
FROM generate_series(
  '2024-01-01'::timestamp,
  '2024-12-01'::timestamp,
  '1 month'::interval
) AS periods(period)
LEFT JOIN databox.invoices i
  ON DATE_TRUNC('month', i.invoices_invoice_date) = periods.period
LEFT JOIN databox.id_mapping im
  ON i.invoices_id_databox = im.id_databox
  AND im.end_date IS NULL
GROUP BY periods.period
ORDER BY periods.period ASC;
```

**Points clés :**
- `generate_series` crée toutes les périodes, y compris celles sans données (avril, juillet, août = 0)
- `LEFT JOIN` sur invoices et id_mapping pour ne pas perdre les périodes vides
- `COALESCE(..., 0)` pour afficher 0 au lieu de NULL
- `TO_CHAR` pour un affichage lisible des mois
- Adapter l'intervalle pour trimestre (`'3 months'`) ou année (`'1 year'`)

**Résultat (extrait) :**

| mois | nb_factures | ca_facture_ht |
|------|-------------|---------------|
| 2024-01 | 3 | 1 073,33 |
| 2024-02 | 5 | 1 252,00 |
| 2024-03 | 9 | 389,20 |
| 2024-04 | 0 | 0,00 |
| 2024-05 | 3 | 1 746,20 |
| 2024-06 | 1 | 7 005,00 |

---

### Exemple 5 : invoices_sign — CA net vs CA brut

**Question :** "Quel est le CA net facturé en 2024 ?"

**SQL :**
```sql
SELECT
  COUNT(*) FILTER (WHERE i.invoices_sign = 1) AS nb_factures,
  COUNT(*) FILTER (WHERE i.invoices_sign = -1) AS nb_avoirs,
  SUM(i.invoices_amount_lines_excl_vat) AS ca_brut_ht,
  SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_net_ht
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL
  AND i.invoices_invoice_date >= '2024-01-01'
  AND i.invoices_invoice_date < '2025-01-01';
```

**Points clés :**
- `SUM(montant * invoices_sign)` : les avoirs (sign=-1) se déduisent automatiquement
- `SUM(montant)` sans le signe donne le CA brut (factures + avoirs comptés positivement)
- `COUNT FILTER` pour séparer le nombre de factures et d'avoirs
- **Piège** : sans `* invoices_sign`, le CA serait gonflé de 3 639 € (montant des avoirs compté deux fois)

**Résultat :**

| nb_factures | nb_avoirs | ca_brut_ht | ca_net_ht |
|-------------|-----------|------------|-----------|
| 25 | 10 | 17 730,35 | 14 091,41 |

---

### Exemple 6 : Aliases multiples id_mapping

**Question :** "CA facturé par client en 2024, avec nom du commercial"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  sr.sales_reps_name AS commercial,
  SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_net_ht
FROM databox.invoices i
INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON i.invoices_idcor_account = acc_im.id_correspondence
  AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
LEFT JOIN databox.id_mapping rep_im
  ON i.invoices_idcor_sales_rep = rep_im.id_correspondence
  AND rep_im.end_date IS NULL
LEFT JOIN databox.sales_reps sr ON rep_im.id_databox = sr.sales_reps_id_databox
WHERE doc_im.end_date IS NULL
  AND i.invoices_invoice_date >= '2024-01-01'
  AND i.invoices_invoice_date < '2025-01-01'
GROUP BY a.accounts_company_name_1, sr.sales_reps_name
ORDER BY ca_net_ht DESC;
```

**Points clés :**
- **3 jointures id_mapping** avec aliases distincts obligatoires :
  - `doc_im` : version courante du document (facture)
  - `acc_im` : résolution du compte client via `idcor_account`
  - `rep_im` : résolution du commercial via `idcor_sales_rep`
- Chaque jointure id_mapping porte son propre filtre `end_date IS NULL`
- **Piège** : sans alias, PostgreSQL rejette la requête (ambiguïté sur `id_databox`, `end_date`, etc.)

**Résultat (extrait) :**

| client | commercial | ca_net_ht |
|--------|-----------|-----------|
| ALCAN | (null) | 7 005,00 |
| VELOLAND CHOLET | (null) | 3 442,53 |
| AERONEF | (null) | 2 050,00 |
