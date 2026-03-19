# Schéma de base de données Databox — Référence complète

> **Usage** : Cette documentation est destinée à permettre l'écriture de requêtes SQL autonomes sur la base Databox. Elle couvre la structure des tables, les conventions de nommage, les relations entre entités et les patterns de requêtes éprouvés.

---

## 1. Vue d'ensemble

- **Schéma PostgreSQL** : `databox`
- Toutes les tables sont préfixées par leur domaine (ex : `invoices_*`, `sales_orders_*`)
- Chaque ligne de chaque table est **versionnée** via la table centrale `id_mapping` (voir `docs/ID-MAPPING.md`)
- Les identifiants cross-entités utilisent le suffixe `_idcor_*` (ex : `invoices_idcor_account`) — ce sont des `id_correspondence` stables dans le temps

---

## 2. Conventions générales

### 2.1 Règle d'or : filtrer sur la version courante

**Toujours** joindre `id_mapping` et filtrer `end_date IS NULL` pour ne voir que les enregistrements courants :

```sql
SELECT *
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL;
```

### 2.2 Colonnes systématiquement présentes dans chaque table

| Colonne                      | Description |
|------------------------------|-------------|
| `{entity}_id_databox`        | PK de la table = FK vers `id_mapping.id_databox` |
| `{entity}_inserted_at`       | Date d'insertion dans Databox |
| `erp_{entity}_id`            | ID dans le système ERP source |
| `erp_{entity}_custom_fields` | Champs custom ERP (JSONB) |
| `plm_{entity}_id`            | ID dans le système PLM |
| `plm_{entity}_custom_fields` | Champs custom PLM (JSONB) |
| `crm_{entity}_id`            | ID dans le système CRM |
| `crm_{entity}_custom_fields` | Champs custom CRM (JSONB) |

### 2.3 Relations cross-entités via `idcor`

Les relations entre entités n'utilisent **pas** de FK directes, mais des colonnes `*_idcor_*` contenant l'`id_correspondence` de l'entité cible. Pour résoudre un `idcor` en ligne concrète, il faut passer par `id_mapping` :

```sql
-- Résoudre invoices_idcor_account → ligne accounts
LEFT JOIN databox.id_mapping acc_im
    ON i.invoices_idcor_account = acc_im.id_correspondence
    AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a
    ON acc_im.id_databox = a.accounts_id_databox
```

### 2.4 Jointure article via lignes (cas spécial)

Les colonnes `*_id_product` dans les tables de lignes contiennent l'`id_databox` de l'article **en texte**. La jointure doit caster :

```sql
LEFT JOIN databox.articles art
    ON sol.sales_orders_lines_id_product = art.articles_id_databox::text
LEFT JOIN databox.id_mapping art_im
    ON art_im.id_databox = art.articles_id_databox
```

---

## 3. Tables par domaine

### 3.1 Identification

#### `id_mapping`
| Colonne             | Type        | Description |
|---------------------|-------------|-------------|
| `id_databox`        | BIGSERIAL PK | Identifiant interne, unique par version |
| `id_correspondence` | TEXT UNIQUE  | Identifiant stable dans le système source |
| `start_date`        | TIMESTAMP    | Début de validité |
| `end_date`          | TIMESTAMP    | Fin de validité — `NULL` = version courante |
| `src_system_type`   | TEXT         | Système source (`ERP`, `CRM`, `PLM`…) |
| `tgt_system_type`   | TEXT         | Système cible |

---

### 3.2 Tiers

#### `accounts` — Comptes clients
| Colonne clé                    | Description |
|--------------------------------|-------------|
| `accounts_id_databox`          | PK / FK id_mapping |
| `accounts_account_code`        | Code compte (identifiant métier unique) |
| `accounts_company_name_1/2`    | Raison sociale |
| `accounts_category`            | Catégorie |
| `accounts_account_type`        | Type de compte |
| `accounts_is_active`           | Actif/inactif |
| `accounts_currency`            | Devise |
| `accounts_payment_term`        | Conditions de paiement |
| `accounts_tax_rule`            | Règle de TVA |
| `accounts_country`             | Pays |
| `accounts_id_billing_account`  | ID compte de facturation (interne) |
| `accounts_id_pay_by_account`   | ID compte de paiement (interne) |
| `accounts_id_account_risk`     | ID compte risque (interne) |
| `accounts_id_account_group`    | ID groupe de comptes (interne) |

