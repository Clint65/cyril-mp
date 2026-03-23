---
name: twx-databox
description: >
  Développement ThingWorx assisté pour le projet DataBox. Génère des services ThingWorx (ThingShapes XML),
  des mashups Dashboard (GridStack responsive) et des tiles (Grid, Synth, Metric) en s'appuyant sur les
  patterns du projet existant. Découvre les endpoints API REST NestJS et crée les entités ThingWorx correspondantes.
  TOUJOURS utiliser cette skill dès que le travail concerne le projet databox-twx-dashboard ou ThingWorx DataBox,
  même si l'utilisateur ne mentionne pas explicitement la skill. Cela inclut : ajout d'entité, nouveau dashboard,
  nouvelle tile, nouveau service ThingWorx, modification d'un service existant, exploration des endpoints API,
  génération ou modification de XML ThingWorx, IIOT, mashup, debug XML ThingWorx, vérification de couverture API,
  création de graphiques ECharts pour dashboards, services JavaScript ThingWorx, requêtes PostgreSQL/InfluxDB pour IIOT,
  calcul TRS/OEE, indicateurs KPI. Déclencher aussi quand l'utilisateur mentionne "thingworx", "twx", "databox dashboard",
  "tile", "thingshape", "endpoint API databox", "service JavaScript twx", "mashup XML", "GridStack", "FCADDataGrid",
  "FCADECharts", "arrêts production", "indicateur device", "TRS", "pareto arrêts".
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Agent
  - AskUserQuestion
  - mcp__plugin_context7_context7__resolve-library-id
  - mcp__plugin_context7_context7__query-docs
---

# TWX-DataBox : Assistant de développement ThingWorx

Cette skill assiste le développement de nouvelles fonctionnalités dans ThingWorx pour le projet DataBox. Elle couvre deux domaines : **DataOps** (ERP/CRM/PLM via API REST) et **IIOT** (Site/Line/Workstation/Devices).

## Chemins du projet

- **Projet ThingWorx** : `/Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/`
- **Fichiers XML individuels** : `08-SourceControl/DataBox/` (Things, ThingShapes, Mashups, DataShapes)
- **API REST NestJS** : `/Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/`
- **Controllers API** : `databox-data-access/src/data-databox/*/`
- **Sources widgets GitLab** : `https://gitlab.com/4cad-group/rd-4cad/data-platform/dashboarding/extensions/`

## Processus principal

<routing>

Quand l'utilisateur demande quelque chose, identifier le cas d'usage :

| Demande | Action |
|---------|--------|
| "Quels endpoints API sont disponibles ?" | → Workflow `discover-api-endpoints` |
| "Ajouter une entité DataOps" / "Nouveau dashboard pour X" | → Workflow `add-dataops-entity` |
| "Ajouter des services à un ThingShape existant" | → Workflow `add-services-to-existing` |
| "Créer un dashboard details pour X" | → Workflow `create-details-dashboard` |
| "Ajouter une tile IIOT" / "Nouvel indicateur device" | → Workflow `add-iiot-tile` |
| "Créer un service IIOT" / "Requête InfluxDB/PostgreSQL" | → Workflow `create-iiot-services` |
| "Créer un KPI / indicateur DataOps" / "Graphique stats" | → Workflow `create-kpi-services` |
| "Enregistrer une tile dans le catalogue" | → Workflow `create-tile-catalog-entry` |
| "Créer un ThingShape pour X" | → Génération ThingShape seul |
| "Créer un dashboard pour X" | → Génération Dashboard seul |
| "Créer une tile Grid/Synth/Metric" | → Génération Tile seule |
| "Quels ThingShapes existent ?" | → Lister `08-SourceControl/DataBox/ThingShapes/` |
| "Quels dashboards existent ?" | → Lister `08-SourceControl/DataBox/Mashups/` |
| "Quelle est la couverture API ?" | → Consulter `references/api-twx-mapping.md` |

</routing>

## Avant de générer : que lire ?

Avant toute génération, lire le fichier de référence ET un exemple existant :

