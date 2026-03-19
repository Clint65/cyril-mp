# Architecture : Systeme de listes de valeurs

## Vue d'ensemble

Le systeme de listes de valeurs remplace l'ancien `CSVListService` (fichiers CSV charges en memoire) par une structure en base de donnees multi-systemes avec un framework de loaders extensible.

```
                    +-----------------------+
                    |  Endpoint /columns    |
                    |  (AbstractService)    |
                    +-----------+-----------+
                                |
                                | getFormatterParams(col, locale)
                                v
                    +-----------+-----------+
                    |  ValueListService     |
                    |  (cache config +      |
                    |   queries DB)         |
                    +-----------+-----------+
                                |
                                v
              +-----------------+------------------+
              |            Base de donnees          |
              |  value_list -> value_list_entry     |
              |  -> value_list_label               |
              |  -> value_list_system_mapping       |
              |  -> value_list_column_config        |
              +------------------------------------+
                                ^
                                | upsert
                    +-----------+-----------+
                    |  ValueListLoadService |
                    |  (orchestration)      |
                    +-----------+-----------+
                                ^
                                | parse()
                    +-----------+-----------+
                    |  ValueListParser      |
                    |  (X3, Salesforce...)   |
                    +-----------------------+
```

## Schema de la base (schema `databox`)

### value_list
Liste de valeurs (ex: "account_type", "local_menus_419").

| Colonne | Type | Description |
|---------|------|-------------|
| id | serial PK | Identifiant |
| code | text UNIQUE | Code unique (snake_case) |
| name | text | Nom lisible |
| description | text? | Description optionnelle |

### value_list_entry
Entrees individuelles avec un code neutre (independant du systeme source).

| Colonne | Type | Description |
|---------|------|-------------|
| id | serial PK | Identifiant |
| value_list_id | integer FK | Reference vers value_list |
| neutral_code | text | Code agnostique systeme |
| sort_order | integer? | Ordre de tri |

Contrainte : `UNIQUE(value_list_id, neutral_code)`

### value_list_label
Labels multilingues par entree. Table separee pour permettre l'ajout de nouvelles langues.

| Colonne | Type | Description |
|---------|------|-------------|
| id | serial PK | Identifiant |
| entry_id | integer FK | Reference vers value_list_entry |
| locale | text | Code langue (en, fr, fr-CA) |
| label | text | Libelle dans cette langue |

Contrainte : `UNIQUE(entry_id, locale)`

### value_list_system_mapping
Correspondance entre code neutre et code systeme source.

| Colonne | Type | Description |
|---------|------|-------------|
| id | serial PK | Identifiant |
| entry_id | integer FK | Reference vers value_list_entry |
| system_type | text | Systeme source (x3, salesforce, plm) |
| system_code | text | Code dans ce systeme |

Contrainte : `UNIQUE(entry_id, system_type)`

### value_list_column_config
Configuration : quelle colonne de la base utilise quelle liste.

| Colonne | Type | Description |
|---------|------|-------------|
| id | serial PK | Identifiant |
| column_name | text UNIQUE | Nom de la colonne (ex: accountsAccountType) |
| value_list_id | integer FK | Reference vers value_list |
| is_array | boolean | true si la colonne contient un tableau de codes |

## Composants NestJS

### ValueListService (`src/value-list/value-list.service.ts`)
Service @Global() injectable. Charge la configuration des colonnes en cache au demarrage (`OnModuleInit`).

**Methodes :**
- `hasConfig(columnName)` : boolean (synchrone, utilise le cache)
- `getFormatterParams(columnName, locale)` : retourne `Record<string, string>` ou `Record<string, string>[]`

**Format de sortie :** `{ "code": "label (code)" }` — compatible avec le format attendu par le frontend.

**Gestion des lookupArray :** pour les colonnes `is_array=true`, le service cherche toutes les `value_list` dont le code partage le meme prefixe (ex: `product_statistic_family_0`, `_1`, `_2`...), les trie par index et retourne un tableau.

### ValueListLoadService (`src/value-list/value-list-load.service.ts`)
Service d'orchestration pour le chargement. Recoit les parsers via injection (`VALUE_LIST_PARSERS` token).

**Methodes :**
- `loadFromFile(systemType, content, fileName)` : parse + persiste
- `persistParsedData(parsedLists, systemType)` : persiste directement (utilise par le script seed)
- `getRegisteredParsers()` : liste des parsers disponibles

**Upsert :** chaque operation est idempotente — les entrees existantes sont mises a jour, les nouvelles creees.

### ValueListController (`src/value-list/value-list.controller.ts`)
Endpoint REST sous `/databox/value-lists/`.

- `POST /load` : multipart avec champs `systemType` (body) et `file` (fichier CSV)

### ValueListModule (`src/value-list/value-list.module.ts`)
Module `@Global()` importe dans `AppModule`. Enregistre les parsers via `useFactory`.

## Integration avec AbstractService

`AbstractService.getColumnConfig()` utilise `ValueListService` :

```typescript
// Verifie si la colonne a un lookup (synchrone, cache)
if (!this.valueListService.hasConfig(col)) {
    return baseConfig;
}
// Charge les formatterParams depuis la base (async)
const formatterParams = await this.valueListService.getFormatterParams(col, currentLang);
```

Les methodes `getColumnConfig()` et `getColumns()` sont `async`.

## Fichiers de configuration

### link.config.json

Conserve pour le parser X3 et le script seed. Deux sections :

- **fileKey** : pour chaque type de fichier CSV, quelles colonnes forment la cle
- **columns** : pour chaque colonne de la base, quel fichier CSV et quelle colonne contient le label

Seuls les fichiers references dans `columns` sont charges par le parser. Les fichiers presents dans `fileKey` mais pas dans `columns` sont ignores (pas encore utilises).

## Formats CSV X3

Le parser X3 (`x3-csv-parser.ts`) gere 3 formats :

1. **Simple** (1 cle) : `FCADLink_AccountType_ENG_*.csv` — une liste par fichier
2. **Multi-liste** (2 cles) : `FCADLink_LocalMenus_FRA_*.csv` — plusieurs listes dans un fichier (colonne 0 = id liste)
3. **Multi-fichier array** : `FCADLink_ProductStatisticFamily_0_ENG_*.csv` — un fichier par index de tableau
