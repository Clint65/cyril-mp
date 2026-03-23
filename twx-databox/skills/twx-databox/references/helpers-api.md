# API des Things Helper

## Table des matières

1. [DataBox_Helper_Platform](#databox_helper_platform) - Appels API REST, grilles, pagination
2. [DataBox_Helper_Platform_Dashboards](#databox_helper_platform_dashboards) - Grilles tiles, catalogue, colonnes
3. [DataBox_Helper_Devices](#databox_helper_devices) - Simulation données IIOT
4. [DataBox_Helper_Devices_Dashboards](#databox_helper_devices_dashboards) - Catalogue tiles devices, graphiques string
5. [DataBox_Helper_TRS (via DataBox_TS_TRS)](#databox_helper_trs) - TRS, MTBF, Pareto, gestion arrêts
6. [Quand utiliser quel Helper ?](#quand-utiliser-quel-helper-)

---

## DataBox_Helper_Platform

Thing principal pour les appels API REST et la gestion des données.
Fichier : `08-SourceControl/DataBox/Things/DataBox_Helper_Platform.xml`

### Get(api) → JSON

Appel GET à l'API REST. Retourne du JSON brut.

```javascript
let params = {
    method: "GET",
    headers: {"Accept-Language": "fr-FR"},
    url: Things["DataBox_Configuration"].api_base_url + "/databox/" + api,
    timeout: 2000,
    ignoreSSLErrors: true
};
let result = Resources["ContentLoaderFunctions"].GetJSON(params);
```

### Post(api, body) → JSON

Appel POST à l'API REST.

```javascript
let params = {
    headers: {"Accept-Language": "fr"},
    url: Things["DataBox_Configuration"].api_base_url + "/databox/" + api,
    ignoreSSLErrors: true,
    content: body
};
let result = Resources["ContentLoaderFunctions"].PostJSON(params);
```

### GetInfotable(api) → INFOTABLE[DataBox_DS_Identification]

Appel GET avec header `Accept: thingworx/infotable`. Retourne une INFOTABLE ThingWorx.

```javascript
let params = {
    headers: {Accept: "thingworx/infotable", "Accept-Language": "fr-FR"},
    method: "GET",
    url: Things["DataBox_Configuration"].api_base_url + "/databox/" + api,
    timeout: 2000,
    ignoreSSLErrors: true
};
let data = Resources["ContentLoaderFunctions"].GetJSON(params);
let json = {dataShape: data.dataShape, rows: data.rows};
let result = Resources["InfoTableFunctions"].FromJSON({json: data});
```

### FetchData(table, size, page, filter, sort, id, ids, startDate, endDate) → JSON

Recherche paginée avec filtres et tri. Utilisé par FCADDataGrid en mode pagination serveur.

```javascript
let fil = filter ? filter.array : [];
let sor = sort ? sort.array : [];
let api = id ? table + "/" + id : table;
ids = (ids && ids.array) ? ids.array : [];
let res = me.Post({
    api: api,
    body: {page: page, size: size, filter: fil, sort: sor, ids: ids, startDate: startDate, endDate: endDate}
});
result = {last_page: res.last_page, data: res.data};
```

### GetGridConfig(table, columns, pagination) → JSON

Configuration de grille pour FCADDataGrid (mode local, colonnes filtrées).

```javascript
let storedColumnConfiguration = me.GetGridColumns({table: table});
if (columns && columns.array && columns.array.length > 0) {
    let cols = JSON.parse(columns.array);
    storedColumnConfiguration.array = storedColumnConfiguration.array.filter(col => cols.includes(col.field));
}
result = {
    layout: "fitDataStretch",
    renderHorizontal: "virtual",
    debugInvalidOptions: false,
    selectableRows: 1,
    movableColumns: true,
    columns: storedColumnConfiguration.array
};
if (pagination) {
    result["pagination"] = "local";
    result["paginationSize"] = 10;
    result["paginationSizeSelector"] = [3, 6, 8, 10];
}
```

### GetGridConfigPagination(table) → JSON

Configuration de grille avec pagination serveur (progressive scroll). Utilisé pour les listes principales.

```javascript
let storedColumnConfiguration = me.GetGridColumns({table: table});
result = {
    layout: "fitDataFill",
    debugInvalidOptions: false,
    selectableRows: 1,
    movableColumns: true,
    columns: storedColumnConfiguration.array,
    ajaxURL: "/Thingworx/Things/DataBox_Helper_Platform/services/FetchData",
    progressiveLoad: "scroll",
    progressiveLoadScrollMargin: 300,
    ajaxConfig: "POST",
    ajaxContentType: "json",
    ajaxParams: {table: table},
    filterMode: "remote",
    sortMode: "remote"
};
```

### GetHistory(table, idCorrespondence, id) → INFOTABLE[DataBox_DS.Timeline]

Historique des modifications d'une entité, formaté pour FCADVerticalTimeline.

### saveInfotableConfig(input) → INFOTABLE

Passthrough pour persister une configuration dans un mashup parameter.

---

## DataBox_Helper_Platform_Dashboards

Thing pour la gestion des dashboards, tiles et configurations de grilles dans les tiles.
Fichier : `08-SourceControl/DataBox/Things/DataBox_Helper_Platform_Dashboards.xml`

### getGridConfig(table, columnVisibility, pagination) → JSON

Configuration de grille pour les tiles (avec visibilité colonnes par utilisateur).

```javascript
let storedColumnConfiguration = me.getColumns({table: table, columnVisibility: columnVisibility});
result = {
    layout: "fitDataStretch",
    debugInvalidOptions: false,
    selectableRows: false,
    movableColumns: true,
    columns: storedColumnConfiguration.array
};
if (pagination) {
    result["pagination"] = "local";
    result["paginationSize"] = 10;
    result["paginationSizeSelector"] = [3, 6, 8, 10];
}
```

**Différence avec Helper_Platform.GetGridConfig :** utilise `columnVisibility` (INFOTABLE DS_GridColumnVisibility) pour filtrer les colonnes par utilisateur, et `selectableRows: false`.

### getTileCatalog(dashboardName) → JSON

Retourne le catalogue de tiles pour un dashboard donné. Scanne les mashups par tags.

### getColumnVisibility(existingColumnVisibility, table) → INFOTABLE[DS_GridColumnVisibility]

Retourne la visibilité des colonnes. Si vide, initialise depuis l'API `/columns` avec Common visible et ERP/PLM/CRM/Identification masqués.

### getDocumentation(fileName) → TEXT

Charge un fichier markdown depuis `SystemRepository/TileDocumentation/`.

### saveTileConfig(input) → INFOTABLE

Passthrough pour persister la configuration d'une tile.

---

---

## DataBox_Helper_Devices

Thing pour la simulation de données IIOT (test/démo).
Fichier : `08-SourceControl/DataBox/Things/DataBox_Helper_Devices.xml`
Implémente : DataBox_TS_Network

### mirror(input) → INFOTABLE

Passthrough simple.

### simulattion(thing, property, previous, min, max, maxChange, volatility) → NOTHING

Simule une valeur avec variation aléatoire contrôlée. Utilisé pour les démos sans équipement réel.

| Paramètre | Type | Default | Description |
|-----------|------|---------|-------------|
| thing | STRING | | Nom du Thing cible |
| property | STRING | | Propriété à modifier |
| previous | NUMBER | | Valeur précédente |
| min | INTEGER | | Valeur minimale |
| max | INTEGER | | Valeur maximale |
| maxChange | NUMBER | 2.0 | Changement maximum par tick |
| volatility | NUMBER | 0.5 | Facteur d'aléatoire (0-1) |

```javascript
const randomFactor = (Math.random() - 0.5) * 2;
const change = randomFactor * volatility * maxChange;
let newTemp = Math.max(min, Math.min(max, previous + change));
Things[thing][property] = Math.round(newTemp * 10) / 10;
```

---

## DataBox_Helper_Devices_Dashboards

Thing pour les dashboards devices (catalogue tiles, graphiques).
Fichier : `08-SourceControl/DataBox/Things/DataBox_Helper_Devices_Dashboards.xml`
Implémente : DataBox_TS_Devices

### getTileCatalog() → JSON

Catalogue de tiles pour les dashboards devices. Structure en 3 sections :
- **Capteurs** (DeviceProperty) : Général, Numériques, Textes, Booléens
- **Indicateurs** (DeviceIndicators) : Performance
- **Objets liés** (DeviceLinkObjects) : Production

Utilise les tags de mashups pour filtrer : `"Dashboards:Dash_Device;[Vocabulary]:[SubSection]"`

### getStringPropertyGraph(config, startDate, endDate) → JSON

Graphique ECharts pour l'historique de propriétés string. Récupère via `DataBox_DB_Influx.getStringPropertyHistory()` et construit un scatter chart avec points de contexte sur des lignes Y séparées.

---

## DataBox_Helper_TRS

Thing pour les calculs TRS, MTBF, Pareto et gestion des arrêts.
Fichier : `08-SourceControl/DataBox/Things/DataBox_Helper_TRS.xml`
Implémente : DataBox_TS_TRS, DataBox_TS_TreeCause
Services définis dans le ThingShape DataBox_TS_TRS.

### Services d'analyse

| Service | Paramètres | Retour | Description |
|---------|-----------|--------|-------------|
| `calculcateTRSIndicators` | thingName, startDate, endDate, groupBy | JSON | Calcul TRS complet (NF E60-182) |
| `computeTRS` | thingName, downTimesByType | JSON | Calcul indicateurs depuis temps d'arrêt |
| `getMTBF` | thingName, startDate, endDate, groupBy | INFOTABLE | Mean Time Between Failures |
| `getParetoData` | thingName, startDate, endDate | JSON | Analyse Pareto (occurrence + durée) |
| `getParetoDataOccurence` | thingName, startDate, endDate | INFOTABLE | Pareto par nombre d'arrêts |
| `getParetoDataTime` | thingName, startDate, endDate | INFOTABLE | Pareto par durée totale |
| `getParetoDataRunning` | thingName, startDate, endDate | JSON | Pareto détaillé avec listes d'arrêts |
| `getDowntimesSumTimeOccurence` | thingName, startDate, endDate | INFOTABLE | Résumé par cause |
| `getDowntimesSumTimeOccurenceGroupBy` | thingName, startDate, endDate, groupBy, include* | INFOTABLE | Résumé par période et cause |
| `getDowntimesByCauseAndElement` | thingName, startDate, endDate, index, include* | JSON | Arrêts détaillés filtrés |
| `durationToString` | duration (LONG) | STRING | Secondes → "Xj XXhXXmXXs" |

### Services de gestion des arrêts

| Service | Paramètres | Retour | Description |
|---------|-----------|--------|-------------|
| `startActivity` | startDate, causeId, dataCustom, comment, division | LONG | Démarre un arrêt (INSERT) |
| `stopActivity` | activityId, stopDate | NOTHING | Clôture un arrêt (UPDATE) |
| `updateActivityComment` | activityId, comment | NOTHING | Modifie le commentaire |

### Paramètre groupBy

Utilisé par calculcateTRSIndicators, getMTBF, getDowntimesSumTimeOccurenceGroupBy :

| Valeur | Description |
|--------|-------------|
| `"none"` | Pas de groupement (résultat unique pour la période) |
| `"day"` | Groupement par jour |
| `"week"` | Groupement par semaine |
| `"month"` | Groupement par mois |

### Paramètres include* (filtres par type de cause)

| Paramètre | Default | Description |
|-----------|---------|-------------|
| includeRunning | true | Inclure fonctionnement normal |
| includeClosing | true | Inclure fermetures |
| includeScheduled | true | Inclure arrêts planifiés |
| includeUnscheduled | true | Inclure arrêts non planifiés |

---

## Quand utiliser quel Helper ?

| Cas d'usage | Helper | Service |
|-------------|--------|---------|
| Appel API GET → JSON | Platform | Get(api) |
| Appel API GET → INFOTABLE | Platform | GetInfotable(api) |
| Appel API POST | Platform | Post(api, body) |
| Pagination serveur (Grid principale) | Platform | FetchData(...) |
| Config grille dans Dashboard | Platform | GetGridConfigPagination(table) |
| Config grille dans ThingShape (small) | Platform | GetGridConfig(table, columns, pagination) |
| Config grille dans Tile | Platform_Dashboards | getGridConfig(table, columnVisibility, pagination) |
| Catalogue de tiles DataOps | Platform_Dashboards | getTileCatalog(dashboardName) |
| Catalogue de tiles Devices | Devices_Dashboards | getTileCatalog() |
| Visibilité colonnes | Platform_Dashboards | getColumnVisibility(...) |
| Historique entité | Platform | GetHistory(table, idCorrespondence, id) |
| Calcul TRS/OEE | Helper_TRS (via TS_TRS) | calculcateTRSIndicators(...) |
| Analyse Pareto arrêts | Helper_TRS (via TS_TRS) | getParetoData(...) |
| MTBF | Helper_TRS (via TS_TRS) | getMTBF(...) |
| Démarrer/arrêter un arrêt | Helper_TRS (via TS_TRS) | startActivity/stopActivity |
| Simulation données | Helper_Devices | simulattion(...) |
| Graphique propriétés string | Devices_Dashboards | getStringPropertyGraph(...) |
