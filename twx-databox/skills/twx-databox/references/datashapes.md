# DataShapes du projet DataBox

9 DataShapes dans `08-SourceControl/DataBox/DataShapes/`

## Table des matières

1. [DS_Identification](#databox_ds_identification) - Entité métier principale (retour des services DataOps)
2. [DS_Dashboard](#databox_ds_dashboard) - Stockage dashboards utilisateur
3. [DS_DashboardDevice](#databox_ds_dashboarddevice) - Dashboards par équipement
4. [DS_Tile](#databox_ds_tile) - Métadonnées tiles (catalogue)
5. [DS_Menu](#databox_ds_menu) - Entrées du menu de navigation
6. [DS_GridColumnVisibility](#databox_ds_gridcolumnvisibility) - Visibilité colonnes par utilisateur
7. [DS_PropertyValues](#databox_ds_propertyvalues) - Valeurs de propriétés avec unités
8. [DS_NumericProperty](#databox_ds_numericproperty) - Configuration graphiques propriétés
9. [DS.Timeline](#databox_dstimeline) - Événements chronologiques

## Relations entre DataShapes

```
DS_Identification ──────── Retourné par tous les services DataOps (get[Entity], get[Entity]Lines, etc.)
                           Utilisé comme paramètre selectedObjects dans les services KPI

DS_Dashboard ──────────── Stocké dans DATABOX.Dashboards (PostgreSQL)
DS_DashboardDevice ────── Stocké dans DATABOX.Dashboards avec thing_name

DS_Tile ───────────────── ConfigurationTable de chaque mashup Tile
                           Lu par getTileCatalog pour construire le catalogue

DS_Menu ───────────────── Retourné par getMainMenu, getUserSideMenu (navigation)

DS_GridColumnVisibility ── Paramètre de getGridConfig (Dashboards Helper)
                           Persisté par utilisateur dans le mashup

DS.Timeline ───────────── Retourné par GetHistory (historique des modifications)
                           Utilisé comme Data du widget FCADVerticalTimeline

DS_NumericProperty ────── Configuration des graphiques de propriétés InfluxDB
DS_PropertyValues ─────── Valeurs live des propriétés d'un device
```

---

## DataBox_DS_Identification

DataShape principal utilisé comme type de retour de la plupart des services DataOps.

| Champ | Type | PK | Default | Description |
|-------|------|----|---------|-------------|
| code | STRING | | | Code métier |
| endDate | DATETIME | | | Date fin |
| idBom | STRING | | | ID nomenclature |
| idCorrespondence | STRING | | | ID externe (ERP/CRM/PLM) |
| idDatabox | NUMBER | | | ID interne Databox |
| isHistory | BOOLEAN | | false | Est un historique |
| isLatest | BOOLEAN | | true | Est la version courante |
| parentId | STRING | | | ID parent |
| srcSystemType | STRING | | | Système source |
| startDate | DATETIME | | | Date début |
| tgtSystemType | STRING | | | Système cible |

**Exemple JSON (retour d'un service getAccount) :**
```json
{
  "idDatabox": 42,
  "idCorrespondence": "ACC-001",
  "code": "CLI001",
  "startDate": "2025-01-15T00:00:00Z",
  "endDate": null,
  "srcSystemType": "ERP",
  "tgtSystemType": "CRM",
  "isLatest": true,
  "isHistory": false,
  "parentId": null,
  "idBom": null
}
```

## DataBox_DS_Dashboard

Stockage des dashboards utilisateur (table DATABOX.Dashboards).

| Champ | Type | Description |
|-------|------|-------------|
| dashboards_content | JSON | Contenu JSON du dashboard (tiles, positions) |
| dashboards_group | STRING | Groupe du dashboard |
| dashboards_id | LONG | ID auto-incrémenté |
| dashboards_is_shared | BOOLEAN | Partagé ou non |
| dashboards_name | STRING | Nom du dashboard |
| dashboards_owner | STRING | Propriétaire |

## DataBox_DS_DashboardDevice

Dashboards spécifiques à un équipement.

| Champ | Type | Description |
|-------|------|-------------|
| dashboards_content | JSON | Contenu JSON |
| dashboards_id | STRING | ID |
| dashboards_is_shared | BOOLEAN | Partagé |
| dashboards_name | STRING | Nom |
| dashboards_owner | STRING | Propriétaire |
| dashboards_thing_name | STRING | Nom du Thing associé |

## DataBox_DS_Tile

Métadonnées des tiles pour le catalogue. Utilisé dans ConfigurationTableDefinitions des mashups Tile.

| Champ | Type | PK | Description |
|-------|------|----|-------------|
| tileName | STRING | **PK** | Nom du mashup tile |
| tileDescription | STRING | | Description |
| tileDocumentation | STRING | | Path vers fichier .md de documentation |
| tileImage | STRING | | Path vers icône PNG |
| tileWidth | NUMBER | | Largeur par défaut dans GridStack |
| tileHeight | NUMBER | | Hauteur par défaut |
| tileMenu | JSON | | Configuration menu (catégorie, sous-catégorie) |
| classes | STRING | | Classes CSS |
| connect | JSON | | Configuration de connexion |
| locked | BOOLEAN | | Verrouillée |
| moveable | BOOLEAN | | Déplaçable |
| resizable | BOOLEAN | | Redimensionnable |
| order | NUMBER | | Ordre d'affichage |

## DataBox_DS_Menu

Structure des entrées du menu de navigation.

| Champ | Type | Description |
|-------|------|-------------|
| type | STRING | Type d'entrée |
| title | STRING | Titre affiché |
| icon | STRING | Classe d'icône (FontAwesome/Material) |
| mashup | STRING | Nom du mashup cible |
| dashboard | STRING | Nom du dashboard |
| dashboardId | LONG | ID du dashboard |
| badge | STRING | Badge (compteur) |
| badgeColor | STRING | Couleur du badge |
| parent | INTEGER | ID parent (hiérarchie) |
| section | STRING | Section du menu |
| group | STRING | Groupe |
| id | STRING | Identifiant |

**Exemple JSON (ConfigurationTable d'une tile) :**
```json
{
  "tileName": "DataBox_Tile_Accounts_Grid",
  "tileDescription": "Liste des comptes clients",
  "tileDocumentation": "DataBox_Tile_Accounts_Grid.md",
  "tileImage": "/TileIcon/DataBox_Tile_Accounts_Grid.png",
  "tileWidth": 12,
  "tileHeight": 6,
  "tileMenu": {"category": "Comptes", "subcategory": "Listes"},
  "classes": "",
  "connect": null,
  "locked": false,
  "moveable": true,
  "resizable": true,
  "order": 1
}
```

## DataBox_DS_GridColumnVisibility

Visibilité des colonnes par utilisateur.

| Champ | Type | Description |
|-------|------|-------------|
| field | STRING | Nom du champ |
| title | STRING | Titre de la colonne |
| visible | BOOLEAN | Visible ou non |

## DataBox_DS_PropertyValues

Valeurs de propriétés avec unités.

| Champ | Type | Description |
|-------|------|-------------|
| name | STRING | Nom de la propriété |
| value | STRING | Valeur |
| type | STRING | Type de donnée |
| unit | STRING | Unité de mesure |

## DataBox_DS_NumericProperty

Configuration des graphiques de propriétés numériques.

| Champ | Type | Description |
|-------|------|-------------|
| name | STRING | Nom de la propriété |
| displayPoints | BOOLEAN | Afficher les points |
| step | STRING | Type de step |
| axis | STRING | Axe Y associé |
| smooth | BOOLEAN | Courbe lissée |

## DataBox_DS.Timeline

Structure pour les timelines (historique, arrêts).

| Champ | Type | Description |
|-------|------|-------------|
| DateField | DATETIME | Date de l'événement |
| ColorField | STRING | Couleur (primary/secondary/tertiary) |
| LabelField | STRING | Label |
| MessageField | STRING | Message détaillé |
| IconField | STRING | Icône |
| IdField | STRING | Identifiant |
