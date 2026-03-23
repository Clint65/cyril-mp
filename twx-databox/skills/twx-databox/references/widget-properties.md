# Propriétés des Widgets FCAD

Sources : repos GitLab `4cad-group/rd-4cad/data-platform/dashboarding/extensions/`

## Table des matières

1. [FCADDataGrid](#fcaddatagrid-databox-twx-grid) - Grille de données (Tabulator)
2. [FCADECharts](#fcadecharts-databox-twx-echarts) - Graphiques (Apache ECharts)
3. [FCADGridStack2](#fcadgridstack2-databox-twx-gridstack) - Conteneur dashboard drag-and-drop
4. [FCADVerticalTimeline](#fcadverticaltimeline-databox-twx-verticaltimeline) - Timeline verticale

---

## FCADDataGrid (databox-twx-grid)

Basé sur **Tabulator**. Widget principal pour afficher des listes de données.

### Propriétés principales

| Propriété | Type | Binding | Description |
|-----------|------|---------|-------------|
| Data | INFOTABLE | target | Données du grid |
| DataJSON | JSON | target | Données alternatives en JSON |
| Config | JSON | target | Configuration Tabulator (colonnes, layout, pagination) |
| SelectedRows | INFOTABLE | source | Lignes sélectionnées |
| PreSelectedRows | INFOTABLE | target | Pré-sélection de lignes |
| SelectedRow | JSON | source | Ligne sélectionnée (objet unique) |
| ActionId | STRING | source | ID de la ligne lors d'une action |
| HasSelectedRow | BOOLEAN | source | Indique si une ligne est sélectionnée |
| IdField | FIELDNAME | - | Champ identifiant unique |
| ParentField | FIELDNAME | - | Champ parent (mode arbre) |
| IsTree | BOOLEAN | - | Active le mode arborescent |
| IsMenu | BOOLEAN | - | Mode menu (clic simple sélectionne) |
| CheckboxSelection | BOOLEAN | - | Colonne de cases à cocher |
| AutoSelectFirstRow | BOOLEAN | - | Sélection auto première ligne |
| ColumnEditable | BOOLEAN | - | Menu visibilité des colonnes |
| MakeTile | BOOLEAN | - | Style carte (ombre, coins arrondis) |
| FullHeight | BOOLEAN | - | Occupe 100% de la hauteur |
| DateFormat | STRING | - | Format moment.js (défaut: "DD/MM/YYYY HH:mm:ss") |
| Master | STRING | - | Nom du mashup maître pour navigation |
| DataOut | INFOTABLE | source | Données après modification |
| Menus | JSON | target | Menus contextuels |
| Buttons | JSON | target | Définitions de boutons |

### Événements

| Événement | Description |
|-----------|-------------|
| changed | Données modifiées (édition, suppression, réorganisation) |
| selectionChanged | Sélection de lignes modifiée |
| DeleteRow | Ligne supprimée |
| Details | Clic sur bouton détails d'une ligne → propage ActionId |
| [custom] | Événements custom via propriété `Events` (JSON array de noms) |

### Services

| Service | Description |
|---------|-------------|
| ExportCSV | Exporte en CSV |
| ExportXLS | Exporte en Excel |
| save | Sauvegarde (déclenche changed) |
| addRow | Ajoute une ligne vide |
| deleteRow | Supprime la sélection |
| ShowLoading | Affiche le spinner |

### Format Config

```json
{
  "layout": "fitDataStretch",
  "debugInvalidOptions": false,
  "selectableRows": 1,
  "movableColumns": true,
  "columns": [
    {"field": "code", "title": "Code", "width": 120, "headerFilter": true},
    {"field": "name", "title": "Nom"},
    {"field": "date", "title": "Date", "type": "date"},
    {"field": "amount", "title": "Montant", "type": "number"}
  ],
  "pagination": "local",
  "paginationSize": 10,
  "paginationSizeSelector": [3, 6, 8, 10]
}
```

### Types de colonnes

`date`, `duration`, `icon`, `color`, `boolean`, `number`, `array`, `arrayLink`, `json`, `link`, `lookupArray`

---

## FCADECharts (databox-twx-echarts)

Basé sur **Apache ECharts**. Widget de visualisation (graphiques, jauges, camemberts).

### Propriétés principales

| Propriété | Type | Binding | Description |
|-----------|------|---------|-------------|
| Options | JSON | target | Configuration ECharts complète |
| OptionsText | STRING | target | Configuration en texte |
| formatXAxisDate | BOOLEAN | - | Formate l'axe X comme dates |
| dateFormat | STRING | - | Format dayjs (défaut: "DD/MM/YYYY HH:mm:ss") |
| theme | STRING | - | Thème: default, chalk, dark, fcad, fcad2, infographic, macaron, walden, wonderland |
| customTheme | JSON | - | Thème personnalisé |
| legendSelectedOne | BOOLEAN | - | Mode bouton radio pour légendes |
| resetSeries | BOOLEAN | - | replaceMerge: 'series' |
| synchronize | BOOLEAN | - | Synchronise zoom entre graphiques |
| selectedLegends | JSON | source | Légendes sélectionnées |
| selectedStartDate | DATETIME | source | Date début après zoom |
| selectedEndDate | DATETIME | source | Date fin après zoom |
| mouseEvent | JSON | source | Données du clic |

### Événements

| Événement | Description |
|-----------|-------------|
| mouseEventChanged | Clic sur un élément du graphique |

### Services

| Service | Description |
|---------|-------------|
| exportCSV | Exporte les données en CSV |
| showLoading | Affiche le spinner |
| hideLoading | Cache le spinner |

### Types de graphiques supportés

`line`, `bar`, `pie`, `scatter`, `radar`, `gauge`, `heatmap`, `treemap`, `sunburst`, `funnel`, `sankey`, `boxplot`, `candlestick`, `graph`, `tree`, `custom`

### Raccourcis de formatage durée

`"secondString"`, `"minuteString"`, `"hourString"`, `"dayString"` → convertit en format "Xd HH:MM:SS"

### Exemple minimal

```json
{
  "xAxis": {"type": "category", "data": ["Jan", "Feb", "Mar"]},
  "yAxis": {"type": "value"},
  "series": [{"data": [120, 200, 150], "type": "bar"}]
}
```

---

## FCADGridStack2 (databox-twx-gridstack)

Conteneur de dashboard avec tiles drag-and-drop.

### Propriétés principales

| Propriété | Type | Binding | Description |
|-----------|------|---------|-------------|
| DataJSON | JSON | source+target | Configuration des tiles (array) |
| Config | JSON | target | Configuration GridStack |
| Disable | BOOLEAN | target | Désactive drag/drop |
| NeedSaving | BOOLEAN | source | État non sauvegardé |
| TilesToAdd | JSON | target | Tiles à ajouter |
| BubbleValues | JSON | target | Valeurs de badges |
| ParameterValues | JSON | target | Paramètres par tile |
| GlobalParameterValues | JSON | target | Paramètres globaux |
| UpdatedParameters | JSON | source | Paramètres mis à jour |
| AutoSave | BOOLEAN | - | Sauvegarde auto |
| AsyncLoading | BOOLEAN | - | Chargement asynchrone des tiles |

### Événements

| Événement | Description |
|-----------|-------------|
| changed | Disposition modifiée (drag, resize) |
| removed | Tile supprimée |

### Services

| Service | Description |
|---------|-------------|
| save | Sauvegarde l'état (met à jour DataJSON) |
| setupDragin | Configure le drag-in externe |

### Format DataJSON (tiles)

```json
[{
  "id": "tile-1",
  "mashup": "DataBox_Tile_Accounts_Grid",
  "x": 0, "y": 0, "width": 12, "height": 6,
  "resizable": true, "moveable": true, "locked": false,
  "classes": "", "order": 1
}]
```

### Config par défaut

```json
{
  "float": true, "column": 24, "animate": true,
  "acceptWidgets": true, "removable": true,
  "handle": ".dragme",
  "marginTop": "10px", "marginBottom": "10px",
  "marginLeft": "10px", "marginRight": "10px",
  "cellHeight": "auto", "minRow": 24
}
```

---

## FCADVerticalTimeline (databox-twx-verticaltimeline)

Timeline verticale pour afficher des événements chronologiques (arrêts, activités).

### Propriétés principales

| Propriété | Type | Binding | Description |
|-----------|------|---------|-------------|
| Data | INFOTABLE | target | Données de la timeline |
| IconField | FIELDNAME | - | Champ icône |
| ColorField | FIELDNAME | - | Champ couleur (primary/secondary/tertiary/success/danger/warning) |
| DateField | FIELDNAME | - | Champ date/timestamp |
| LabelField | FIELDNAME | - | Champ label |
| MessageField | FIELDNAME | - | Champ message |
| TypeField | FIELDNAME | - | Champ type (PANEL ou LINE) |
| DataEventField | FIELDNAME | - | Champ boutons action (JSON) |
| DataEvents | JSON | - | Boutons d'action globaux |
| NumberOfEvents | INTEGER | - | Nombre d'événements dynamiques (défaut: 5) |
| TimelinePosition | STRING | - | Position: "left", "center", "right" |
| EventsOrder | STRING | target | Ordre: "desc" ou "asc" |
| MilestonesFrequency | STRING | - | Jalons: "none", "day", "week", "month" |
| EventDateFormat | STRING | - | Format dates événements |
| AllowSelection | BOOLEAN | - | Permet la sélection |

### Événements

Event1 à EventN (dynamiques selon NumberOfEvents) + SelectionChanged

### Couleurs disponibles

`primary`, `secondary`, `tertiary`, `success`, `danger`, `warning`, `link`, `alert`, `info`
