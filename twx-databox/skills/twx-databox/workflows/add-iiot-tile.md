# Workflow : Ajouter une tile IIOT

## Objectif
Créer une nouvelle tile pour un indicateur industriel (IIOT) : alertes, graphiques temps réel, timeline d'arrêts, ou indicateurs TRS/OEE.

## Types de tiles IIOT

| Type | Description | Données source | Widget principal |
|------|------------|----------------|------------------|
| Alerts | Alertes actives d'un équipement | PostgreSQL (ACTIVITY) | FCADDataGrid |
| PropertyGraph | Graphique temps réel d'une propriété | InfluxDB | FCADECharts |
| StopsTimeline | Chronologie des arrêts | PostgreSQL (ACTIVITY + CAUSE) | FCADVerticalTimeline |
| TRS/OEE | Indicateurs TRS (Dispo, Perf, Qualité) | PostgreSQL + InfluxDB | FCADECharts (gauge) |

## Processus

### Étape 1 : Identifier l'équipement et le type de tile

1. Demander le Thing cible (ex: "CXL_EQ_Perceuse")
2. Demander le type de tile souhaité
3. Pour PropertyGraph : demander la propriété à afficher

### Étape 2 : Lire les patterns existants

Les services IIOT sont définis dans les ThingShapes :
- `DataBox_TS_Devices` (49 services) - référence principale
- `DataBox_TS_TRS` (15 services) - indicateurs TRS

```bash
cat /Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/08-SourceControl/DataBox/ThingShapes/DataBox_TS_Devices.xml
```

### Étape 3 : Vérifier les services disponibles

Le Thing cible doit implémenter les ThingShapes nécessaires :
- Pour Alerts : DataBox_TS_Devices
- Pour PropertyGraph : DataBox_TS_Devices + propriétés loggées dans InfluxDB
- Pour StopsTimeline : DataBox_TS_TRS
- Pour TRS/OEE : DataBox_TS_TRS

**Si les services de collecte de données n'existent pas encore** (nouveau type d'indicateur, nouvelle requête), utiliser d'abord le workflow `create-iiot-services.md` pour créer :
1. Le service Data (requête PostgreSQL ou InfluxDB)
2. Le service Graph (transformation en options ECharts)

### Étape 4 : Générer la tile

Lire un mashup tile IIOT existant comme référence :

| Type de tile | Fichier de référence |
|-------------|---------------------|
| PropertyGraph | `Mashups/DataBox_Tile_DevicePropertyGraph.xml` |
| Alerts | `Mashups/DataBox_Tile_DeviceAlert.xml` |
| StopsTimeline | `Mashups/DataBox_Tile_DeviceStopTimeLine.xml` |
| TRS | `Mashups/DataBox_Tile_DeviceTRS.xml` |
| BooleanStats | `Mashups/DataBox_Tile_DeviceBooleanStats.xml` |
| Donut | `Mashups/DataBox_Tile_DeviceStopsDonutOccurence.xml` |
| Pareto | `Mashups/DataBox_Tile_DeviceStopsParetoTime.xml` |
| MTBF | `Mashups/DataBox_Tile_DeviceMTBF.xml` |

Créer le nouveau mashup en suivant le pattern du fichier de référence.

### Étape 5 : Enregistrer dans le catalogue (optionnel)

Utiliser le workflow `create-tile-catalog-entry.md` pour rendre la tile disponible dans le catalogue de sélection.

### Étape 6 : Checklist de validation

- [ ] Le fichier mashup commence par `<?xml version="1.0" encoding="UTF-8"?>` et `<Entities majorVersion="9" minorVersion="7">`
- [ ] `projectName="DataBox"` est présent
- [ ] Le nom suit la convention `DataBox_Tile_Device[Type]` ou `DataBox_Tile_[Entity]_[Type]`
- [ ] Les UUIDs dans le JSON mashup sont uniques (pas copiés d'un autre mashup)
- [ ] Les services référencés dans Data existent dans le ThingShape cible
- [ ] Les paramètres du mashup (thingName, property, startDate, endDate) sont déclarés si nécessaires
- [ ] Les DataBindings connectent le service Graph au widget FCADECharts (ou le service Data au FCADDataGrid pour Alerts)
- [ ] L'événement Loaded déclenche le chargement des données
- [ ] La ConfigurationTableDefinition `DataBox_DS_Tile` est présente pour le catalogue

### Étape 7 : Résumé

Présenter :
- Services créés (si applicable, via `create-iiot-services`)
- Fichier mashup tile créé avec chemin complet
- Instructions d'import dans ThingWorx