| Ce que tu génères | Référence à lire | Exemple existant à lire |
|-------------------|-------------------|------------------------|
| ThingShape | `references/service-patterns.md` | `ThingShapes/DataBox_TS_Accounts.xml` |
| Dashboard | `references/mashup-patterns.md` | `Mashups/DataBox_Dashboard_Accounts.xml` |
| Dashboard Details | `references/mashup-patterns.md` | `Mashups/DataBox_Dashboard_AccountsDetails.xml` |
| Tile Grid | `references/tile-patterns.md` | `Mashups/DataBox_Tile_Accounts_Grid.xml` |
| Tile Synth | `references/tile-patterns.md` | `Mashups/DataBox_Tile_AccountsDocuments_Synth.xml` |
| Tile Metric/KPI | `references/tile-patterns.md` | `Mashups/DataBox_Tile_AccountsQuotesConversionRate.xml` |
| Tile Device IIOT | `references/tile-patterns.md` | `Mashups/DataBox_Tile_DevicePropertyGraph.xml` |
| Service IIOT (SQL/Influx) | `references/service-patterns-iiot.md` | `ThingShapes/DataBox_TS_Devices.xml` ou `DataBox_TS_TRS.xml` |
| Service KPI/Graph ECharts | `references/service-patterns-iiot.md` (gauge) ou `references/service-patterns.md` | `ThingShapes/DataBox_TS_Dash_Comptes.xml` |
| Services à ajouter | `references/service-patterns.md` | Le ThingShape cible (lire en entier) |

Tous les fichiers exemples sont dans : `08-SourceControl/DataBox/`

## Capacités

### DataOps (ERP/CRM/PLM)

