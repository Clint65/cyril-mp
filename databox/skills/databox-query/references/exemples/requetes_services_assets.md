# Exemples NL→SQL — Services, Parc installé, Production & Cross-domaine

> Requêtes couvrant les tables service_contracts, customer_assets, production enrichie et requêtes transversales projet/client 360°.

---

## Contrats de service

### Exemple 1 : Contrats actifs par client

**Question :** "Quels clients ont un contrat de maintenance actif ?"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  sc.service_contracts_contract_code,
  sc.service_contracts_contract_designation,
  sc.service_contracts_start_date,
  sc.service_contracts_end_date,
  sc.service_contracts_annual_royalty AS montant_annuel
FROM databox.service_contracts sc
INNER JOIN databox.id_mapping sc_im ON sc.service_contracts_id_databox = sc_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON sc.service_contracts_idcor_sold_to_account = acc_im.id_correspondence
  AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE sc_im.end_date IS NULL
  AND sc.service_contracts_is_canceled = false
  AND sc.service_contracts_is_closed = false
  AND sc.service_contracts_end_date >= NOW()
ORDER BY a.accounts_company_name_1;
```

**Points clés :**
- Trois critères pour un contrat "actif" : pas annulé, pas clos, date de fin dans le futur
- Résolution client via `idcor_sold_to_account` → `id_mapping` → `accounts`

---

### Exemple 2 : CA récurrent par client

**Question :** "Quel est le CA récurrent annuel par client ?"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  COUNT(*) AS nb_contrats,
  SUM(sc.service_contracts_annual_royalty) AS ca_recurrent
FROM databox.service_contracts sc
INNER JOIN databox.id_mapping sc_im ON sc.service_contracts_id_databox = sc_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON sc.service_contracts_idcor_sold_to_account = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE sc_im.end_date IS NULL
  AND sc.service_contracts_is_canceled = false
  AND sc.service_contracts_is_closed = false
GROUP BY a.accounts_company_name_1
ORDER BY ca_recurrent DESC;
```

---

## Parc installé

### Exemple 3 : Parc installé par client

**Question :** "Quel est le parc installé chez mes clients ?"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  ca.customer_assets_asset_designation AS equipement,
  ca.customer_assets_brand AS marque,
  ca.customer_assets_quantity AS quantite,
  ca.customer_assets_start_date AS date_installation
FROM databox.customer_assets ca
INNER JOIN databox.id_mapping ca_im ON ca.customer_assets_id_databox = ca_im.id_databox
INNER JOIN databox.customer_assets_lines cal ON cal.customer_assets_id_databox = ca.customer_assets_id_databox
LEFT JOIN databox.id_mapping acc_im
  ON cal.customer_assets_lines_idcor_account = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE ca_im.end_date IS NULL
ORDER BY a.accounts_company_name_1, ca.customer_assets_start_date DESC;
```

**Points clés :**
- Le lien client est sur `customer_assets_lines` (pas sur `customer_assets` directement)
- Un même asset peut être affecté à plusieurs clients (via plusieurs lignes)

---

### Exemple 4 : Actifs sans contrat de maintenance

**Question :** "Quels équipements client n'ont pas de contrat de maintenance ?"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  ca.customer_assets_asset_designation AS equipement,
  ca.customer_assets_quantity AS quantite
FROM databox.customer_assets ca
INNER JOIN databox.id_mapping ca_im ON ca.customer_assets_id_databox = ca_im.id_databox
INNER JOIN databox.customer_assets_lines cal ON cal.customer_assets_id_databox = ca.customer_assets_id_databox
LEFT JOIN databox.id_mapping acc_im
  ON cal.customer_assets_lines_idcor_account = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE ca_im.end_date IS NULL
  AND ca.customer_assets_idcor_service_contract_line IS NULL
ORDER BY a.accounts_company_name_1;
```

**Points clés :**
- `customer_assets_idcor_service_contract_line IS NULL` = pas de lien vers un contrat

---

## Open items enrichis

### Exemple 5 : Encours par type de document

**Question :** "Quel est l'encours ventilé par type de document ?"

