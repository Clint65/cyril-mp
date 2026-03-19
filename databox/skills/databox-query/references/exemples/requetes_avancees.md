# Exemples NL→SQL — Cas avancés

> Requêtes validées par exécution sur la base Databox.
> Couvre multi-devises, décodage value_lists, et clients à risque.

---

## Multi-devises

### Exemple 1 : CA total en devise de référence

**Question :** "Quel est mon CA total, toutes devises confondues ?"

**SQL :**
```sql
SELECT
  SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_sans_conversion,
  SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign
    * COALESCE(i.invoices_currency_rate, 1)) AS ca_converti_eur
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL;
```

**Points clés :**
- `* COALESCE(invoices_currency_rate, 1)` pour convertir en devise de référence
- `COALESCE` pour gérer les factures sans taux (= devise de référence, rate = 1)
- Même pattern applicable sur `quotes_currency_rate`, `sales_orders_currency_rate`

**Résultat :**

| ca_sans_conversion | ca_converti_eur |
|-------------------|-----------------|
| 12 274 987,26 | 12 199 085,74 |

---

### Exemple 2 : CA par devise

**Question :** "Répartition de mon CA par devise"

**SQL :**
```sql
SELECT
  i.invoices_currency AS devise,
  COUNT(DISTINCT i.invoices_id_databox) AS nb_factures,
  SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_devise_originale,
  SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign
    * COALESCE(i.invoices_currency_rate, 1)) AS ca_converti_eur
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL
GROUP BY i.invoices_currency
ORDER BY ca_converti_eur DESC;
```

**Résultat :**

| devise | nb_factures | ca_devise_originale | ca_converti_eur |
|--------|-------------|--------------------|-----------------|
| EUR | 277 | 11 171 904 | ~11 171 904 |
| GBP | 6 | 877 504 | ~877 504 |
| CHF | 17 | 77 585 | ~81 087 |
| USD | 4 | 53 619 | ~42 285 |
| AOA | 2 | 91 077 | ~679 |

---

## Décodage value_lists

### Exemple 3 : Liste des colonnes décodables

**Question :** "Quelles colonnes ont un décodage via value_list ?"

**SQL :**
```sql
SELECT vlcc.column_name, vl.code AS value_list_code, vl.name AS value_list_name, vlcc.is_array
FROM databox.value_list_column_config vlcc
JOIN databox.value_list vl ON vl.id = vlcc.value_list_id
ORDER BY vlcc.column_name;
```

**Points clés :**
- 58 colonnes décodables dans le schéma
- `column_name` en **camelCase** (ex : `salesOrdersOrderStatus`, pas `sales_orders_order_status`)
- `is_array = true` pour les colonnes contenant un tableau de codes (ex : `salesOrdersLinesFamilyCode`)

**Colonnes les plus utiles :**

| column_name | value_list | Usage |
|-------------|-----------|-------|
| `quotesQuoteStatus` | local_menus_430 | Statut devis |
| `salesOrdersOrderStatus` | local_menus_415 | Statut commande |
| `salesOrdersDeliveryStatus` | local_menus_417 | Statut livraison |
| `salesOrdersInvoiceStatus` | local_menus_418 | Statut facturation |
| `manufacturingOrdersStatus` | local_menus_363 | Statut OF |
| `accountsAccountType` | account_type | Type de compte |
| `accountsCategory` | customer_category | Catégorie client |
| `accountsPaymentTerm` | payment_term | Conditions paiement |

---

## Clients à risque

### Exemple 4 : Clients en baisse de CA > 20%

**Question :** "Quels clients ont un CA en baisse de plus de 20% par rapport à l'année dernière ?"

