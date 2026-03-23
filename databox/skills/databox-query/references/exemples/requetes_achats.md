# Exemples NL→SQL — Domaines Achats, Production & Projets

> Requêtes validées sur la base Databox. Production validée avec données réelles (3 674 OF).
> Achats, réceptions et projets : syntaxe validée (tables vides dans le jeu actuel).

---

## Achats

### Exemple 1 : Top fournisseurs par montant d'achat

**Question :** "Quels sont mes principaux fournisseurs ?"

**SQL :**
```sql
SELECT
  s.suppliers_name AS fournisseur,
  COUNT(DISTINCT po.purchase_orders_id_databox) AS nb_commandes,
  SUM(po.purchase_orders_total_amount_excl_vat) AS total_ht
FROM databox.purchase_orders po
INNER JOIN databox.id_mapping doc_im ON po.purchase_orders_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping sup_im
  ON po.purchase_orders_idcor_supplier = sup_im.id_correspondence AND sup_im.end_date IS NULL
LEFT JOIN databox.suppliers s ON sup_im.id_databox = s.suppliers_id_databox
WHERE doc_im.end_date IS NULL
GROUP BY s.suppliers_name
ORDER BY total_ht DESC
LIMIT 10;
```

**Points clés :**
- Même pattern idcor que pour les comptes clients, mais via `idcor_supplier` → `suppliers`
- Nom fournisseur = `suppliers_name` (pas `company_name_1` comme pour accounts)
- Montant = `purchase_orders_total_amount_excl_vat` (pas `amount_lines_excl_vat`)

**Note :** Table `purchase_orders` non alimentée dans le jeu actuel.

---

### Exemple 2 : Écarts commande/réception fournisseur

**Question :** "Quelles commandes fournisseurs ont des écarts de réception ?"

**SQL :**
```sql
SELECT
  po.purchase_orders_code,
  s.suppliers_name AS fournisseur,
  SUM(pol.purchase_orders_lines_ordered_purchase_quantity) AS qty_commandee,
  SUM(pol.purchase_orders_lines_received_purchase_quantity) AS qty_recue,
  SUM(pol.purchase_orders_lines_ordered_purchase_quantity)
    - SUM(pol.purchase_orders_lines_received_purchase_quantity) AS ecart
FROM databox.purchase_orders_lines pol
INNER JOIN databox.purchase_orders po ON pol.purchase_orders_id_databox = po.purchase_orders_id_databox
INNER JOIN databox.id_mapping doc_im ON po.purchase_orders_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping sup_im
  ON po.purchase_orders_idcor_supplier = sup_im.id_correspondence AND sup_im.end_date IS NULL
LEFT JOIN databox.suppliers s ON sup_im.id_databox = s.suppliers_id_databox
WHERE doc_im.end_date IS NULL
GROUP BY po.purchase_orders_code, s.suppliers_name
HAVING SUM(pol.purchase_orders_lines_ordered_purchase_quantity)
  - SUM(pol.purchase_orders_lines_received_purchase_quantity) > 0
ORDER BY ecart DESC;
```

**Points clés :**
- Colonnes quantités : `_ordered_purchase_quantity` et `_received_purchase_quantity` (pas `_ordered_quantity`)
- `HAVING` pour ne garder que les commandes avec écart positif
- Code commande = `purchase_orders_code` (pas `order_code`)

---

## Production

### Exemple 3 : OF par statut avec décodage value_list

**Question :** "Combien d'ordres de fabrication par statut ?"

**SQL :**
```sql
SELECT
  mo.manufacturing_orders_status AS code,
  vll.label AS statut,
  COUNT(*) AS nb
FROM databox.manufacturing_orders mo
INNER JOIN databox.id_mapping im ON mo.manufacturing_orders_id_databox = im.id_databox
LEFT JOIN databox.value_list_column_config vlcc ON vlcc.column_name = 'manufacturing_orders_status'
LEFT JOIN databox.value_list_entry vle
  ON vle.value_list_id = vlcc.value_list_id
  AND vle.neutral_code = mo.manufacturing_orders_status::text
LEFT JOIN databox.value_list_label vll ON vll.entry_id = vle.id AND vll.locale = 'fr'
WHERE im.end_date IS NULL
GROUP BY mo.manufacturing_orders_status, vll.label
ORDER BY mo.manufacturing_orders_status;
```

**Points clés :**
- Colonne = `manufacturing_orders_status` (pas `order_status`)
- Cast `::text` sur le statut (integer) pour la jointure value_list_entry (neutral_code = text)
- Statuts : 1="En attente" (3 357), 2="En cours" (164), 4="Exclue" (153)

**Résultat :**

| code | statut | nb |
|------|--------|-----|
| 1 | En attente | 3 357 |
| 2 | En cours | 164 |
| 4 | Exclue | 153 |

---

### Exemple 4 : OF en retard

**Question :** "Quels ordres de fabrication sont en retard ?"

**SQL :**
```sql
SELECT
  mo.manufacturing_orders_code,
  mo.manufacturing_orders_end_date,
  vll.label AS statut
FROM databox.manufacturing_orders mo
INNER JOIN databox.id_mapping im ON mo.manufacturing_orders_id_databox = im.id_databox
LEFT JOIN databox.value_list_column_config vlcc ON vlcc.column_name = 'manufacturing_orders_status'
LEFT JOIN databox.value_list_entry vle
  ON vle.value_list_id = vlcc.value_list_id
  AND vle.neutral_code = mo.manufacturing_orders_status::text
LEFT JOIN databox.value_list_label vll ON vll.entry_id = vle.id AND vll.locale = 'fr'
WHERE im.end_date IS NULL
  AND mo.manufacturing_orders_end_date < NOW()
  AND mo.manufacturing_orders_status IN (1, 2)
ORDER BY mo.manufacturing_orders_end_date DESC;
```

**Points clés :**
- `end_date < NOW()` = date fin planifiée dépassée
- `status IN (1, 2)` = En attente ou En cours (pas terminé/exclu)
- Dates : `manufacturing_orders_start_date`, `manufacturing_orders_end_date`, `manufacturing_orders_closing_date`

---

## Projets

### Exemple 5 : Projets par client

**Question :** "Quels projets pour le client X ?"

**SQL :**
```sql
SELECT
  p.projects_project_code,
  a.accounts_company_name_1 AS client
FROM databox.projects p
INNER JOIN databox.id_mapping doc_im ON p.projects_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON p.projects_idcor_account = acc_im.id_correspondence AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE doc_im.end_date IS NULL
ORDER BY p.projects_project_code;
```

**Points clés :**
- Même pattern idcor que pour les factures/commandes
- Tables liées : `projects_tasks_lines`, `projects_tasks_operations_lines`

**Note :** Table `projects` non alimentée dans le jeu actuel.

---

## Différences de nommage vs DB-SCHEMA.md

| DB-SCHEMA.md documente | Nom réel en base |
|------------------------|-----------------|
| `accounts_company_name1` | `accounts_company_name_1` |
| `purchase_orders_order_code` | `purchase_orders_code` |
| `purchase_orders_amount_lines_excl_vat` | `purchase_orders_total_amount_excl_vat` |
| `purchase_orders_lines_ordered_quantity` | `purchase_orders_lines_ordered_purchase_quantity` |
| `purchase_orders_lines_received_quantity` | `purchase_orders_lines_received_purchase_quantity` |
| `manufacturing_orders_order_status` | `manufacturing_orders_status` |
| `suppliers_company_name` | `suppliers_name` |