**SQL :**
```sql
SELECT
  oi.open_items_business_partner_type AS partenaire,
  oi.open_items_document_type AS type_doc,
  CASE
    WHEN oi.open_items_business_partner_type = 'customer' AND oi.open_items_document_type = '*SO' THEN 'Commande client'
    WHEN oi.open_items_business_partner_type = 'customer' THEN 'Facture client'
    WHEN oi.open_items_business_partner_type = 'supplier' AND oi.open_items_document_type = '*PO' THEN 'Commande fournisseur'
    WHEN oi.open_items_business_partner_type = 'supplier' THEN 'Facture fournisseur'
  END AS libelle,
  COUNT(*) AS nb_echeances,
  SUM((oi.open_items_amount_in_company_currency - oi.open_items_paid_amount_in_company_currency)
    * oi.open_items_sign) AS encours
FROM databox.open_items oi
INNER JOIN databox.id_mapping im ON oi.open_items_id_databox = im.id_databox
WHERE im.end_date IS NULL
GROUP BY oi.open_items_business_partner_type, oi.open_items_document_type
ORDER BY encours DESC;
```

**Points clés :**
- 4 types d'open_items identifiables par la combinaison `business_partner_type` + `document_type`
- `open_items_id_document` et `open_items_idcor_document` permettent de remonter au document source
- `open_items_document_line_number` ordonne les échéances multiples d'un même document

---

### Exemple 6 : Échéances d'un document avec multi-lignes

**Question :** "Quelles sont les échéances de la commande X ?"

**SQL :**
```sql
SELECT
  oi.open_items_document_number,
  oi.open_items_document_line_number AS echeance_num,
  oi.open_items_amount_in_currency AS montant,
  oi.open_items_paid_amount_in_currency AS paye,
  oi.open_items_payment_date AS date_echeance,
  oi.open_items_closed_status AS statut
FROM databox.open_items oi
INNER JOIN databox.id_mapping im ON oi.open_items_id_databox = im.id_databox
WHERE im.end_date IS NULL
  AND oi.open_items_idcor_document = 'CMD-XXXX'  -- remplacer par le code commande
ORDER BY oi.open_items_document_line_number;
```

---

## Production enrichie

### Exemple 7 : Taux de rejet par poste de travail

**Question :** "Quel est le taux de rejet par poste de production ?"

**SQL :**
```sql
SELECT
  ol.operations_lines_workstation_code AS poste,
  ol.operations_lines_designation AS operation,
  SUM(ol.operations_lines_produced_quantity) AS total_produit,
  SUM(ol.operations_lines_rejected_quantity) AS total_rejete,
  ROUND(100.0 * SUM(ol.operations_lines_rejected_quantity)
    / NULLIF(SUM(ol.operations_lines_expected_quantity), 0), 2) AS taux_rejet_pct
FROM databox.operations_lines ol
INNER JOIN databox.manufacturing_orders mo ON ol.manufacturing_orders_id_databox = mo.manufacturing_orders_id_databox
INNER JOIN databox.id_mapping im ON mo.manufacturing_orders_id_databox = im.id_databox
WHERE im.end_date IS NULL
GROUP BY ol.operations_lines_workstation_code, ol.operations_lines_designation
ORDER BY taux_rejet_pct DESC;
```

---

### Exemple 8 : Temps de cycle par opération

**Question :** "Quel est le temps moyen par opération sur la ligne de production ?"

**SQL :**
```sql
SELECT
  ol.operations_lines_workstation_code AS poste,
  ol.operations_lines_designation AS operation,
  COUNT(*) AS nb_passages,
  ROUND(AVG(ol.operations_lines_expected_run_time)::numeric, 2) AS temps_moyen_min
FROM databox.operations_lines ol
INNER JOIN databox.manufacturing_orders mo ON ol.manufacturing_orders_id_databox = mo.manufacturing_orders_id_databox
INNER JOIN databox.id_mapping im ON mo.manufacturing_orders_id_databox = im.id_databox
WHERE im.end_date IS NULL
GROUP BY ol.operations_lines_workstation_code, ol.operations_lines_designation
ORDER BY MIN(ol.operations_lines_number);
```

---

## Cross-domaine

