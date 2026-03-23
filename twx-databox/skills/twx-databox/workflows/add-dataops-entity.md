# Workflow : Ajouter une entité DataOps complète

## Objectif
Créer tous les éléments ThingWorx nécessaires pour une nouvelle entité DataOps : ThingShape avec services, Dashboard, et Tiles.

## Prérequis
- L'entité doit avoir des endpoints dans l'API REST NestJS
- Utiliser le workflow `discover-api-endpoints.md` si nécessaire pour identifier les endpoints

## Processus

### Étape 1 : Identifier l'entité et ses endpoints

1. Demander à l'utilisateur le nom de l'entité (ex: "PurchaseOrders", "Receipts")
2. Scanner le controller NestJS correspondant :
   ```bash
   cat /Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/src/data-databox/[module]/*.controller.ts
   ```
3. Lister tous les endpoints disponibles (standards + custom)

### Étape 2 : Lire le controller NestJS et noter les noms EXACTS

**Critique** : les noms de sous-routes ne suivent aucun pattern régulier dans l'API. Il faut copier les noms exacts depuis les décorateurs `@Get()` du controller. Exemples de pièges :

| Entité | Sous-route lignes | Attention |
|--------|------------------|-----------|
| purchaseOrders | `purchaseOrdersLines` | Avec le **s** de Orders |
| salesOrders | `salesOrderLines` | **Sans** le s de Orders |
| invoices | `invoiceLines` | **Singulier** |
| quotes | `quoteLines` | **Singulier** |
| deliveries | `deliveryLines` | **Singulier** |
| serviceContracts | `serviceContractLines` | **Singulier** |

```bash
# Lire le controller pour obtenir les noms exacts
cat /Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/src/data-databox/[module]/*.controller.ts
```

Copier les noms tels quels dans les chemins API des services ThingWorx.

### Étape 3 : Lire un ThingShape de référence

Avant de générer, lire un ThingShape existant similaire pour comprendre les patterns exacts :

```bash
# Choisir le ThingShape le plus proche de l'entité cible
cat /Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/08-SourceControl/DataBox/ThingShapes/DataBox_TS_Accounts.xml
```

Consulter aussi `references/service-patterns.md` pour les patterns documentés.

### Étape 4 : Générer le ThingShape

Créer le fichier `08-SourceControl/DataBox/ThingShapes/DataBox_TS_[EntityName].xml` avec :

1. **ServiceDefinitions** - Déclarer les services :
   - `get[Entity](id, idcor)` → GET par ID ou IDCorrespondence
   - `get[Entity][SubEntity](id)` → GET sous-entités (relations)
   - `get[Entity]Lines(id[Entity])` → GET lignes
   - `get[Entity]GridConfig(small, pagination)` → Config grille
   - `get[Entity]LinesGridConfig(small, pagination)` → Config grille lignes
   - `setSelected[Entity](selected[Entity])` → Passthrough sélection

2. **ServiceImplementations** - Implémenter chaque service en JavaScript :
   - Utiliser `Things["DataBox_Helper_Platform"].GetInfotable({api: "..."})` pour les GET
   - Utiliser `me.GetGridConfig({table:"...", pagination:..., columns: []})` pour les GridConfig

3. **Permissions** - Appliquer les permissions standard

### Étape 5 : Générer le Dashboard (optionnel)

Lire un dashboard existant comme référence :
```bash
cat /Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/08-SourceControl/DataBox/Mashups/DataBox_Dashboard_Accounts.xml
```

Créer `08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[EntityName].xml`

### Étape 6 : Mettre à jour GetTableLinks

**Critique** : Pour que la navigation fonctionne dans les grilles FCADDataGrid (clic sur une ligne → ouverture du Dashboard Details), il faut ajouter une entrée dans le service `GetTableLinks` de `DataBox_Helper_Platform`.

Fichier : `08-SourceControl/DataBox/Things/DataBox_Helper_Platform.xml`
Service : `GetTableLinks` (vers ligne 935)

Ce service retourne un objet JSON qui mappe les noms de tables API vers les mashups Dashboard Details :

```javascript
result = {
    // Entrées existantes...
    accounts: "DataBox_Dashboard_AccountsDetails&code=",
    quotes: "DataBox_Dashboard_QuotesDetails&code=",
    // ...

    // AJOUTER pour la nouvelle entité :
    [table_api_name]: "DataBox_Dashboard_[EntityName]Details&code=",
    // Si l'entité a des lignes :
    [table_api_name]_lines: "DataBox_Dashboard_[EntityName]LinesDetails&code=",
};
```

**Convention du mapping :**
- Clé = nom de la table dans l'API REST (snake_case, ex: `purchase_orders`, `sales_orders_lines`)
- Valeur = `"[NomMashupDetails]&[paramètre]="` où paramètre est `code=` (par défaut) ou `idcor=` (pour correspondance)
- Pour les variantes avec idCorrespondence, ajouter une clé `_cor` : `purchase_orders_cor: "...&idcor="`

**Où c'est utilisé :** Les services `GetGridConfig` et `GetGridConfigPagination` appellent `me.GetTableLinks()` et fusionnent le résultat dans la config du FCADDataGrid via `Object.assign()`. Le widget utilise ces liens pour la navigation automatique.

### Étape 7 : Générer les Tiles (optionnel)

Selon les services disponibles, générer :
- Tile Grid (si fetchData disponible)
- Tile Details (si getById disponible)
- Tile Synth (si getLines disponible)

### Étape 8 : Checklist de validation

Avant de présenter le résultat, vérifier :

**ThingShape XML :**
- [ ] Le fichier commence par `<?xml version="1.0" encoding="UTF-8"?>` et `<Entities majorVersion="9" minorVersion="7" universal="password">`
- [ ] `projectName="DataBox"` est présent dans l'attribut ThingShape
- [ ] Chaque ServiceDefinition a une ServiceImplementation correspondante (même `name`)
- [ ] Les types de retour sont corrects (INFOTABLE avec `aspect.dataShape="DataBox_DS_Identification"` ou JSON)
- [ ] Les paramètres ont des noms cohérents avec les conventions (id, idcor, idAccount, small, pagination)
- [ ] Le code JavaScript dans CDATA utilise le bon Helper (`GetInfotable` pour INFOTABLE, `Get` pour JSON)
- [ ] Les chemins API correspondent **exactement** aux noms dans les `@Get()` du controller NestJS (ne pas deviner — les noms sont irréguliers)
- [ ] Les permissions standard sont présentes (ServiceInvoke pour Users, Visibility pour Everyone)
- [ ] La balise de fermeture `</Entities>` est présente

**GetTableLinks (si Dashboard Details généré) :**
- [ ] Une entrée `[table_name]: "DataBox_Dashboard_[Entity]Details&code="` est ajoutée dans GetTableLinks
- [ ] Si l'entité a des lignes, une entrée `[table_name]_lines` est aussi ajoutée
- [ ] Le nom de table correspond au nom utilisé dans l'API REST (snake_case)

**Dashboard XML (si généré) :**
- [ ] Le nom suit la convention `DataBox_Dashboard_[EntityName]`
- [ ] Les UUIDs dans le JSON sont uniques (pas copiés d'un autre mashup)
- [ ] Les services référencés dans Data existent réellement
- [ ] Les DataBindings connectent les bonnes sources aux bonnes cibles

### Étape 9 : Résumé

Présenter à l'utilisateur :
- Fichiers créés avec leur chemin complet
- Services générés (liste avec signature)
- Instructions pour importer dans ThingWorx
- Rappel : l'utilisateur doit charger manuellement dans ThingWorx puis faire `npm run download:projects`
