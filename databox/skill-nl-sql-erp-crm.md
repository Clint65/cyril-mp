# Skill NL→SQL : Interrogation Databox ERP/CRM en langage naturel

## Objectif

Créer un skill Claude Code permettant d'interroger la base PostgreSQL **Databox** en langage naturel, sans écrire de SQL.

**Exemples de questions cibles :**
- "Quelle est l'évolution de mon chiffre d'affaires sur les 12 derniers mois ?"
- "Quel est mon taux de conversion devis/commandes par commercial ?"
- "Quels clients n'ont pas commandé depuis 3 mois alors qu'ils commandaient régulièrement ?"
- "Quels sont mes 10 meilleurs clients par CA cette année ?"

---

## Contexte technique

- **SGBD :** PostgreSQL, schéma `databox`
- **Système :** API NestJS Databox — agrège les données ERP, CRM et PLM
- **Tables :** ~50 tables préfixées par domaine (`invoices_*`, `sales_orders_*`, `quotes_*`…)
- **Dossier de travail :** `/Users/cyril/Documents/01_Sources/02_IA/databox-skill/`
- **Accès Claude :** Serveur MCP PostgreSQL configuré via `.mcp.json` local (voir `setup-mcp-postgres-local.md`)
- **Accès BDD :** Utilisateur lecture seule dédié (`claude_readonly`)

### Documentation existante du schéma (dans le dossier de travail)

Trois documents présents dans le dossier de travail constituent la source de vérité du modèle :

- **`DB-SCHEMA.md`**
  Référence complète : structure des tables par domaine, conventions de nommage, relations entre entités, patterns SQL éprouvés (12 patterns documentés)

- **`ID-MAPPING.md`**
  Documentation de la table centrale de versioning : principe SCD Type 2, cycle de vie des entités, conventions de nommage des FK

- **`VALUE-LISTS-ARCHITECTURE.md`**
  Système de décodage des colonnes codées : tables `value_list`, `value_list_entry`, `value_list_label`, `value_list_system_mapping`, `value_list_column_config`

**Ces trois documents sont dans le même dossier que le skill — Claude y a accès directement.**

---

## Particularités critiques du schéma Databox

Ces points sont non-négociables — toute requête SQL qui les ignore produit des résultats faux.

### 1. La règle d'or : toujours filtrer sur la version courante

Chaque entité est versionnée via `id_mapping`. Sans le filtre `end_date IS NULL`, on obtient toutes les versions historiques.

```sql
-- TOUJOURS joindre id_mapping et filtrer end_date IS NULL
SELECT *
FROM databox.invoices i
INNER JOIN databox.id_mapping im ON i.invoices_id_databox = im.id_databox
WHERE im.end_date IS NULL;
```

### 2. Les relations via `idcor` (pas de FK directes entre entités)

Les colonnes `*_idcor_*` contiennent un `id_correspondence` (identifiant stable). Pour résoudre un `idcor` en ligne concrète, il faut passer par `id_mapping` :

```sql
-- Résoudre invoices_idcor_account → ligne accounts
LEFT JOIN databox.id_mapping acc_im
    ON i.invoices_idcor_account = acc_im.id_correspondence
    AND acc_im.end_date IS NULL
LEFT JOIN databox.accounts a
    ON acc_im.id_databox = a.accounts_id_databox
```

### 3. Jointure article : cast TEXT obligatoire

Les colonnes `*_id_product` dans les tables de lignes contiennent l'`id_databox` de l'article **en texte**. Le cast est obligatoire :

```sql
LEFT JOIN databox.articles art
    ON sol.sales_orders_lines_id_product = art.articles_id_databox::text
```

### 4. Alias obligatoires sur `id_mapping`