### Exemple 9 : Vue 360° projet — tous les documents liés

**Question :** "Montre-moi tous les documents liés au projet X"

**SQL :**
```sql
SELECT 'Devis' AS type, im.id_correspondence AS code, q.quotes_quote_date AS date_doc, q.quotes_amount_lines_excl_vat AS montant
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE im.end_date IS NULL AND q.quotes_idcor_project = 'PRJ-XXXX'

UNION ALL
SELECT 'Commande', im.id_correspondence, so.sales_orders_order_date, so.sales_orders_amount_lines_excl_vat
FROM databox.sales_orders so
INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
WHERE im.end_date IS NULL AND so.sales_orders_idcor_project = 'PRJ-XXXX'

UNION ALL
SELECT 'Facture', im.id_correspondence, i.invoices_invoice_date, i.invoices_amount_lines_excl_vat * i.invoices_sign
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL AND i.invoices_idcor_project = 'PRJ-XXXX'

UNION ALL
SELECT 'Commande achat', im.id_correspondence, po.purchase_orders_order_date, po.purchase_orders_total_amount_excl_vat
FROM databox.purchase_orders po
INNER JOIN databox.id_mapping im ON po.purchase_orders_id_databox = im.id_databox
WHERE im.end_date IS NULL AND po.purchase_orders_idcor_project = 'PRJ-XXXX'

ORDER BY date_doc;
```

**Points clés :**
- `UNION ALL` pour croiser 4 tables de documents
- Chaque bloc filtre sur `idcor_project`
- Les factures utilisent `* invoices_sign` pour le montant

---

### Exemple 10 : Vue 360° client — activité complète

**Question :** "Résumé complet de l'activité d'un client"

**SQL :**
```sql
SELECT
  a.accounts_company_name_1 AS client,
  (SELECT COUNT(*) FROM databox.quotes q
   INNER JOIN databox.id_mapping qim ON q.quotes_id_databox = qim.id_databox
   WHERE qim.end_date IS NULL AND q.quotes_idcor_account = acc_im.id_correspondence) AS nb_devis,
  (SELECT COUNT(*) FROM databox.sales_orders so
   INNER JOIN databox.id_mapping sim ON so.sales_orders_id_databox = sim.id_databox
   WHERE sim.end_date IS NULL AND so.sales_orders_idcor_sold_to_account = acc_im.id_correspondence
     AND so.sales_orders_order_status IN (1, 2)) AS nb_commandes,
  (SELECT COALESCE(SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign), 0)
   FROM databox.invoices i
   INNER JOIN databox.id_mapping iim ON i.invoices_id_databox = iim.id_databox
   WHERE iim.end_date IS NULL AND i.invoices_idcor_account = acc_im.id_correspondence) AS ca_total,
  (SELECT COUNT(*) FROM databox.service_contracts sc
   INNER JOIN databox.id_mapping scim ON sc.service_contracts_id_databox = scim.id_databox
   WHERE scim.end_date IS NULL AND sc.service_contracts_idcor_sold_to_account = acc_im.id_correspondence
     AND sc.service_contracts_is_canceled = false) AS nb_contrats,
  (SELECT COALESCE(SUM(ca.customer_assets_quantity), 0)
   FROM databox.customer_assets ca
   INNER JOIN databox.id_mapping caim ON ca.customer_assets_id_databox = caim.id_databox
   INNER JOIN databox.customer_assets_lines cal ON cal.customer_assets_id_databox = ca.customer_assets_id_databox
   WHERE caim.end_date IS NULL AND cal.customer_assets_lines_idcor_account = acc_im.id_correspondence) AS nb_assets
FROM databox.accounts a
INNER JOIN databox.id_mapping acc_im ON a.accounts_id_databox = acc_im.id_databox
WHERE acc_im.end_date IS NULL
  AND a.accounts_company_name_1 = 'NOM_CLIENT';  -- remplacer
```

**Points clés :**
- Sous-requêtes corrélées pour agréger depuis chaque table liée
- Chaque sous-requête a son propre `id_mapping` avec `end_date IS NULL`
- Pattern extensible : ajouter d'autres sous-requêtes pour open_items, projets, etc.
