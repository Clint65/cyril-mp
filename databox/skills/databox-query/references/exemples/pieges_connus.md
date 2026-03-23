# Pièges connus du schéma Databox

> Chaque piège est documenté avec une requête fausse, une requête correcte et l'impact mesuré.
> **Toute requête SQL générée par le skill DOIT être vérifiée contre ces 6 pièges.**

## Résumé

| # | Piège | Symptôme | Impact mesuré |
|---|-------|----------|---------------|
| 1 | Oubli de `end_date IS NULL` | Résultats multipliés (versions historiques) | Devis : 659 au lieu de 327 (×2) |
| 2 | Résolution idcor sans id_mapping | Client non résolu, jointure vide | 82% des factures sans client (55 vs 309) |
| 3 | Jointure article sans cast TEXT | Erreur SQL ou jointure vide | Erreur PostgreSQL : `operator does not exist: text = bigint` |
| 4 | Aliases id_mapping manquants | Erreur SQL | `table name "id_mapping" specified more than once` |
| 5 | invoices_sign ignoré | CA gonflé (avoirs comptés positivement) | +3 639 € d'écart (17 730 vs 14 091) |
| 6 | Codes bruts au lieu de libellés | Résultat illisible pour l'utilisateur | '1', '2', '3' au lieu de noms explicites |

---

## Piège 1 : Oubli de `end_date IS NULL`

**Symptôme :** Le nombre d'enregistrements est supérieur à la réalité — chaque version historique est comptée.

**Requête FAUSSE :**
```sql
-- DANGER : sans filtre, on compte toutes les versions (courantes + archivées)
SELECT COUNT(*) AS nb_devis FROM databox.quotes;
-- Résultat : 659 devis
```

**Requête CORRECTE :**
```sql
SELECT COUNT(*) AS nb_devis
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE im.end_date IS NULL;
-- Résultat : 327 devis (versions courantes uniquement)
```

**Impact :** 659 vs 327 — les devis sont comptés **deux fois** sans le filtre. Sur les factures l'écart est moindre (310 vs 308) mais sur les devis c'est un facteur ×2.

**Règle :** TOUJOURS joindre `id_mapping` et filtrer `end_date IS NULL` sur chaque table versionnée.

---

## Piège 2 : Résolution idcor sans passer par id_mapping

**Symptôme :** La jointure vers l'entité liée (client, fournisseur, commercial) est vide ou presque — la plupart des liens ne se résolvent pas.

**Requête FAUSSE :**
```sql
-- DANGER : jointure directe idcor → table cible (bypass id_mapping)
SELECT COUNT(*) AS nb_matches
FROM databox.invoices i
LEFT JOIN databox.accounts a ON i.invoices_idcor_account = a.accounts_id_databox::text
WHERE a.accounts_id_databox IS NOT NULL;
-- Résultat : 55 factures avec client résolu
```

**Requête CORRECTE :**
```sql
-- Les colonnes idcor contiennent un id_correspondence, pas un id_databox
SELECT COUNT(*) AS nb_matches
FROM databox.invoices i
LEFT JOIN databox.id_mapping acc_im
  ON i.invoices_idcor_account = acc_im.id_correspondence
  AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE a.accounts_id_databox IS NOT NULL;
-- Résultat : 309 factures avec client résolu
```

**Impact :** 55 vs 309 — **82% des factures perdent leur client** sans passer par id_mapping. Le `idcor` est un `id_correspondence` (identifiant stable), pas un `id_databox` (identifiant interne par version).

**Règle :** TOUJOURS résoudre un `idcor` via `id_mapping.id_correspondence` → `id_mapping.id_databox` → table cible.

---

## Piège 3 : Jointure article sans cast TEXT

**Symptôme :** Erreur SQL ou jointure silencieusement vide (selon le SGBD).

**Requête FAUSSE :**
```sql
-- DANGER : types incompatibles (text vs bigint)
SELECT COUNT(*) AS nb_lignes_avec_article
FROM databox.sales_orders_lines sol
LEFT JOIN databox.articles art
  ON sol.sales_orders_lines_id_product = art.articles_id_databox
WHERE art.articles_product_code IS NOT NULL;
-- Erreur : operator does not exist: text = bigint
```

**Requête CORRECTE :**
```sql
SELECT COUNT(*) AS nb_lignes_avec_article
FROM databox.sales_orders_lines sol
LEFT JOIN databox.articles art
  ON sol.sales_orders_lines_id_product = art.articles_id_databox::text
WHERE art.articles_product_code IS NOT NULL;
-- Résultat : 3 697 lignes avec article résolu
```

**Impact :** PostgreSQL refuse la requête (erreur de type). Le cast `::text` est **obligatoire** car `*_id_product` est stocké en TEXT alors que `articles_id_databox` est un BIGINT.

**Règle :** TOUJOURS caster avec `::text` quand on joint via `*_id_product`.

---

## Piège 4 : Aliases id_mapping manquants

**Symptôme :** Erreur SQL dès qu'on joint `id_mapping` plus d'une fois dans la même requête.