#### `accounts_address_lines` — Adresses
| Colonne clé                              | Description |
|------------------------------------------|-------------|
| `accounts_address_lines_id`              | PK / FK id_mapping |
| `accounts_id_databox`                    | FK → accounts |
| `accounts_address_lines_address_code`    | Code adresse |
| `accounts_address_lines_address_type`    | Type d'adresse |

#### `accounts_contact_lines` — Contacts
| Colonne clé                              | Description |
|------------------------------------------|-------------|
| `accounts_contact_lines_id`              | PK / FK id_mapping |
| `accounts_id_databox`                    | FK → accounts |
| `accounts_contact_lines_contact_code`    | Code contact |

#### `carriers` — Transporteurs
Structure similaire à `accounts` avec `carriers_id_databox` + lignes d'adresses et contacts.

#### `suppliers` — Fournisseurs
Structure similaire à `accounts` avec `suppliers_id_databox` + lignes d'adresses et contacts.
Nom du fournisseur : `suppliers_name` (pas `suppliers_company_name`).

---

### 3.3 Articles & Nomenclatures

#### `articles`
| Colonne clé                        | Description |
|------------------------------------|-------------|
| `articles_id_databox`              | PK / FK id_mapping |
| `articles_product_code`            | Code article (identifiant métier) |
| `articles_product_designation`     | Désignation |
| `articles_product_category`        | Catégorie produit |
| `articles_family_code`             | Code famille (array) |
| `articles_stock_unit`              | Unité de stock |
| `articles_sales_unit`              | Unité de vente |

#### `bom` — Nomenclatures standard
| Colonne clé              | Description |
|--------------------------|-------------|
| `bom_id_databox`         | PK / FK id_mapping |
| `bom_bom_code`           | Code nomenclature |

#### `bom_lines` — Lignes de nomenclature
| Colonne clé              | Description |
|--------------------------|-------------|
| `bom_lines_id`           | PK / FK id_mapping |
| `bom_id_databox`         | FK → bom |
| `bom_lines_id_component` | ID composant (interne) |
| `bom_lines_quantity`     | Quantité |

Variantes : `bom_commercial`, `bom_commercial_lines`, `bom_subcontracting`, `bom_subcontracting_material_lines`, `bom_subcontracting_service_lines`.

---

### 3.4 Cycle vente

#### `quotes` — Devis
| Colonne clé                          | Description |
|--------------------------------------|-------------|
| `quotes_id_databox`                  | PK / FK id_mapping |
| `quotes_quote_code`                  | Code devis (identifiant métier) |
| `quotes_idcor_account`               | `id_correspondence` du compte client |
| `quotes_quote_date`                  | Date du devis |
| `quotes_validity_date`               | Date de validité |
| `quotes_quote_status`                | Statut (`1`=ouvert, `2`=commandé, `3`=partiellement commandé) |
| `quotes_currency`                    | Devise |
| `quotes_currency_rate`               | Taux de change |
| `quotes_amount_lines_excl_vat`       | Montant HT |
| `quotes_amount_lines_incl_vat`       | Montant TTC |
| `quotes_total_margin`                | Marge totale |
| `quotes_idcor_sales_rep`             | `id_correspondence` du commercial |
| `quotes_opportunity`                 | Opportunité liée |
| `quotes_project`                     | Projet lié |

#### `quotes_lines` — Lignes de devis
| Colonne clé                          | Description |
|--------------------------------------|-------------|
| `quotes_lines_id`                    | PK / FK id_mapping |
| `quotes_id_databox`                  | FK → quotes |
| `quotes_lines_idcor_product`         | `id_correspondence` de l'article |
| `quotes_lines_id_product`            | `id_databox` article en TEXT (pour jointure) |
| `quotes_lines_product_designation`   | Désignation article |
| `quotes_lines_quantity`              | Quantité |
| `quotes_lines_gross_price`           | Prix brut |
| `quotes_lines_net_price`             | Prix net |
| `quotes_lines_amount_excl_vat`       | Montant ligne HT |
| `quotes_lines_amount_incl_vat`       | Montant ligne TTC |
| `quotes_lines_family_code`           | Famille produit (array) |