Quand on joint `id_mapping` plusieurs fois dans la même requête (ex : résoudre l'entité principale + un compte client), chaque jointure doit avoir un alias distinct :

```sql
INNER JOIN databox.id_mapping doc_im ON i.invoices_id_databox = doc_im.id_databox
LEFT JOIN databox.id_mapping acc_im ON i.invoices_idcor_account = acc_im.id_correspondence
```

### 5. `invoices_sign` pour les agrégations comptables

Les avoirs ont `invoices_sign = -1`, les factures normales `+1`. Toujours multiplier dans les SUM pour éviter de comptabiliser les avoirs positivement.

### 6. Décodage des colonnes codées via le système de listes de valeurs

Les colonnes contenant des codes (statuts, types, catégories…) se décodent via 4 tables dédiées, **pas** via une table de paramétrage générique.

**Tables impliquées :**
- `value_list` — catalogue des listes (ex : `account_type`, `quotes_status`)
- `value_list_entry` — entrées avec un `neutral_code` agnostique du système source
- `value_list_label` — libellés multilingues par entrée (`locale` = `fr`, `en`…)
- `value_list_system_mapping` — correspondance entre `neutral_code` et code système source (x3, salesforce…)
- `value_list_column_config` — table pivot : quelle colonne BDD utilise quelle liste

**Pattern de jointure pour obtenir le libellé d'une colonne codée :**

```sql
-- Exemple : décoder accounts_account_type en français
LEFT JOIN databox.value_list_column_config vlcc
    ON vlcc.column_name = 'accountsAccountType'
LEFT JOIN databox.value_list_entry vle
    ON vle.value_list_id = vlcc.value_list_id
    AND vle.neutral_code = a.accounts_account_type
LEFT JOIN databox.value_list_label vll
    ON vll.entry_id = vle.id AND vll.locale = 'fr'
```

**Découvrir quelles colonnes ont un lookup :**
```sql
SELECT column_name, value_list_id, is_array
FROM databox.value_list_column_config
ORDER BY column_name;
```

**Statuts devis (cas fréquent, pas de jointure nécessaire) :**
- `quotes_quote_status` : `'1'`=ouvert, `'2'`=commandé, `'3'`=partiellement commandé
- Devis converti = statut `'2'` ou `'3'`

> Pour les requêtes analytiques (CA, taux de conversion…), les jointures de décodage sont optionnelles — on travaille sur les codes. Elles deviennent nécessaires quand la réponse doit afficher des libellés lisibles.

---

## Architecture du skill

### Exemple de structure des fichiers

```
skill-nl-sql/
├── SKILL.md                    # Définition du skill (system prompt + instructions)
└── references/
    ├── schema/
    │   ├── db-schema.md            # Copie/lien vers DB-SCHEMA.md
    │   ├── id-mapping.md           # Copie/lien vers ID-MAPPING.md
    │   └── relations_cles.md       # Résumé des relations inter-domaines
    ├── metier/
    │   ├── kpis.md                 # Définition précise de chaque KPI
    │   └── glossaire.md            # Termes métier → colonnes SQL + patterns
    └── exemples/
        ├── requetes_ventes.md      # Few-shot NL→SQL sur le cycle vente
        ├── requetes_achats.md      # Few-shot NL→SQL sur le cycle achat
        └── pieges_connus.md        # Pièges spécifiques Databox
```

> Note : `references/schema/` peut directement pointer vers les docs existantes plutôt que de les dupliquer.

### Logique d'exécution en 5 étapes

1. **COMPRENDRE** — Classifier la demande (quel domaine ? quelle période ? quel filtre ?)
2. **IDENTIFIER** — Quelles tables et jointures sont nécessaires ?
3. **GÉNÉRER** — Produire le SQL en appliquant systématiquement les règles Databox
4. **VALIDER** — Vérifier : `end_date IS NULL` présent ? aliases corrects ? `invoices_sign` appliqué ?
5. **EXÉCUTER** — Lancer la requête et formater le résultat en langage naturel

---

## Mode de construction : dialogue itératif

Les fichiers `references/metier/` ne se remplissent pas en une seule passe. Le schéma technique est déjà documenté — ce qui reste à construire par dialogue, c'est le sens **métier** :

1. Claude lit les deux docs de référence
2. Claude formule des hypothèses sur les KPIs et définitions métier
3. Claude pose des questions ciblées UNE PAR UNE pour valider
4. L'utilisateur confirme ou précise
5. Claude met à jour `glossaire.md` et `kpis.md` au fil du dialogue

### Exemples de questions que Claude doit poser

- "Le CA que vous mesurez est basé sur les factures (`invoices`) ou les commandes (`sales_orders`) ?"
- "Pour le CA, vous tenez compte des avoirs (signe -1) en déduction ?"
- "Un devis 'partiellement commandé' (statut 3) est-il considéré comme converti dans votre taux de conversion ?"
- "Les commandes directes sans devis (sans `sales_orders_idcor_quote`) sont-elles incluses dans le taux de conversion ?"
- "Quand vous dites 'client à surveiller', quel est le critère principal : retard de paiement, baisse de CA, absence de commande ?"

### Règle d'or
Claude ne suppose jamais une définition métier sans confirmation. Mieux vaut poser une question de trop que générer un SQL silencieusement faux.

---

## Pièces maîtresses à construire (via le dialogue)

### 1. Le glossaire métier (`glossaire.md`)
Chaque terme validé par l'utilisateur, traduit en SQL Databox :
- "Chiffre d'affaires", "devis converti", "client actif", "taux de conversion"...
- Avec les filtres exacts (`invoices_sign`, statuts, périodes)

### 2. Les exemples few-shot (15-20 paires NL→SQL)
Co-construits et validés, incluant les patterns récurrents :
- Agrégation par période avec `generate_series`
- Résolution `idcor` → entité
- Multi-devises avec `currency_rate`

---

## Risques et points d'attention

1. **Oubli du filtre `end_date IS NULL`** — résultats multipliés par le nombre de versions historiques
2. **Alias manquants sur `id_mapping`** — erreur SQL si jointures multiples sans alias
3. **Cast TEXT manquant** sur `*_id_product` — jointure article silencieusement vide
4. **`invoices_sign` ignoré** — les avoirs comptent positivement dans le CA
5. **Définitions métier ambiguës** — CA facturé vs commandé, devis partiellement converti
6. **Multi-devises** — montants non comparables sans appliquer `currency_rate`

---

## Livrable attendu

Un skill Claude Code invocable qui :
- Comprend une question en français
- Génère et exécute le SQL Databox correct (avec `id_mapping`, `idcor`, aliases)
- Retourne une réponse en langage naturel avec les données
- Sert de référence validée pour le port vers Flowise (voir `flowise-nl-sql-erp-crm.md`)