**Requête FAUSSE :**
```sql
-- DANGER : deux jointures id_mapping sans alias
SELECT i.invoices_invoice_code, a.accounts_company_name_1
FROM databox.invoices i
INNER JOIN databox.id_mapping ON i.invoices_id_databox = id_mapping.id_databox
LEFT JOIN databox.id_mapping ON i.invoices_idcor_account = id_mapping.id_correspondence
LEFT JOIN databox.accounts a ON id_mapping.id_databox = a.accounts_id_databox
WHERE id_mapping.end_date IS NULL;
-- Erreur : table name "id_mapping" specified more than once
```

**Requête CORRECTE :**
```sql
SELECT i.invoices_invoice_code, a.accounts_company_name_1
FROM databox.invoices i
INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping acc_im
  ON i.invoices_idcor_account = acc_im.id_correspondence
  AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a ON acc_im.id_databox = a.accounts_id_databox
WHERE doc_im.end_date IS NULL;
```

**Impact :** Erreur bloquante. Chaque jointure `id_mapping` doit avoir un alias unique.

**Convention d'alias :**
- `doc_im` ou `{entity}_im` : version courante du document principal
- `acc_im` : résolution du compte client
- `rep_im` : résolution du commercial
- `sup_im` : résolution du fournisseur
- `art_im` : résolution de l'article

---

## Piège 5 : invoices_sign ignoré dans les agrégations

**Symptôme :** Le CA est gonflé — les avoirs (invoices_sign = -1) sont additionnés au lieu d'être soustraits.

**Requête FAUSSE :**
```sql
-- DANGER : les avoirs sont comptés positivement
SELECT SUM(i.invoices_amount_lines_excl_vat) AS ca_ht
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL
  AND i.invoices_invoice_date >= '2024-01-01'
  AND i.invoices_invoice_date < '2025-01-01';
-- Résultat : 17 730,35 € (FAUX — avoirs inclus positivement)
```

**Requête CORRECTE :**
```sql
SELECT SUM(i.invoices_amount_lines_excl_vat * i.invoices_sign) AS ca_net_ht
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL
  AND i.invoices_invoice_date >= '2024-01-01'
  AND i.invoices_invoice_date < '2025-01-01';
-- Résultat : 14 091,41 € (CORRECT — avoirs déduits)
```

**Impact :** +3 639 € d'écart (25% de surévaluation du CA). Le montant des avoirs est compté deux fois : une fois positivement, une fois non déduit.

**Règle :** TOUJOURS multiplier par `invoices_sign` dans les `SUM` sur la table `invoices`. Même logique pour `open_items_sign` sur `open_items`.

---

## Piège 6 : Codes bruts au lieu de libellés décodés

**Symptôme :** Les résultats affichent des codes numériques ('1', '2', '3') au lieu de libellés compréhensibles.

**Requête FAUSSE :**
```sql
-- Résultat illisible pour l'utilisateur métier
SELECT quotes_quote_status, COUNT(*) AS nb
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
WHERE im.end_date IS NULL
GROUP BY quotes_quote_status;
-- Résultat : '1' → 245, '2' → 18, '3' → 64
```

**Requête CORRECTE :**
```sql
SELECT q.quotes_quote_status AS code, vll.label AS statut, COUNT(*) AS nb
FROM databox.quotes q
INNER JOIN databox.id_mapping im ON q.quotes_id_databox = im.id_databox
LEFT JOIN databox.value_list_column_config vlcc
  ON vlcc.column_name = 'quotes_quote_status'
LEFT JOIN databox.value_list_entry vle
  ON vle.value_list_id = vlcc.value_list_id
  AND vle.neutral_code = q.quotes_quote_status
LEFT JOIN databox.value_list_label vll
  ON vll.entry_id = vle.id AND vll.locale = 'fr'
WHERE im.end_date IS NULL
GROUP BY q.quotes_quote_status, vll.label;
-- Résultat : '1' → "Non commandé" (245), '2' → "Partiellement commandé" (18), '3' → "Totalement commandé" (64)
```

**Impact :** Résultat illisible vs lisible. Pour les requêtes analytiques (agrégations sur codes), le décodage est optionnel. Il devient obligatoire quand la réponse doit afficher des libellés à l'utilisateur.

**Note importante :** Les libellés value_list pour les statuts devis sont :
- `'1'` = "Non commandé" (ouvert)
- `'2'` = "Partiellement commandé"
- `'3'` = "Totalement commandé"

Ceci **diffère** de DB-SCHEMA.md qui documente `'2'` = commandé et `'3'` = partiellement commandé. **Les libellés value_list sont la source de vérité.**

**Pattern de décodage générique :**
```sql
LEFT JOIN databox.value_list_column_config vlcc ON vlcc.column_name = '{snake_case_column_name}'
LEFT JOIN databox.value_list_entry vle ON vle.value_list_id = vlcc.value_list_id AND vle.neutral_code = {table}.{column}
LEFT JOIN databox.value_list_label vll ON vll.entry_id = vle.id AND vll.locale = 'fr'
```

Le `column_name` dans `value_list_column_config` est en **snake_case** (nom BDD, ex : `quotes_quote_status`).
