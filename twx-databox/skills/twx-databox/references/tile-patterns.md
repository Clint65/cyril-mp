# Patterns de Tiles par Type

## Table des matières

1. [Caractéristiques communes](#caractéristiques-communes-à-toutes-les-tiles)
2. [Tile Grid](#type-1--tile-grid-liste-de-données) - FCADDataGrid
3. [Tile Synth](#type-2--tile-synth-synthèse-de-documents) - Compteurs/totaux
4. [Tile Metric/KPI](#type-3--tile-metrickpi-graphiques) - FCADECharts
5. [Tile Device IIOT](#type-4--tile-device-iiot) - Graphs/Alerts/Timeline
6. [Tiles avec Configuration](#tiles-avec-configuration-_config)
7. [Structure XML commune d'une Tile](#structure-xml-commune-dune-tile)

---

## Caractéristiques communes à toutes les tiles

- `aspect.isFlex="true"` et `aspect.isResponsive="true"`
- `projectName="DataBox"`
- Tags pour le filtrage catalogue : `"[Entity]:[Type]"`
- ConfigurationTableDefinition référençant `DataBox_DS_Tile` (voir workflow create-tile-catalog-entry)
- Sections Data toujours avec Session et UserExtensions

## Type 1 : Tile Grid (liste de données)

**Convention de nommage :** `DataBox_Tile_[Entity]_Grid`
**Référence :** `DataBox_Tile_Accounts_Grid.xml`

**Structure :**
- Un widget FCADDataGrid comme élément principal
- Service `GetInfotable` ou `FetchData` pour charger les données
- Service `getGridConfig` pour la configuration des colonnes
- Événement `Details` pour naviguer vers les détails
- Événement `Selected` pour stocker la sélection en Session
- Services `ExportXLS` / `ExportCSV` liés à des boutons

**Flux de données :**
```
Mashup Loaded
  → getGridConfig (Things_DataBox_Helper_Platform_Dashboards)
    → ServiceInvokeCompleted
      → GetInfotable (Things_DataBox_Helper_Platform) avec api="[entity]"
        → Data → FCADDataGrid.Data
  → getGridConfig.result → FCADDataGrid.Config
```

**Bindings clés :**
```
getGridConfig.result (JSON) → FCADDataGrid.Config
GetInfotable (INFOTABLE) → FCADDataGrid.Data
FCADDataGrid.SelectedRows → Session.selected[Entity]
Session.selected[Entity] → FCADDataGrid.SelectedRows (restauration)
FCADDataGrid.ActionId → navigationfunction.code (pour Details)
Mashup.columnsVisibility → navigationfunction (pour config colonnes)
Mashup.columnsVisibility → getGridConfig.columnVisibility (paramètre)
```

## Type 2 : Tile Synth (synthèse de documents)

**Convention de nommage :** `DataBox_Tile_[Entity]_Synth`
**Référence :** `DataBox_Tile_AccountsDocuments_Synth.xml`

**Structure :**
- Conteneurs flex pour afficher des compteurs/totaux
- Service `get[Entity]Synth` qui retourne des données agrégées
- Souvent accompagné d'un mashup `_Synth_Details` pour le détail

**Flux de données :**
```
Mashup Loaded (ou paramètre reçu)
  → get[Entity]Synth (Things_DataBox_Helper_Platform_Dashboards)
    → Data → container-synth (widget conteneur)
```

**Variante Synth + Details :**
- `DataBox_Tile_[Entity]_Synth` : Vue synthétique (compteurs, totaux)
- `DataBox_Tile_[Entity]_Synth_Details` : Vue détaillée avec FCADDataGrid

## Type 3 : Tile Metric/KPI (graphiques)

**Convention de nommage :** `DataBox_Tile_[Entity][Metric]`
**Référence :** `DataBox_Tile_AccountsQuotesConversionRate.xml`

**Structure :**
- Widget FCADECharts pour le graphique
- Service qui retourne les données formatées pour ECharts
- Souvent accompagné d'un mashup `Details` avec un FCADDataGrid

**Flux de données :**
```
Mashup Loaded (ou paramètre dates reçu)
  → get[Metric]Graph (Things_DataBox_Helper_Platform_Dashboards)
    → Data → chart-widget (FCADECharts)
```

**Types de métriques existantes :**
- Taux de conversion (ConversionRate)
- Valorisation (ValorisationRate, GlobalValorisationRate)
- Top articles (TopSoldArticles, TopForecastArticles)
- Taux commandes/devis (SalesOrdersQuotesRate)

## Type 4 : Tile Device (IIOT)

**Convention de nommage :** `DataBox_Tile_Device[Type]`
**Exemples :** DevicePropertyGraph, DeviceAlert, DeviceStops, DeviceTRS, DeviceBooleanStats

**Structure :**
- Widget FCADECharts (graphs) ou FCADDataGrid (alertes) ou FCADVerticalTimeline (stops)
- Services appelés sur le Thing device directement ou via helpers
- Paramètres : thingName, property, startDate, endDate

**Sous-types :**

| Tile | Widget | Service | Description |
|------|--------|---------|-------------|
| DevicePropertyGraph | FCADECharts | getNumberPropertyHistory | Graphique temps réel d'une propriété |
| DeviceBooleanStats | FCADECharts | getCountAndDurationBoolean | Stats on/off d'une propriété |
| DeviceAlert | FCADDataGrid | getAlerts | Liste des alertes actives |
| DeviceStops | FCADDataGrid | getStops | Liste des arrêts |
| DeviceStopTimeLine | FCADVerticalTimeline | getStopsTimeline | Chronologie des arrêts |
| DeviceTRS | FCADECharts (gauge) | getTRS | Indicateurs TRS/OEE |
| DeviceMTBF | FCADECharts | getMTBF | Mean Time Between Failures |
| DeviceStopsDonut | FCADECharts (pie) | getStopsDonut | Répartition des arrêts |
| DeviceStopsPareto | FCADECharts (bar) | getStopsPareto | Pareto des causes d'arrêt |

## Tiles avec Configuration (_Config)

Certaines tiles ont un mashup de configuration associé :
- `DataBox_Tile_DevicePropertyGraph.xml` + `DataBox_Tile_DevicePropertyGraph_Config.xml`
- `DataBox_Tile_DeviceAlert.xml` + `DataBox_Tile_DeviceAlert_Config.xml`

Le mashup _Config permet à l'utilisateur de paramétrer la tile (choisir la propriété, les seuils, etc.)

## Structure XML commune d'une Tile

Extrait réel de `DataBox_Tile_Accounts_Grid.xml` (simplifié pour montrer la structure) :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Entities majorVersion="9" minorVersion="7" universal="password">
  <Mashups>
    <Mashup
     aspect.isFlex="true"
     aspect.isResponsive="true"
     aspect.mashupType="mashup"
     columns="0.0"
     description="Accounts"
     name="DataBox_Tile_Accounts_Grid"
     projectName="DataBox"
     rows="0.0"
     tags="Accounts:List">

      <!-- Permissions standard -->
      <DesignTimePermissions>...</DesignTimePermissions>
      <RunTimePermissions></RunTimePermissions>
      <VisibilityPermissions>
        <Visibility>
          <Principal isPermitted="true" name="Everyone" type="Organization"/>
        </Visibility>
      </VisibilityPermissions>

      <!-- ConfigurationTable pour le catalogue de tiles -->
      <ConfigurationTableDefinitions>
        <ConfigurationTableDefinition
         category="" dataShapeName="DataBox_DS_Tile" description=""
         isHidden="false" isMultiRow="false" name="TileConfiguration"
         ordinal="0" source="IMPORT"/>
      </ConfigurationTableDefinitions>

      <ConfigurationTables>
        <!-- MobileSettings (toujours présent) -->
        <ConfigurationTable dataShapeName="" description="Mashup Mobile Settings"
         isHidden="true" isMultiRow="false" name="MobileSettings" ordinal="0">
          <Rows><Row>
            <disableZoom>false</disableZoom>
            <fullScreenMode>true</fullScreenMode>
            <heightType>device-height</heightType>
            <widthType>device-width</widthType>
          </Row></Rows>
        </ConfigurationTable>
      </ConfigurationTables>

      <ParameterDefinitions></ParameterDefinitions>

      <!-- Contenu JSON du mashup -->
      <mashupContent><![CDATA[
      {
        "CustomMashupCss": "",
        "Data": {
          "Session": {
            "DataName": "Session", "EntityType": "Session", "Id": "session",
            "Services": [{"Name": "GetGlobalSessionValues", "Target": "GetGlobalSessionValues"}]
          },
          "Things_DataBox_Helper_Platform": {
            "DataName": "Things_DataBox_Helper_Platform",
            "EntityName": "DataBox_Helper_Platform",
            "EntityType": "Things",
            "Id": "unique-uuid-1",
            "Services": [{
              "APIMethod": "post", "Id": "unique-uuid-2",
              "Name": "GetInfotable",
              "Parameters": {"api": "accounts"},
              "Target": "GetInfotable"
            }]
          },
          "Things_DataBox_Helper_Platform_Dashboards": {
            "DataName": "Things_DataBox_Helper_Platform_Dashboards",
            "EntityName": "DataBox_Helper_Platform_Dashboards",
            "EntityType": "Things",
            "Id": "unique-uuid-3",
            "Services": [{
              "APIMethod": "post", "Id": "unique-uuid-4",
              "Name": "getGridConfig",
              "Parameters": {"table": "accounts"},
              "Target": "getGridConfig"
            }]
          },
          "UserExtensions": {
            "DataName": "UserExtensions", "EntityType": "UserExtensions", "Id": "UserExtensions",
            "Services": [{"Name": "GetCurrentUserExtensionProperties", "Target": "GetCurrentUserExtensionProperties"}]
          }
        },
        "DataBindings": [
          {
            "PropertyMaps": [{"SourceProperty": "result", "TargetProperty": "Config"}],
            "SourceArea": "Data", "SourceId": "getGridConfig",
            "SourceSection": "Things_DataBox_Helper_Platform_Dashboards",
            "TargetArea": "UI", "TargetId": "FCADDataGrid-6"
          },
          {
            "PropertyMaps": [{"SourceProperty": "", "SourcePropertyType": "InfoTable", "TargetProperty": "Data"}],
            "SourceArea": "Data", "SourceDetails": "AllData",
            "SourceId": "GetInfotable", "SourceSection": "Things_DataBox_Helper_Platform",
            "TargetArea": "UI", "TargetId": "FCADDataGrid-6"
          }
        ],
        "Events": [
          {
            "EventTriggerEvent": "Loaded", "EventTriggerId": "mashup-root",
            "EventHandlerId": "Things_DataBox_Helper_Platform_Dashboards",
            "EventHandlerService": "getGridConfig"
          },
          {
            "EventTriggerEvent": "ServiceInvokeCompleted",
            "EventTriggerId": "getGridConfig",
            "EventTriggerSection": "Things_DataBox_Helper_Platform_Dashboards",
            "EventHandlerId": "Things_DataBox_Helper_Platform",
            "EventHandlerService": "GetInfotable"
          },
          {
            "EventTriggerEvent": "Details", "EventTriggerId": "FCADDataGrid-6",
            "EventHandlerId": "navigationfunction-13",
            "EventHandlerService": "Navigate"
          }
        ],
        "UI": {
          "Properties": {
            "Id": "mashup-root", "Type": "mashup",
            "Master": "DataBox_Master", "ResponsiveLayout": true
          },
          "Widgets": [{
            "Properties": {
              "Type": "flexcontainer", "Id": "flexcontainer-2",
              "flex-direction": "column", "flex-grow": 1
            },
            "Widgets": [{
              "Properties": {
                "Type": "FCADDataGrid", "Id": "FCADDataGrid-6",
                "flex-grow": 1
              }
            }]
          }]
        }
      }
      ]]></mashupContent>
    </Mashup>
  </Mashups>
</Entities>
```

**Points clés pour générer une nouvelle tile :**
- Remplacer `"accounts"` par le nom de l'entité cible dans les Parameters des services
- Générer des UUIDs uniques pour chaque Id (ne jamais copier ceux d'un mashup existant)
- Adapter les tags pour le catalogue (`"[Entity]:List"` pour Grid, `"[Entity]:KPI"` pour Metric)
- Le flux est toujours : Loaded → getGridConfig → ServiceInvokeCompleted → GetInfotable → FCADDataGrid
