# Patterns de Mashups - Dashboard et Structure JSON

## Table des matières

1. [Structure XML d'un Mashup](#structure-xml-dun-mashup)
2. [Structure JSON du mashupContent](#structure-json-du-mashupcontent)
   - [Data - Sources de données](#1-data---sources-de-données)
   - [DataBindings - Liaisons](#3-databindings---liaisons-de-données)
   - [Events - Événements](#4-events---gestion-dévénements)
   - [UI - Arbre de widgets](#5-ui---arbre-de-widgets)
3. [Pattern Dashboard (conteneur)](#pattern-dashboard-conteneur)
4. [Pattern de navigation Dashboard → Details](#pattern-de-navigation-dashboard--details)
5. [Fichiers de référence existants](#fichiers-de-référence-existants)

---

## Structure XML d'un Mashup

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Entities majorVersion="9" minorVersion="7" universal="password">
  <Mashups>
    <Mashup
     aspect.isFlex="true"
     aspect.isResponsive="true"
     aspect.mashupType="mashup"
     columns="0.0"
     description="[Description]"
     documentationContent=""
     homeMashup=""
     name="[MashupName]"
     projectName="DataBox"
     rows="0.0"
     tags="[Tags]">

      <DesignTimePermissions>...</DesignTimePermissions>
      <RunTimePermissions></RunTimePermissions>
      <VisibilityPermissions>
          <Visibility>
              <Principal isPermitted="true" name="Everyone" type="Organization"></Principal>
          </Visibility>
      </VisibilityPermissions>
      <ConfigurationTableDefinitions>...</ConfigurationTableDefinitions>
      <ConfigurationTables>...</ConfigurationTables>
      <ParameterDefinitions></ParameterDefinitions>
      <Things></Things>
      <ThingShapes></ThingShapes>
      <ThingTemplates></ThingTemplates>
      <mashupContent><![CDATA[
        { /* JSON Structure */ }
      ]]></mashupContent>
    </Mashup>
  </Mashups>
</Entities>
```

## Structure JSON du mashupContent

Le JSON encodé en CDATA contient 5 sections principales :

### 1. CustomMashupCss
```json
"CustomMashupCss": ""
```

### 2. Data - Sources de données
Déclare les Things/Services appelés par le mashup.

```json
"Data": {
  "Session": {
    "DataName": "Session",
    "EntityType": "Session",
    "Id": "session",
    "Services": [{
      "APIMethod": "post",
      "Characteristic": "Services",
      "Id": "SessionInterface",
      "Name": "GetGlobalSessionValues",
      "Parameters": {},
      "Target": "GetGlobalSessionValues"
    }]
  },
  "Things_DataBox_Helper_Platform": {
    "DataName": "Things_DataBox_Helper_Platform",
    "EntityName": "DataBox_Helper_Platform",
    "EntityType": "Things",
    "Id": "[UUID]",
    "Services": [{
      "APIMethod": "post",
      "Characteristic": "Services",
      "Id": "[UUID]",
      "Name": "[ServiceName]",
      "Parameters": { "param1": "value1" },
      "Target": "[ServiceName]",
      "Caching": {
        "cache": false,
        "cacheKeyParams": [],
        "cacheStrategy": "INSTANCE",
        "maxResultSets": "10"
      }
    }]
  },
  "UserExtensions": {
    "DataName": "UserExtensions",
    "EntityType": "UserExtensions",
    "Id": "UserExtensions",
    "Services": [{
      "Name": "GetCurrentUserExtensionProperties",
      "Target": "GetCurrentUserExtensionProperties"
    }]
  }
}
```

**Règles :**
- Session et UserExtensions sont toujours présents
- Le DataName pour un Thing suit le pattern `Things_[ThingName]`
- Chaque service a un UUID unique pour son Id
- Les Parameters contiennent les valeurs statiques des paramètres

### 3. DataBindings - Liaisons de données
Connecte les sorties de services/widgets aux entrées d'autres widgets/services.

```json
"DataBindings": [{
  "Id": "[UUID]",
  "PropertyMaps": [{
    "SourceProperty": "result",
    "SourcePropertyBaseType": "JSON",
    "SourcePropertyType": "Field",
    "TargetProperty": "Config",
    "TargetPropertyBaseType": "JSON",
    "TargetPropertyType": "property"
  }],
  "SourceArea": "Data",
  "SourceDetails": "AllData",
  "SourceId": "[ServiceName]",
  "SourceSection": "Things_[ThingName]",
  "TargetArea": "UI",
  "TargetId": "[WidgetId]",
  "TargetSection": ""
}]
```

**Areas possibles :** UI, Data, Mashup, Session
**SourceDetails possibles :** AllData, SelectedRows
**PropertyTypes :** property, Field, InfoTable, Parameter, Property

### 4. Events - Gestion d'événements
Connecte les événements aux handlers (services, navigation, etc.)

```json
"Events": [{
  "Id": "[UUID]",
  "EventTriggerArea": "Mashup",
  "EventTriggerEvent": "Loaded",
  "EventTriggerId": "mashup-root",
  "EventHandlerArea": "Data",
  "EventHandlerId": "Things_[ThingName]",
  "EventHandlerService": "[ServiceName]"
}]
```

**Événements courants :**
- `Mashup > Loaded` : Déclenche le chargement initial des données
- `UI > Details` (FCADDataGrid) : Clic sur une ligne → navigation détails
- `UI > Clicked` (button) : Clic bouton → action
- `Data > ServiceInvokeCompleted` : Service terminé → charger données suivantes
- `Mashup > gridStackConfigChanged` : Config grille modifiée → sauvegarder

### 5. UI - Arbre de widgets
```json
"UI": {
  "Properties": {
    "Area": "Mashup",
    "Id": "mashup-root",
    "Master": "DataBox_Master",
    "ResponsiveLayout": true,
    "Style": "DefaultMashupStyle",
    "StyleTheme": "PTC Convergence Theme",
    "Type": "mashup",
    "Width": 1024,
    "Height": 618
  },
  "Widgets": [/* arbre de widgets */]
}
```

## Pattern Dashboard (conteneur)

Un Dashboard utilise typiquement :
- Master: `DataBox_Master` (navigation, menu, header)
- Un `flexcontainer` racine en `column`
- FCADGridStack2 comme conteneur de tiles (ou directement des tiles en flex)
- Appel service `getGridConfig` au Loaded
- Navigation vers Details via expression + navigationfunction

## Pattern de navigation Dashboard → Details

```json
// 1. Expression construit l'URL
"Events": [{
  "EventTriggerEvent": "Details",
  "EventTriggerId": "FCADDataGrid-14",
  "EventHandlerId": "expression2-15",
  "EventHandlerService": "Evaluate"
}],
// 2. NavigationFunction navigue
"Events": [{
  "EventTriggerEvent": "Changed",
  "EventTriggerId": "expression2-15",
  "EventHandlerId": "navigationfunction-10",
  "EventHandlerService": "Navigate"
}],
// 3. Binding expression output → navigation URL
"DataBindings": [{
  "SourceId": "expression2-15",
  "SourceProperty": "Output",
  "TargetId": "navigationfunction-10",
  "TargetProperty": "URL"
}]
```

## Fichiers de référence existants

Pour générer un nouveau mashup, toujours lire un mashup existant similaire :

| Type | Fichier de référence |
|------|---------------------|
| Dashboard liste | `DataBox_Dashboard_Accounts.xml` |
| Dashboard details | `DataBox_Dashboard_AccountsDetails.xml` |
| Tile Grid | `DataBox_Tile_Accounts_Grid.xml` |
| Tile Synth | `DataBox_Tile_AccountsDocuments_Synth.xml` |
| Tile Metric/KPI | `DataBox_Tile_AccountsQuotesConversionRate.xml` |
| Tile Device | `DataBox_Tile_DevicePropertyGraph.xml` |