1. **Découverte des endpoints API** : Scanne les controllers NestJS pour lister tous les endpoints disponibles. Identifie les endpoints standards (hérités d'AbstractController) et custom.

2. **Génération de ThingShapes** : Crée des fichiers XML ThingShape avec les services correspondant aux endpoints API.

3. **Génération de Dashboards** : Crée des mashups Dashboard (liste) et DashboardDetails (détails d'une entité).

4. **Génération de Tiles** : Crée des tiles Grid (FCADDataGrid), Synth (synthèse), Metric (FCADECharts).

### IIOT (Site/Line/Workstation/Devices)

1. **Services PostgreSQL** : Services utilisant `Things["DataBox_DB"].runQuery()` pour DATABOX.ACTIVITY, DATABOX.CAUSE.

2. **Services InfluxDB** : Services utilisant `Things["DataBox_DB_Influx"]` pour séries temporelles.

3. **Tiles IIOT** : Tiles Alerts, PropertyGraph, StopsTimeline, TRS/OEE, MTBF, Pareto.

## Couverture actuelle

Entités API **sans ThingShape** (candidates à la génération) :
- `purchaseOrders` - Commandes d'achat
- `purchaseInvoices` - Factures d'achat
- `receipts` - Réceptions
- `openItems` - Articles en attente
- `suppliers` - Fournisseurs
- `carriers` - Transporteurs

Voir `references/api-twx-mapping.md` pour le tableau complet.

## Exemple d'utilisation

```
Utilisateur : "Crée un ThingShape pour PurchaseOrders"

Skill :
1. Lit references/service-patterns.md pour les patterns
2. Lit le controller NestJS purchaseOrders pour les endpoints disponibles
3. Lit DataBox_TS_Accounts.xml comme modèle
4. Génère DataBox_TS_PurchaseOrders.xml avec :
   - getPurchaseOrder(id, idcor)
   - getPurchaseOrderLines(idPo)
   - getPurchaseOrdersGridConfig(small, pagination)
   - getPurchaseOrderLinesGridConfig(pagination)
   - getPurchaseOrderInvoicingElements(idPo)
5. Écrit dans 08-SourceControl/DataBox/ThingShapes/
```

## Règles de génération XML

1. **Toujours lire un exemple existant avant de générer** : Ne jamais inventer de structure XML.

2. **Écrire dans les fichiers individuels** : Créer/modifier uniquement dans `08-SourceControl/DataBox/`, jamais le XML consolidé `06-Entities/DataBox.xml`.

3. **Respecter les conventions de nommage** : Voir `references/xml-conventions.md`.

4. **Structure XML ThingWorx 9.7** :
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Entities majorVersion="9" minorVersion="7" universal="password">
       <!-- Entité ici -->
   </Entities>
   ```

5. **Services JavaScript** : Code dans `<![CDATA[ ... ]]>`. Utiliser `Things["DataBox_Helper_Platform"].GetInfotable({api: "..."})` pour les appels API REST.

6. **Noms de routes API : TOUJOURS lire le controller NestJS** : Les noms de sous-routes (lignes, relations) ne suivent pas de pattern régulier. Par exemple `purchaseOrdersLines` (avec s), `salesOrderLines` (sans s), `invoiceLines` (singulier), `quoteLines` (singulier). Ne jamais deviner le nom — toujours le copier depuis le `@Get('...')` du controller.

7. **Permissions** : ServiceInvoke pour Users, Visibility pour Everyone.

8. **UUIDs dans les mashups** : Les IDs dans le JSON des mashups doivent être des UUIDs uniques. Générer de nouveaux UUIDs pour chaque nouveau mashup.

## Documentation ECharts via Context7

Pour générer des graphiques ECharts, utiliser Context7 pour consulter la documentation officielle et les exemples :

```
Library ID : /apache/echarts-doc (1442 snippets, doc officielle)
```

**Quand l'utiliser :**
- Quand l'utilisateur demande un type de graphique non couvert dans `references/service-patterns-iiot.md`
- Pour trouver les bonnes options ECharts pour un type de visualisation spécifique (heatmap, treemap, sankey, etc.)
- Pour des configurations avancées (dataZoom, visualMap, animations, thèmes)
- Pour vérifier la syntaxe d'une option ECharts

**Exemples de requêtes Context7 :**
- `"gauge chart configuration options"` → pour les jauges TRS
- `"stacked bar chart with categories"` → pour les pareto
- `"pie donut chart with custom colors"` → pour les répartitions
- `"line chart with time axis and dataZoom"` → pour les séries temporelles
- `"heatmap calendar configuration"` → pour les calendriers de production

## Processus de découverte API

```bash
# Lister les modules
ls /Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/src/data-databox/

# Scanner les endpoints d'un module
grep -n "@Get\|@Post\|@Delete\|@Patch\|@Controller" \
  /Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/src/data-databox/[module]/*.controller.ts
```

Endpoints standards (hérités d'AbstractController) :
`GET /` | `POST /` | `GET /:id` | `GET /idCorrespondence/:id` | `GET /code/:code` | `GET /columns` | `GET /dataShape` | `GET /columns-dictionary` | `GET /history/:idCorrespondence` | `POST /byDate`

## Références

| Fichier | Contenu |
|---------|---------|
| `references/xml-conventions.md` | Conventions de nommage, structure XML, permissions |
| `references/service-patterns.md` | Patterns de services ThingShape (code JS réel) |
| `references/mashup-patterns.md` | Structure JSON des mashups (Data, Bindings, Events, UI) |
| `references/tile-patterns.md` | Patterns par type de tile (Grid, Synth, Metric, Device) |
| `references/api-twx-mapping.md` | Mapping endpoints API ↔ services TWX + couverture |
| `references/widget-properties.md` | Propriétés des widgets FCAD (DataGrid, ECharts, GridStack) |
| `references/datashapes.md` | DataShapes existants et leurs champs |
| `references/helpers-api.md` | API des Things Helper (Platform, Dashboards) |
| `references/service-patterns-iiot.md` | Patterns IIOT : requêtes SQL (arrêts, pareto, TRS), InfluxDB, graphiques gauge, code réel |

## Workflows

| Workflow | Cas d'usage |
|----------|------------|
| `workflows/discover-api-endpoints.md` | Lister les endpoints API REST disponibles |
| `workflows/add-dataops-entity.md` | Ajouter une entité complète (service + dashboard + tiles) |
| `workflows/add-services-to-existing.md` | Ajouter des services à un ThingShape existant |
| `workflows/create-details-dashboard.md` | Créer un Dashboard Details (pattern Dashboard + Details) |
| `workflows/create-tile-catalog-entry.md` | Enregistrer une tile dans le catalogue |
| `workflows/create-iiot-services.md` | Créer des services IIOT (requêtes PostgreSQL/InfluxDB + graphiques ECharts) |
| `workflows/create-kpi-services.md` | Créer des services KPI DataOps (stats API + graphiques ECharts) |
| `workflows/add-iiot-tile.md` | Ajouter une tile IIOT pour un équipement |
