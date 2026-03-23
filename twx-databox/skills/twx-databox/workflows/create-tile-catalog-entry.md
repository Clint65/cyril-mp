# Workflow : Enregistrer une tile dans le catalogue

## Objectif
Chaque tile doit être enregistrée dans le système de catalogue pour apparaître dans l'interface de sélection de tiles. Cela implique la ConfigurationTableDefinition dans le mashup tile et optionnellement une documentation et une icône.

## Système de catalogue

Le catalogue de tiles est géré par :
- **DataBox_CatalogTile** (mashup) - Interface de sélection
- **DataBox_Helper_Platform_Dashboards.getTileCatalog** - Service qui liste les tiles
- **DataBox_DS_Tile** (DataShape) - Structure des métadonnées de tile

## DataShape DS_Tile

| Champ | Type | Description |
|-------|------|-------------|
| tileName | STRING (PK) | Nom du mashup tile |
| width | NUMBER | Largeur par défaut dans GridStack |
| height | NUMBER | Hauteur par défaut dans GridStack |
| image | STRING | Chemin vers l'icône PNG |
| menu | JSON | Configuration menu (catégorie, sous-catégorie) |
| classes | STRING | Classes CSS additionnelles |
| connect | JSON | Configuration de connexion (paramètres passés) |
| order | NUMBER | Ordre d'affichage dans le catalogue |

## Processus

### Étape 1 : Ajouter la ConfigurationTableDefinition dans la tile

Chaque mashup tile doit déclarer une ConfigurationTableDefinition référençant DS_Tile :

```xml
<ConfigurationTableDefinition
 category=""
 dataShapeName="DataBox_DS_Tile"
 description=""
 isHidden="false"
 isMultiRow="false"
 name="TileConfiguration"
 ordinal="0"
 source="IMPORT"></ConfigurationTableDefinition>
```

Et la ConfigurationTable correspondante avec les valeurs :

```xml
<ConfigurationTable dataShapeName="DataBox_DS_Tile" description=""
 isMultiRow="false" name="TileConfiguration" ordinal="0">
    <Rows>
        <Row>
            <tileName><![CDATA[DataBox_Tile_[Entity]_[Type]]]></tileName>
            <width><![CDATA[6]]></width>
            <height><![CDATA[4]]></height>
            <image><![CDATA[/TileIcon/[entity]_[type].png]]></image>
            <menu><json><![CDATA[{"category":"[Category]","subcategory":"[SubCategory]"}]]></json></menu>
            <order><![CDATA[10]]></order>
        </Row>
    </Rows>
</ConfigurationTable>
```

### Étape 2 : Créer la documentation (optionnel)

Créer un fichier markdown dans :
```
07-Repositories/SystemRepository/TileDocumentation/DataBox_Tile_[Entity]_[Type].md
```

Ce fichier décrit ce que la tile affiche et comment l'utiliser.

### Étape 3 : Créer l'icône (optionnel)

Ajouter un fichier PNG dans :
```
07-Repositories/SystemRepository/TileIcon/[entity]_[type].png
```

Taille recommandée : 200x150 pixels.

### Étape 4 : Configurer les tags du mashup

Les tags du mashup tile permettent le filtrage dans le catalogue :

```xml
<Mashup ... tags="[Entity]:[Type];Dashboards:[Dashboard]">
```

Exemples :
- `tags="Accounts:List"` pour une tile Grid
- `tags="Dashboards:Dash_Comptes;Quotes:Synth"` pour une tile Synth
- `tags="Dashboards:Dash_Comptes;Quotes:KPI"` pour une tile Metric
