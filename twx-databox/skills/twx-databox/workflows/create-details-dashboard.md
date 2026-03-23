# Workflow : Créer un Dashboard Details

## Objectif
Créer un dashboard de détails pour une entité. Le pattern Dashboard + DashboardDetails est omniprésent : le Dashboard liste les entités, et quand on clique sur une ligne, on navigue vers le DashboardDetails qui affiche les détails de l'entité sélectionnée.

## Pattern de navigation

```
DataBox_Dashboard_[Entity]           →  Liste (Grid) des entités
    ↓ clic sur une ligne (événement Details du FCADDataGrid)
DataBox_Dashboard_[Entity]Details    →  Détails de l'entité sélectionnée
    contient : Header + Tiles de sous-entités (lignes, relations)
```

## Exemples existants

| Dashboard | Details | Contenu Details |
|-----------|---------|-----------------|
| DataBox_Dashboard_Accounts | DataBox_Dashboard_AccountsDetails | Adresses, Contacts, Devis, Commandes, Factures, Livraisons |
| DataBox_Dashboard_Quotes | DataBox_Dashboard_QuotesDetails | Lignes de devis, Commandes associées |
| DataBox_Dashboard_Invoices | DataBox_Dashboard_InvoicesDetails | Lignes de facture |
| DataBox_Dashboard_SalesOrders | DataBox_Dashboard_SalesOrdersDetails | Lignes de commande |

## Processus

### Étape 1 : Identifier les sous-entités

Scanner le controller NestJS pour les endpoints de sous-ressources :
```bash
grep -n "@Get\|@Post" /Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/src/data-databox/[module]/*.controller.ts
```

Chaque sous-ressource correspond à une tile dans le DashboardDetails :
- `GET /entity/entityLines/:id` → Tile Grid des lignes
- `GET /entity/quotes/:id` → Tile Grid des devis associés
- `GET /entity/invoices/:id` → Tile Grid des factures associées

### Étape 2 : Lire un DashboardDetails existant comme référence

```bash
cat /Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/08-SourceControl/DataBox/Mashups/DataBox_Dashboard_AccountsDetails.xml
```

Analyser la structure du JSON mashupContent :

**Section Data** — Les services appelés :
```json
"Data": {
  "Things_DataBox_Helper_Platform": {
    "EntityName": "DataBox_Helper_Platform",
    "Services": [
      {"Name": "getAccount", "Parameters": {}, "Target": "getAccount"},
      {"Name": "getAccountQuotes", "Parameters": {}, "Target": "getAccountQuotes"}
    ]
  }
}
```

**Section Events** — Chargement au Loaded + navigation :
```json
"Events": [
  {
    "EventTriggerEvent": "Loaded",
    "EventTriggerId": "mashup-root",
    "EventHandlerService": "getAccount"
  }
]
```

**Section UI** — Structure typique d'un Details :
```
flexcontainer (column)
├── Header (flexcontainer row) : infos principales de l'entité
│   ├── Label "Code" + ValueDisplay
│   ├── Label "Nom" + ValueDisplay
│   └── Label "Date" + ValueDisplay
├── Tabs ou GridStack : sous-entités
│   ├── Tile Grid : Lignes de l'entité
│   ├── Tile Grid : Documents associés (devis, commandes...)
│   └── Tile Grid : Autres relations
└── Footer : bouton retour
```

### Étape 3 : Configurer la navigation depuis le Dashboard parent

Dans le Dashboard parent (DataBox_Dashboard_[Entity].xml), le clic sur une ligne déclenche la navigation vers le Details via 3 widgets :

1. **FCADDataGrid** émet l'événement `Details` avec `ActionId` = id de la ligne
2. **expression** widget construit l'URL :
   ```
   Output = "/Thingworx/Runtime/index.html#mashup=DataBox_Dashboard_[Entity]Details&id=" + ActionId
   ```
3. **navigationfunction** widget reçoit l'URL et navigue

Le binding dans le JSON :
```json
{"SourceId": "FCADDataGrid-N", "SourceProperty": "ActionId", "TargetId": "expression-N", "TargetProperty": "param1"},
{"SourceId": "expression-N", "SourceProperty": "Output", "TargetId": "navigationfunction-N", "TargetProperty": "URL"}
```

### Étape 4 : Générer le DashboardDetails

Créer `08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[Entity]Details.xml` en suivant le mashup de référence.

Points clés :
- Déclarer les **ParameterDefinitions** pour recevoir l'id depuis la navigation
- Dans la section Data, appeler `get[Entity](id)` pour charger l'entité
- Pour chaque sous-entité, appeler `get[Entity][SubEntity](id)` et `get[Entity][SubEntity]GridConfig()`
- Configurer les DataBindings entre services et widgets
- Générer des **UUIDs uniques** pour tous les IDs (ne pas copier ceux d'un autre mashup)

### Étape 5 : Configurer la navigation retour

Ajouter un bouton ou breadcrumb qui navigue vers le Dashboard parent. Le pattern utilise un `navigationfunction` avec l'URL du Dashboard parent.

### Étape 6 : Checklist de validation

- [ ] Le nom suit la convention `DataBox_Dashboard_[Entity]Details`
- [ ] `projectName="DataBox"` est présent
- [ ] Le mashup déclare un ParameterDefinition pour l'id entrant
- [ ] L'événement Loaded déclenche le chargement de l'entité via `get[Entity](id)`
- [ ] Les services de sous-entités sont déclarés dans la section Data
- [ ] Chaque sous-entité a un DataBinding vers un widget FCADDataGrid
- [ ] Les UUIDs dans le JSON mashup sont tous uniques
- [ ] La navigation retour vers le Dashboard parent est configurée
- [ ] Le Dashboard parent a le binding FCADDataGrid.Details → expression → navigationfunction

### Étape 7 : Créer les dashboards de lignes (optionnel)

Pour les sous-entités qui ont elles-mêmes des détails, créer aussi :
- `DataBox_Dashboard_[Entity]LinesDetails.xml`

Exemple : `DataBox_Dashboard_QuotesLinesDetails.xml` affiche les détails d'une ligne de devis.
