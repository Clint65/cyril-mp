# Table `id_mapping` — Modèle de données

## Rôle

La table `id_mapping` est le **registre central de versioning** du schéma Databox. Elle sert à deux choses :

1. **Identifier** chaque enregistrement de manière unique dans Databox (`idDatabox`)
2. **Versionner** les entités dans le temps en conservant l'historique de leurs modifications

Toutes les tables métier du schéma (comptes, articles, factures, etc.) n'ont **pas leur propre clé primaire auto-générée** : elles référencent `id_mapping.id_databox` comme clé primaire étrangère.

---

## Définition de la table

```sql
-- Schéma : databox
CREATE TABLE id_mapping (
    id_databox        BIGSERIAL    PRIMARY KEY,
    id_correspondence TEXT         UNIQUE,
    start_date        TIMESTAMP,
    end_date          TIMESTAMP,
    src_system_type   TEXT,
    tgt_system_type   TEXT
);
```

| Colonne            | Type        | Contrainte | Description |
|--------------------|-------------|------------|-------------|
| `id_databox`       | `BIGSERIAL` | PK         | Identifiant interne Databox, auto-incrémenté. Unique par version. |
| `id_correspondence`| `TEXT`      | UNIQUE     | Identifiant dans le système source (ERP, CRM, PLM…). Stable dans le temps pour une même entité. |
| `start_date`       | `TIMESTAMP` |            | Date de début de validité de cette version. |
| `end_date`         | `TIMESTAMP` |            | Date de fin de validité. `NULL` = version courante. |
| `src_system_type`  | `TEXT`      |            | Système d'origine de la donnée (ex : `"ERP"`, `"CRM"`, `"PLM"`). |
| `tgt_system_type`  | `TEXT`      |            | Système cible. |

---

## Principe de versioning (Slowly Changing Dimension de type 2)

Le modèle implémente un **SCD Type 2** : chaque modification d'une entité ne met pas à jour l'enregistrement existant, mais crée une nouvelle ligne avec un nouveau `id_databox`.

### Cycle de vie d'une entité

```
Création :
  id_databox=1001 | id_correspondence="ART-123" | start_date=2024-01-01 | end_date=NULL

Modification :
  id_databox=1001 | id_correspondence="ART-123" | start_date=2024-01-01 | end_date=2024-06-15  ← archivé
  id_databox=1542 | id_correspondence="ART-123" | start_date=2024-06-15 | end_date=NULL        ← courant
```

**Règles clés :**
- `end_date IS NULL` → version courante (la seule visible par défaut dans les requêtes)
- `end_date IS NOT NULL` → version historique (archivée)
- Toutes les versions d'une même entité partagent le même `id_correspondence`
- `id_databox` est différent à chaque nouvelle version

---

## Relations avec les tables métier

La table `id_mapping` est la table **centrale** du schéma. Les 40+ tables métier y sont reliées via une FK `ON DELETE CASCADE`.

### Tables en relation directe

**Tiers**
- `accounts` (comptes clients)
- `accounts_address_lines` (adresses)
- `accounts_contact_lines` (contacts)
- `carriers` (transporteurs)
- `carriers_address_lines`
- `carriers_contact_lines`
- `suppliers` (fournisseurs)
- `suppliers_address_lines`
- `suppliers_contact_lines`

**Articles & Nomenclatures**
- `articles`
- `bom` (nomenclatures)
- `bom_lines`
- `bom_commercial`
- `bom_commercial_lines`
- `bom_subcontracting`
- `bom_subcontracting_material_lines`
- `bom_subcontracting_service_lines`

**Cycle vente**
- `quotes` (devis)
- `quotes_lines`
- `sales_orders` (commandes clients)
- `sales_orders_lines`
- `sales_reps` (représentants commerciaux)
- `deliveries` (livraisons)
- `deliveries_lines`
- `invoices` (factures)
- `invoices_lines`

**Cycle achat**
- `purchase_orders` (commandes fournisseurs)
- `purchase_orders_lines`
- `purchase_orders_invoicing_elements`
- `purchase_invoices` (factures fournisseurs)
- `purchase_invoices_lines`
- `receipts` (réceptions)
- `receipts_lines`

**Production**
- `manufacturing_orders` (ordres de fabrication)
- `manufactured_products_lines`
- `components_lines`
- `operations_lines`
- `means_lines`
- `authorizations_lines`

**Autres**
- `documents`
- `open_items` (encours)
- `opportunities` (opportunités)
- `quality_events` (événements qualité)
- `actions_lines`
- `projects`
- `projects_tasks_lines`
- `projects_tasks_operations_lines`
- `service_contracts` (contrats de service)
- `service_contracts_lines`
- `customer_assets` (équipements client)
- `customer_assets_lines`
- `customer_due_dates`

### Convention de nommage des FK

Chaque table métier possède une colonne `{entity}_id_databox` (ex : `articles_id_databox`, `invoices_id_databox`) qui est à la fois :
- **clé primaire** de la table métier
- **clé étrangère** vers `id_mapping.id_databox`

Pour les tables de lignes (`_lines`), la convention est `{entity}_lines_id` (ex : `invoices_lines_id`).

```sql
-- Exemple : table articles
ALTER TABLE articles
  ADD CONSTRAINT articles_id_databox_fkey
  FOREIGN KEY (articles_id_databox)
  REFERENCES id_mapping(id_databox)
  ON DELETE CASCADE;
```

---

## Schéma de relation simplifié

```
id_mapping
├── id_databox (PK, BIGSERIAL)
├── id_correspondence (UNIQUE TEXT)
├── start_date
├── end_date          ← NULL = version courante
├── src_system_type
└── tgt_system_type
      │
      ├──< articles (articles_id_databox)
      ├──< accounts (accounts_id_databox)
      ├──< invoices (invoices_id_databox)
      │     └──< invoices_lines (invoices_lines_id)
      ├──< sales_orders (sales_orders_id_databox)
      │     └──< sales_orders_lines (sales_orders_lines_id)
      ├──< manufacturing_orders (manufacturing_orders_id_databox)
      │     ├──< manufactured_products_lines
      │     ├──< components_lines
      │     ├──< operations_lines
      │     ├──< means_lines
      │     └──< authorizations_lines
      └──< ... (toutes les autres tables métier)
```

> Note : les tables de lignes (`_lines`) ont leur propre entrée dans `id_mapping`, distincte de l'entrée de leur entité parente. Un ordre de fabrication et chacune de ses lignes ont chacun un `id_databox` indépendant.

---

## Requêtes SQL de référence

### Lire la version courante d'une entité

```sql
SELECT a.*, m.id_correspondence, m.start_date, m.src_system_type
FROM databox.articles a
INNER JOIN databox.id_mapping m ON m.id_databox = a.articles_id_databox
WHERE m.end_date IS NULL;
```

### Lire toutes les versions d'une entité (historique)

```sql
SELECT a.*, m.start_date, m.end_date
FROM databox.articles a
INNER JOIN databox.id_mapping m ON m.id_databox = a.articles_id_databox
WHERE m.id_correspondence = 'ART-00123'
ORDER BY m.start_date ASC;
```

### Supprimer une entité (cascade)

Supprimer une ligne dans `id_mapping` supprime automatiquement la ligne correspondante dans toutes les tables métier (`ON DELETE CASCADE`). À l'inverse, les tables métier ne peuvent pas être supprimées sans passer par `id_mapping`.