#### `sales_orders` — Commandes clients
| Colonne clé                                      | Description |
|--------------------------------------------------|-------------|
| `sales_orders_id_databox`                        | PK / FK id_mapping |
| `sales_orders_order_code`                        | Code commande |
| `sales_orders_idcor_sold_to_account`             | `id_correspondence` client vendu à |
| `sales_orders_idcor_bill_to_account`             | `id_correspondence` client facturé |
| `sales_orders_order_date`                        | Date commande |
| `sales_orders_currency`                          | Devise |
| `sales_orders_currency_rate`                     | Taux de change |
| `sales_orders_amount_lines_excl_vat`             | Montant total HT |
| `sales_orders_amount_lines_incl_vat`             | Montant total TTC |
| `sales_orders_amount_to_deliver_excluding_vat`   | Reste à livrer HT |
| `sales_orders_total_margin`                      | Marge totale |
| `sales_orders_idcor_quote`                       | `id_correspondence` du devis source |
| `sales_orders_quote_code`                        | Code du devis source |
| `sales_orders_order_status`                      | Statut commande |
| `sales_orders_delivery_status`                   | Statut livraison |
| `sales_orders_invoice_status`                    | Statut facturation |
| `sales_orders_idcor_sales_rep`                   | `id_correspondence` du commercial |

#### `sales_orders_lines` — Lignes de commande
| Colonne clé                                   | Description |
|-----------------------------------------------|-------------|
| `sales_orders_lines_id`                       | PK / FK id_mapping |
| `sales_orders_id_databox`                     | FK → sales_orders |
| `sales_orders_lines_idcor_product`            | `id_correspondence` article |
| `sales_orders_lines_id_product`               | `id_databox` article en TEXT |
| `sales_orders_lines_product_designation`      | Désignation |
| `sales_orders_lines_ordered_quantity`         | Quantité commandée |
| `sales_orders_lines_delivered_quantity`       | Quantité livrée |
| `sales_orders_lines_invoiced_quantity`        | Quantité facturée |
| `sales_orders_lines_quantity_to_deliver`      | Reste à livrer |
| `sales_orders_lines_gross_price`              | Prix brut |
| `sales_orders_lines_net_price`                | Prix net |
| `sales_orders_lines_asked_delivery_date`      | Date de livraison demandée |
| `sales_orders_lines_shipment_date`            | Date d'expédition |
| `sales_orders_lines_idcor_quote_line`         | Lien vers ligne de devis source |

#### `sales_reps` — Commerciaux
| Colonne clé              | Description |
|--------------------------|-------------|
| `sales_reps_id_databox`  | PK / FK id_mapping |
| `sales_reps_rep_code`    | Code commercial |
| `sales_reps_name`        | Nom |

#### `deliveries` — Livraisons
| Colonne clé                             | Description |
|-----------------------------------------|-------------|
| `deliveries_id_databox`                 | PK / FK id_mapping |
| `deliveries_delivery_code`              | Code livraison |
| `deliveries_idcor_sold_to_account`      | `id_correspondence` client |
| `deliveries_shipment_date`              | Date d'expédition |
| `deliveries_delivery_date`              | Date de livraison |
| `deliveries_is_validated`               | Validée |
| `deliveries_is_invoiced`                | Facturée |
| `deliveries_carrier`                    | Transporteur |

#### `deliveries_lines` — Lignes de livraison
| Colonne clé                                   | Description |
|-----------------------------------------------|-------------|
| `deliveries_lines_id`                         | PK / FK id_mapping |
| `deliveries_id_databox`                       | FK → deliveries |
| `deliveries_lines_idcor_product`              | `id_correspondence` article |
| `deliveries_lines_id_product`                 | `id_databox` article en TEXT |
| `deliveries_lines_delivered_quantity`         | Quantité livrée |
| `deliveries_lines_returned_quantity`          | Quantité retournée |
| `deliveries_lines_sales_order_code`           | Code commande source |
| `deliveries_lines_idcor_sales_order_line`     | Lien ligne commande source |