**SQL :**
```sql
WITH ca_client_annee AS (
  SELECT
    i.invoices_idcor_account,
    EXTRACT(YEAR FROM i.invoices_invoice_date) AS annee,
    SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca
  FROM databox.invoices i
  INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
  WHERE im.end_date IS NULL
    AND i.invoices_invoice_date >= '2023-01-01' AND i.invoices_invoice_date < '2025-01-01'
  GROUP BY i.invoices_idcor_account, EXTRACT(YEAR FROM i.invoices_invoice_date)
)
SELECT
  a.accounts_company_name_1 AS client,
  n1.ca AS ca_n_1,
  n.ca AS ca_n,
  ROUND(100.0 * (n.ca - n1.ca) / NULLIF(ABS(n1.ca), 0), 2) AS evolution_pct
FROM ca_client_annee n
JOIN ca_client_annee n1
  ON n.invoices_idcor_account = n1.invoices_idcor_account
  AND n.annee = 2024 AND n1.annee = 2023
LEFT JOIN databox.id_mapping acc_im
  ON n.invoices_idcor_account = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE n1.ca > 0
  AND ROUND(100.0 * (n.ca - n1.ca) / NULLIF(ABS(n1.ca), 0), 2) < -20
ORDER BY evolution_pct ASC;
```

**Points clés :**
- CTE pour calculer le CA par client par année
- Auto-jointure pour comparer N et N-1
- `WHERE n1.ca > 0` pour exclure les clients sans CA positif en N-1
- Seuil -20% paramétrable dans la question

**Résultat :**

| client | ca_n_1 | ca_n | evolution_pct |
|--------|--------|------|---------------|
| XNS Suisse | 75 862 | 1 696 | -97,76% |
| AERONEF | 4 318 | 2 050 | -52,52% |

---

### Exemple 5 : Score de risque combiné

**Question :** "Quels clients cumulent plusieurs signaux de risque ?"

**SQL :**
```sql
WITH derniere_commande AS (
  SELECT so.sales_orders_idcor_sold_to_account AS idcor,
    MAX(so.sales_orders_order_date) AS derniere_cde
  FROM databox.sales_orders so
  INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
  WHERE im.end_date IS NULL AND so.sales_orders_order_status IN (1, 2)
  GROUP BY so.sales_orders_idcor_sold_to_account
),
ca_evolution AS (
  SELECT i.invoices_idcor_account AS idcor,
    SUM(CASE WHEN EXTRACT(YEAR FROM i.invoices_invoice_date) = 2024
      THEN i.invoices_amount_lines_excl_vat * i.invoices_sign ELSE 0 END) AS ca_n,
    SUM(CASE WHEN EXTRACT(YEAR FROM i.invoices_invoice_date) = 2023
      THEN i.invoices_amount_lines_excl_vat * i.invoices_sign ELSE 0 END) AS ca_n1
  FROM databox.invoices i
  INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
  WHERE im.end_date IS NULL
  GROUP BY i.invoices_idcor_account
)
SELECT
  a.accounts_company_name_1 AS client,
  dc.derniere_cde,
  CASE WHEN dc.derniere_cde < NOW() - INTERVAL '3 months' THEN 1 ELSE 0 END AS risque_inactif,
  CASE WHEN ce.ca_n1 > 0 AND ce.ca_n < ce.ca_n1 * 0.8 THEN 1 ELSE 0 END AS risque_baisse_ca,
  CASE WHEN dc.derniere_cde < NOW() - INTERVAL '3 months' THEN 1 ELSE 0 END
    + CASE WHEN ce.ca_n1 > 0 AND ce.ca_n < ce.ca_n1 * 0.8 THEN 1 ELSE 0 END AS score_risque
FROM derniere_commande dc
LEFT JOIN ca_evolution ce ON dc.idcor = ce.idcor
LEFT JOIN databox.id_mapping acc_im ON dc.idcor = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
ORDER BY score_risque DESC, dc.derniere_cde ASC;
```

**Points clés :**
- Deux CTE pour calculer chaque signal indépendamment
- Score combiné = somme des risques (0-2, extensible à 3 avec encours quand open_items sera alimenté)
- Seuils paramétrables (3 mois inactivité, -20% CA)
