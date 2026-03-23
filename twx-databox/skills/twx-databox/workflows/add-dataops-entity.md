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

### Étape 2 : Lire un ThingShape de référence

Avant de générer, lire un ThingShape existant similaire pour comprendre les patterns exacts :

```bash
# Choisir le ThingShape le plus proche de l'entité cible
cat /Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/08-SourceControl/DataBox/ThingShapes/DataBox_TS_Accounts.xml
```

Consulter aussi `references/service-patterns.md` pour les patterns documentés.

### Étape 3 : Générer le ThingShape

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

### Étape 4 : Générer le Dashboard (optionnel)

Lire un dashboard existant comme référence :
```bash
cat /Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/08-SourceControl/DataBox/Mashups/DataBox_Dashboard_Accounts.xml
```

Créer `08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[EntityName].xml`

### Étape 5 : Générer les Tiles (optionnel)

Selon les services disponibles, générer :
- Tile Grid (si fetchData disponible)
- Tile Details (si getById disponible)
- Tile Synth (si getLines disponible)

### Étape 6 : Checklist de validation

Avant de présenter le résultat, vérifier :

**ThingShape XML :**
- [ ] Le fichier commence par `<?xml version="1.0" encoding="UTF-8"?>` et `<Entities majorVersion="9" minorVersion="7" universal="password">`
- [ ] `projectName="DataBox"` est présent dans l'attribut ThingShape
- [ ] Chaque ServiceDefinition a une ServiceImplementation correspondante (même `name`)
- [ ] Les types de retour sont corrects (INFOTABLE avec `aspect.dataShape="DataBox_DS_Identification"` ou JSON)
- [ ] Les paramètres ont des noms cohérents avec les conventions (id, idcor, idAccount, small, pagination)
- [ ] Le code JavaScript dans CDATA utilise le bon Helper (`GetInfotable` pour INFOTABLE, `Get` pour JSON)
- [ ] Les chemins API correspondent aux endpoints réels du controller NestJS
- [ ] Les permissions standard sont présentes (ServiceInvoke pour Users, Visibility pour Everyone)
- [ ] La balise de fermeture `</Entities>` est présente

**Dashboard XML (si généré) :**
- [ ] Le nom suit la convention `DataBox_Dashboard_[EntityName]`
- [ ] Les UUIDs dans le JSON sont uniques (pas copiés d'un autre mashup)
- [ ] Les services référencés dans Data existent réellement
- [ ] Les DataBindings connectent les bonnes sources aux bonnes cibles

### Étape 7 : Résumé

Présenter à l'utilisateur :
- Fichiers créés avec leur chemin complet
- Services générés (liste avec signature)
- Instructions pour importer dans ThingWorx
- Rappel : l'utilisateur doit charger manuellement dans ThingWorx puis faire `npm run download:projects`