#### `invoices` — Factures
| Colonne clé                          | Description |
|--------------------------------------|-------------|
| `invoices_id_databox`                | PK / FK id_mapping |
| `invoices_invoice_code`              | Code facture |
| `invoices_idcor_account`             | `id_correspondence` client |
| `invoices_invoice_date`              | Date de facture |
| `invoices_invoice_type`              | Type (avoir, facture...) |
| `invoices_is_validated`              | Validée |
| `invoices_currency`                  | Devise |
| `invoices_currency_rate`             | Taux de change |
| `invoices_amount_lines_excl_vat`     | Montant total HT |
| `invoices_amount_lines_incl_vat`     | Montant total TTC |
| `invoices_sign`                      | Signe comptable (+1/-1) |
| `invoices_idcor_sales_rep`           | `id_correspondence` commercial |

#### `invoices_lines` — Lignes de facture
| Colonne clé                          | Description |
|--------------------------------------|-------------|
| `invoices_lines_id`                  | PK / FK id_mapping |
| `invoices_id_databox`                | FK → invoices |
| `invoices_lines_idcor_product`       | `id_correspondence` article |
| `invoices_lines_id_product`          | `id_databox` article en TEXT |
| `invoices_lines_product_designation` | Désignation |
| `invoices_lines_quantity`            | Quantité |
| `invoices_lines_gross_price`         | Prix brut |
| `invoices_lines_net_price`           | Prix net |
| `invoices_lines_amount_excl_vat`     | Montant ligne HT |
| `invoices_lines_amount_incl_vat`     | Montant ligne TTC |
| `invoices_lines_line_margin`         | Marge ligne |
| `invoices_lines_delivery_code`       | Code livraison source |

---

### 3.5 Cycle achat

#### `purchase_orders` — Commandes fournisseurs
| Colonne clé                                | Description |
|--------------------------------------------|-------------|
| `purchase_orders_id_databox`               | PK / FK id_mapping |
| `purchase_orders_code`                     | Code commande |
| `purchase_orders_idcor_supplier`           | `id_correspondence` fournisseur |
| `purchase_orders_order_date`               | Date commande |
| `purchase_orders_total_amount_excl_vat`    | Montant HT |
| `purchase_orders_order_status`             | Statut |
| `purchase_orders_receipt_status`           | Statut réception |
| `purchase_orders_invoice_status`           | Statut facturation |

#### `purchase_orders_lines` — Lignes commandes fournisseurs
| Colonne clé                                        | Description |
|----------------------------------------------------|-------------|
| `purchase_orders_lines_id`                         | PK / FK id_mapping |
| `purchase_orders_id_databox`                       | FK → purchase_orders |
| `purchase_orders_lines_idcor_product`              | `id_correspondence` article |
| `purchase_orders_lines_ordered_purchase_quantity`  | Quantité commandée |
| `purchase_orders_lines_received_purchase_quantity` | Quantité reçue |
| `purchase_orders_lines_invoiced_quantity`          | Quantité facturée |
| `purchase_orders_lines_gross_price`                | Prix brut |
| `purchase_orders_lines_net_price`                  | Prix net |

#### `purchase_invoices` — Factures fournisseurs
Structure similaire à `invoices` avec `purchase_invoices_idcor_supplier`.

#### `receipts` — Réceptions
| Colonne clé                         | Description |
|-------------------------------------|-------------|
| `receipts_id_databox`               | PK / FK id_mapping |
| `receipts_receipt_code`             | Code réception |
| `receipts_idcor_supplier`           | `id_correspondence` fournisseur |
| `receipts_receipt_date`             | Date de réception |

---

### 3.6 Production

#### `manufacturing_orders` — Ordres de fabrication
| Colonne clé                                  | Description |
|----------------------------------------------|-------------|
| `manufacturing_orders_id_databox`            | PK / FK id_mapping |
| `manufacturing_orders_order_code`            | Code OF |
| `manufacturing_orders_idcor_product`         | `id_correspondence` article fabriqué |
| `manufacturing_orders_order_date`            | Date OF |
| `manufacturing_orders_planned_start_date`    | Date début prévue |
| `manufacturing_orders_planned_end_date`      | Date fin prévue |
| `manufacturing_orders_status`                | Statut |

