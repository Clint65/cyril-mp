# Checklist de validation SQL Databox

> Vérifier chaque requête générée contre ces 6 règles AVANT exécution.
> Toute requête qui ne respecte pas ces règles produit des résultats faux.

## Checklist

- [ ] **1. end_date IS NULL** — Chaque table versionnée est jointe à `id_mapping` avec `WHERE im.end_date IS NULL`
- [ ] **2. Résolution idcor** — Les colonnes `*_idcor_*` sont résolues via `id_mapping.id_correspondence` → `id_mapping.id_databox` → table cible (jamais en jointure directe)
- [ ] **3. Aliases id_mapping** — Chaque jointure `id_mapping` a un alias unique (`doc_im`, `acc_im`, `rep_im`, `sup_im`, `art_im`)
- [ ] **4. Cast TEXT** — Les jointures article via `*_id_product` utilisent `articles_id_databox::text`
- [ ] **5. invoices_sign** — Les agrégations sur `invoices` multiplient par `invoices_sign` dans les `SUM` (idem `open_items_sign` pour `open_items`)
- [ ] **6. Value lists** — Les colonnes codées affichées à l'utilisateur sont décodées via `value_list_column_config` → `value_list_entry` → `value_list_label` (locale = 'fr'). Le `column_name` est en snake_case (nom BDD).

## Règles complémentaires

- [ ] **Multi-devises** — Pour les agrégations cross-devises, multiplier par `COALESCE(currency_rate, 1)`
- [ ] **Statut commandes** — Filtrer `sales_orders_order_status IN (1, 2)` pour exclure les annulées (statut 0)
- [ ] **generate_series** — Utiliser pour les agrégations temporelles, avec `LEFT JOIN` pour inclure les périodes sans données
- [ ] **COALESCE** — Encadrer les agrégations avec `COALESCE(..., 0)` pour les périodes/entités sans données
