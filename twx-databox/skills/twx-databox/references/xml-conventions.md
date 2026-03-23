# Conventions XML ThingWorx - Projet DataBox

## Structure générale d'un fichier XML ThingWorx

Tous les fichiers XML du projet suivent cette structure :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Entities
 majorVersion="9"
 minorVersion="7"
 universal="password">
    <ThingShapes> <!-- ou Things, Mashups, DataShapes, ThingTemplates -->
        <!-- Entités ici -->
    </ThingShapes>
</Entities>
```

## Conventions de nommage

| Type | Convention | Exemples |
|------|-----------|----------|
| ThingShape DataOps | `DataBox_TS_[EntityName]` | DataBox_TS_Accounts, DataBox_TS_Quotes |
| ThingShape IIOT | `DataBox_TS_[Domain]` | DataBox_TS_Devices, DataBox_TS_TRS |
| ThingTemplate | `DataBox_TT_[Level]` | DataBox_TT_Site, DataBox_TT_Line, DataBox_TT_Workstation |
| Dashboard | `DataBox_Dashboard_[Entity]` | DataBox_Dashboard_Accounts, DataBox_Dashboard_Invoices |
| Dashboard Details | `DataBox_Dashboard_[Entity]Details` | DataBox_Dashboard_AccountsDetails |
| Dashboard Lines | `DataBox_Dashboard_[Entity]LinesDetails` | DataBox_Dashboard_InvoicesLinesDetails |
| Tile | `DataBox_Tile_[Entity]_[Type]` | DataBox_Tile_Accounts_Grid (convention déduite) |
| DataShape | `DataBox_DS_[Name]` | DataBox_DS_Identification, DataBox_DS_Dashboard |
| Thing Helper | `DataBox_Helper_[Domain]` | DataBox_Helper_Platform, DataBox_Helper_TRS |
| Thing Config | `DataBox_Configuration` | |
| Thing DB | `DataBox_DB`, `DataBox_DB_Influx` | |
| Project | `DataBox`, `DataBox_Install` | |

## Services - Conventions de nommage

| Pattern | Convention | Exemple |
|---------|-----------|---------|
| Récupérer une entité | `get[Entity]` | `getAccount`, `getQuote` |
| Récupérer des lignes | `get[Entity]Lines` | `getAddressLines`, `getContactLines` |
| Récupérer des sous-entités | `get[Entity][SubEntity]` | `getAccountQuotes`, `getAccountInvoices` |
| Config grille | `get[Entity]GridConfig` | `getAccountsGridConfig`, `getQuotesGridConfig` |
| Config grille lignes | `get[Entity]LinesGridConfig` | `getAddressLinesGridConfig` |
| Sélection | `setSelected[Entity]` | `setSelectedAccount` |

## Emplacement des fichiers

Les fichiers XML individuels sont dans `08-SourceControl/DataBox/` :

```
08-SourceControl/DataBox/
├── Things/           (9 fichiers)
├── ThingShapes/      (27 fichiers)
├── ThingTemplates/   (3 fichiers)
├── DataShapes/       (9 fichiers)
├── Mashups/          (157 fichiers)
├── Projects/         (1 fichier)
├── Groups/           (1 fichier)
├── StyleDefinitions/ (1 fichier)
├── ModelTags/        (9 fichiers)
├── MediaEntities/    (1 fichier)
└── ApplicationKeys/  (1 fichier)
```

## Projet et tags

Toutes les entités DataBox déclarent :
- `projectName="DataBox"` dans l'attribut de l'entité

## Permissions standard

```xml
<RunTimePermissions>
    <Permissions resourceName="*">
        <ServiceInvoke>
            <Principal isPermitted="true" name="Users" type="Group"></Principal>
        </ServiceInvoke>
    </Permissions>
</RunTimePermissions>
<VisibilityPermissions>
    <Visibility>
        <Principal isPermitted="true" name="Everyone" type="Organization"></Principal>
    </Visibility>
</VisibilityPermissions>
```