Lignes de l'OF : `manufactured_products_lines`, `components_lines`, `operations_lines`, `means_lines`, `authorizations_lines` — toutes avec FK → `manufacturing_orders`.

---

### 3.7 Projets & Affaires

#### `projects`
| Colonne clé                      | Description |
|----------------------------------|-------------|
| `projects_id_databox`            | PK / FK id_mapping |
| `projects_project_code`          | Code projet |
| `projects_idcor_account`         | Client associé |

Lignes : `projects_tasks_lines`, `projects_tasks_operations_lines`.

---

### 3.8 Qualité

#### `quality_events`
| Colonne clé                        | Description |
|------------------------------------|-------------|
| `quality_events_id_databox`        | PK / FK id_mapping |
| `quality_events_event_code`        | Code événement |
| `quality_events_idcor_account`     | Client associé |

Lignes : `actions_lines`.

---

### 3.9 Autres

- `documents` — Documents GED/PLM
- `open_items` — Encours/écheances clients
- `customer_assets` + `customer_assets_lines` — Équipements client
- `customer_due_dates` — Échéances clients
- `service_contracts` + `service_contracts_lines` — Contrats de service
- `opportunities` — Opportunités CRM

---

## 4. Patterns de requêtes SQL

### 4.1 Version courante d'une entité

```sql
-- Tous les devis courants
SELECT q.*, im.id_correspondence, im.start_date, im.src_system_type
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE im.end_date IS NULL;
```

### 4.2 Entité par id_correspondence (code stable)

```sql
-- Un compte par son id_correspondence
SELECT a.*, im.id_correspondence
FROM databox.accounts a
INNER JOIN databox.id_mapping im ON a.accounts_id_databox = im.id_databox
WHERE im.id_correspondence = 'COR-ACC-001'
  AND im.end_date IS NULL;
```

### 4.3 Entité avec ses lignes

```sql
-- Factures avec leurs lignes (versions courantes seulement)
SELECT i.invoices_invoice_code,
       i.invoices_invoice_date,
       i.invoices_amount_lines_excl_vat,
       il.invoices_lines_idcor_product,
       il.invoices_lines_product_designation,
       il.invoices_lines_quantity,
       il.invoices_lines_amount_excl_vat
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
INNER JOIN databox.invoices_lines il ON i.invoices_id_databox = il.invoices_id_databox
WHERE im.end_date IS NULL;
```

### 4.4 Filtrer par compte client (via idcor)

```sql
-- Commandes d'un client donné sur une période
SELECT so.sales_orders_order_code,
       so.sales_orders_order_date,
       so.sales_orders_amount_lines_excl_vat
FROM databox.sales_orders so
INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
WHERE so.sales_orders_idcor_sold_to_account = 'COR-ACC-001'
  AND im.end_date IS NULL
  AND so.sales_orders_order_date >= '2024-01-01'
  AND so.sales_orders_order_date < '2025-01-01';
```

### 4.5 Jointure entité → compte (résolution idcor)

```sql
-- Factures avec nom du client
SELECT i.invoices_invoice_code,
       i.invoices_invoice_date,
       a.accounts_account_code,
       a.accounts_company_name1,
       i.invoices_amount_lines_excl_vat
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
LEFT JOIN databox.id_mapping acc_im
    ON i.invoices_idcor_account = acc_im.id_correspondence
    AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE im.end_date IS NULL;
```

### 4.6 Jointure lignes → article (via id_product TEXT)

```sql
-- Top articles commandés en quantité
SELECT art.articles_product_code,
       art_im.id_correspondence AS article_idcor,
       sol.sales_orders_lines_product_designation,
       SUM(sol.sales_orders_lines_ordered_quantity) AS total_qty
FROM databox.sales_orders_lines sol
LEFT JOIN databox.articles art
    ON sol.sales_orders_lines_id_product = art.articles_id_databox::text
LEFT JOIN databox.id_mapping art_im
    ON art_im.id_databox = art.articles_id_databox
INNER JOIN databox.sales_orders so ON sol.sales_orders_id_databox = so.sales_orders_id_databox
INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
WHERE im.end_date IS NULL
GROUP BY art.articles_product_code, art_im.id_correspondence, sol.sales_orders_lines_product_designation
ORDER BY total_qty DESC;
```

