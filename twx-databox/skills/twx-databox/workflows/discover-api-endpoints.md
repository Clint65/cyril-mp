# Workflow : Découvrir les endpoints API REST

## Objectif
Scanner le code source NestJS de databox-data-access pour lister tous les endpoints API disponibles et identifier ceux qui ne sont pas encore couverts par des ThingShapes ThingWorx.

## Processus

### Étape 1 : Lister les modules API

```bash
ls /Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/src/data-databox/
```

Chaque dossier correspond à une entité métier (accounts, articles, quotes, etc.).

### Étape 2 : Scanner les controllers

Pour chaque module, extraire les endpoints depuis le controller :

```bash
# Scanner tous les controllers d'un coup
grep -rn "@Controller\|@Get\|@Post\|@Delete\|@Patch" \
  /Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/src/data-databox/ \
  --include="*.controller.ts"
```

Pour un module spécifique, lire le controller complet :
```bash
cat /Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/src/data-databox/[module]/*.controller.ts
```

Extraire pour chaque endpoint :
- Le décorateur `@Controller('databox/[path]')` → chemin de base
- Les décorateurs `@Get('subpath')`, `@Post('subpath')` → routes custom
- Les paramètres `@Param('id')`, `@Body()`, `@Query()` → paramètres

### Étape 3 : Identifier les endpoints hérités vs custom

Les controllers héritent d'AbstractController qui fournit 10 endpoints standards :

| Endpoint hérité | Verbe | Route |
|----------------|-------|-------|
| findAll | GET | `/` |
| findAllPagination | POST | `/` |
| findById | GET | `/:id` |
| findByIdCorrespondence | GET | `/idCorrespondence/:id` |
| findByCode | GET | `/code/:code` |
| getColumns | GET | `/columns` |
| getDataShape | GET | `/dataShape` |
| getColumnsDictionary | GET | `/columns-dictionary` |
| getHistory | GET | `/history/:idCorrespondence` |
| findAllByDate | POST | `/byDate` |

Tout endpoint dans le controller qui n'est **pas** dans cette liste est un endpoint **custom** (relations, lignes, summary, catalog).

### Étape 4 : Lister les ThingShapes existants

```bash
ls /Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/08-SourceControl/DataBox/ThingShapes/
```

Convention : module `accounts/` → ThingShape `DataBox_TS_Accounts`

### Étape 5 : Croiser et générer le rapport de couverture

Pour chaque module API, vérifier si un ThingShape existe. Présenter le résultat en tableau :

```
| Module API        | ThingShape TWX           | Statut      | Endpoints custom non couverts |
|-------------------|--------------------------|-------------|-------------------------------|
| accounts          | DataBox_TS_Accounts      | COUVERT     | -                             |
| purchaseOrders    | -                        | NON COUVERT | purchaseOrdersLines, invoicingElements |
| suppliers         | -                        | NON COUVERT | addressLines, contactLines    |
```

Pour les modules couverts, vérifier aussi si les endpoints custom sont tous implémentés :
```bash
# Comparer endpoints du controller vs services du ThingShape
grep 'name="get' 08-SourceControl/DataBox/ThingShapes/DataBox_TS_[Entity].xml
```

### Étape 6 : Proposer les actions

Pour chaque entité NON COUVERTE, indiquer :
- L'action recommandée (créer ThingShape via `add-dataops-entity`, ou ajouter services via `add-services-to-existing`)
- Les endpoints custom à couvrir
- Le ThingShape de référence le plus similaire

### Checklist de validation

- [ ] Tous les dossiers dans `src/data-databox/` ont été scannés
- [ ] Les endpoints hérités d'AbstractController sont identifiés séparément des endpoints custom
- [ ] Le tableau de couverture compare modules API vs ThingShapes existants
- [ ] Les endpoints custom non couverts sont listés pour chaque entité
- [ ] Les recommandations d'action sont fournies