### 4.7 Agrégation par compte

```sql
-- CA facturé par client sur une période
SELECT i.invoices_idcor_account,
       a.accounts_account_code,
       a.accounts_company_name1,
       COUNT(DISTINCT i.invoices_id_databox) AS nb_factures,
       COALESCE(SUM(i.invoices_amount_lines_excl_vat), 0) AS total_ht,
       COALESCE(SUM(i.invoices_amount_lines_incl_vat), 0) AS total_ttc
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
LEFT JOIN databox.id_mapping acc_im
    ON i.invoices_idcor_account = acc_im.id_correspondence
    AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE im.end_date IS NULL
  AND i.invoices_invoice_date >= '2024-01-01'
  AND i.invoices_invoice_date < '2025-01-01'
GROUP BY i.invoices_idcor_account, a.accounts_account_code, a.accounts_company_name1
ORDER BY total_ht DESC;
```

### 4.8 Agrégation temporelle avec generate_series (toutes périodes incluses)

```sql
-- CA facturé par mois, sans trous de données
SELECT
    periods.period,
    COALESCE(COUNT(DISTINCT i.invoices_id_databox), 0) AS nb_factures,
    COALESCE(SUM(i.invoices_amount_lines_excl_vat), 0) AS total_ht
FROM generate_series(
    DATE_TRUNC('month', '2024-01-01'::timestamp),
    DATE_TRUNC('month', '2024-12-31'::timestamp),
    '1 month'::interval
) AS periods(period)
LEFT JOIN databox.invoices i
    ON DATE_TRUNC('month', i.invoices_invoice_date) = periods.period
    AND i.invoices_invoice_date >= '2024-01-01'
    AND i.invoices_invoice_date < '2025-01-01'
LEFT JOIN databox.id_mapping im
    ON i.invoices_id_databox = im.id_databox
    AND im.end_date IS NULL
GROUP BY periods.period
ORDER BY periods.period ASC;
```

### 4.9 Taux de conversion devis → commande

```sql
-- Nombre de devis convertis (status 2 ou 3) vs total
SELECT
    COUNT(*) FILTER (WHERE im.end_date IS NULL) AS total_devis,
    COUNT(*) FILTER (WHERE im.end_date IS NULL AND q.quotes_quote_status IN ('2', '3')) AS devis_convertis,
    ROUND(
        100.0 * COUNT(*) FILTER (WHERE im.end_date IS NULL AND q.quotes_quote_status IN ('2', '3'))
        / NULLIF(COUNT(*) FILTER (WHERE im.end_date IS NULL), 0), 2
    ) AS taux_conversion_pct
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE q.quotes_idcor_account = ANY(ARRAY['COR-001', 'COR-002'])
  AND q.quotes_quote_date >= '2024-01-01'
  AND q.quotes_quote_date < '2025-01-01';
```

### 4.10 Montant factures semaine par semaine pour un compte

```sql
-- Issu de InvoiceService.findAllForAccountDateRange
SELECT
    SUM(i.invoices_total) AS total,
    DATE_TRUNC('week', i.invoices_invoice_date) AS week
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL
  AND i.invoices_id_account = 'ACC-INTERNAL-123'
  AND im.start_date >= '2024-01-01'
  AND im.start_date < '2025-01-01'
GROUP BY DATE_TRUNC('week', i.invoices_invoice_date)
ORDER BY week DESC;
```

### 4.11 Historique complet d'une entité

```sql
-- Toutes les versions d'un devis (dont les archivées)
SELECT q.*, im.start_date, im.end_date, im.id_correspondence
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE im.id_correspondence = 'COR-QUOTE-456'
ORDER BY im.start_date ASC;
```

### 4.12 Activité d'un article sur le cycle vente complet

```sql
-- Résumé devis + commandes + livraisons + factures pour un article
WITH article_id AS (
    SELECT im.id_databox
    FROM databox.id_mapping im
    WHERE im.id_correspondence = 'COR-ART-789'
      AND im.end_date IS NULL
)
SELECT
    'quotes'     AS type,
    COUNT(DISTINCT q.quotes_id_databox) AS count,
    SUM(ql.quotes_lines_amount_excl_vat) AS total_ht
FROM databox.quotes_lines ql
INNER JOIN databox.quotes q ON ql.quotes_id_databox = q.quotes_id_databox
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE ql.quotes_lines_idcor_product = 'COR-ART-789'
  AND im.end_date IS NULL

UNION ALL

SELECT
    'sales_orders' AS type,
    COUNT(DISTINCT so.sales_orders_id_databox),
    SUM(sol.sales_orders_lines_net_price * sol.sales_orders_lines_ordered_quantity)
FROM databox.sales_orders_lines sol
INNER JOIN databox.sales_orders so ON sol.sales_orders_id_databox = so.sales_orders_id_databox
INNER JOIN databox.id_mapping im ON so.sales_orders_id_databox = im.id_databox
WHERE sol.sales_orders_lines_idcor_product = 'COR-ART-789'
  AND im.end_date IS NULL

UNION ALL

SELECT
    'invoices' AS type,
    COUNT(DISTINCT i.invoices_id_databox),
    SUM(il.invoices_lines_amount_excl_vat)
FROM databox.invoices_lines il
INNER JOIN databox.invoices i ON il.invoices_id_databox = i.invoices_id_databox
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE il.invoices_lines_idcor_product = 'COR-ART-789'
  AND im.end_date IS NULL;
```

---

## 5. Résumé des relations clés

```
accounts ←──────── quotes          (quotes_idcor_account)
accounts ←──────── sales_orders    (sales_orders_idcor_sold_to_account)
accounts ←──────── invoices        (invoices_idcor_account)
accounts ←──────── deliveries      (deliveries_idcor_sold_to_account)
accounts ←──────── projects        (projects_idcor_account)

quotes   ←──────── sales_orders    (sales_orders_idcor_quote / quote_code)
quotes   ←──────── quotes_lines    (FK directe)

sales_orders ←──── sales_orders_lines  (FK directe)
sales_orders_lines ←── deliveries_lines (idcor_sales_order_line)
invoices ←──────── invoices_lines      (FK directe)
deliveries ←─────── deliveries_lines   (FK directe)

articles ←──────── quotes_lines           (idcor_product / id_product::text)
articles ←──────── sales_orders_lines     (idcor_product / id_product::text)
articles ←──────── invoices_lines         (idcor_product / id_product::text)
articles ←──────── deliveries_lines       (idcor_product / id_product::text)
articles ←──────── purchase_orders_lines  (idcor_product)
articles ←──────── manufacturing_orders   (idcor_product)

suppliers ←──────── purchase_orders    (idcor_supplier)
suppliers ←──────── purchase_invoices  (idcor_supplier)
suppliers ←──────── receipts           (idcor_supplier)
```

---

## 6. Points d'attention

1. **Jointure article** : utiliser `id_product::text` pour joindre via l'id interne, ou `idcor_product` pour joindre via `id_mapping.id_correspondence`. Les deux coexistent.

2. **Alias `id_mapping` obligatoire** quand on joint plusieurs fois la même table (ex : résoudre à la fois l'article et le compte dans la même requête) :
   ```sql
   INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
   LEFT JOIN databox.id_mapping acc_im ON i.invoices_idcor_account = acc_im.id_correspondence
   ```

3. **Montants multi-devises** : les montants sont stockés dans la devise du document. Pour comparer des montants, multiplier par `currency_rate` pour ramener en devise de référence :
   ```sql
   SUM(quotes_amount_lines_excl_vat * quotes_currency_rate)
   ```

4. **Statut devis** : `quotes_quote_status` vaut `'2'` (commandé) ou `'3'` (partiellement commandé) pour les devis convertis en commande.

5. **`invoices_sign`** : vaut `-1` pour les avoirs, `+1` pour les factures normales. Tenir compte de ce signe dans les agrégations comptables.
