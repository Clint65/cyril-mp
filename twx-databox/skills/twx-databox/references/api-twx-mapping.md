# Mapping Endpoints API REST ↔ Services ThingWorx

## Convention de mapping

| Endpoint API NestJS | Service ThingWorx | Pattern |
|--------------------|--------------------|---------|
| `GET /databox/[entity]` | `GetInfotable({api:"[entity]"})` | Liste complète |
| `POST /databox/[entity]` | `Post({api:"[entity]", body:...})` | Recherche paginée |
| `GET /databox/[entity]/:id` | `get[Entity](id)` | Par ID interne |
| `GET /databox/[entity]/idCorrespondence/:id` | `get[Entity](idcor)` | Par ID externe |
| `GET /databox/[entity]/code/:code` | (via Get) | Par code métier |
| `GET /databox/[entity]/columns` | `getGridConfig` (indirectement) | Colonnes |
| `GET /databox/[entity]/history/:id` | `GetHistory({api:...})` | Historique |
| `POST /databox/[entity]/byDate` | (via Post) | Par plage de dates |
| `GET /databox/[entity]/[subEntity]/:id` | `get[Entity][SubEntity](id)` | Relation |
| `GET /databox/[entity]/[entityLines]/:id` | `get[Entity]Lines(id)` | Sous-lignes |
| `POST /databox/[entity]/summary` | `get[Entity]Summary(...)` | Résumé/stats |
| `GET /databox/[entity]/catalog` | `get[Entity]Catalog()` | Catalogue |

## Couverture actuelle : ThingShapes existants vs Entités API

| Entité API | ThingShape TWX | Services | Statut |
|-----------|---------------|----------|--------|
| accounts | DataBox_TS_Accounts | 13 | ✅ Complet |
| articles | DataBox_TS_Articles | 17 | ✅ Complet |
| quotes | DataBox_TS_Quotes | 6 | ✅ Complet |
| salesOrders | DataBox_TS_SalesOrders | 5 | ✅ Complet |
| invoices | DataBox_TS_Invoices | — | ✅ Complet |
| deliveries | DataBox_TS_Deliveries | — | ✅ Complet |
| manufacturingOrders | DataBox_TS_ManufacturingOrders | — | ✅ Complet |
| salesReps | DataBox_TS_SalesReps | — | ✅ Complet |
| serviceContracts | DataBox_TS_ServiceContracts | — | ✅ Complet |
| qualityEvents | DataBox_TS_QualityEvents | — | ✅ Complet |
| customerAssets | DataBox_TS_CustomerAssets | — | ✅ Complet |
| customerDueDates | DataBox_TS_CustomerDueDates | — | ✅ Complet |
| projects | DataBox_TS_Projects | — | ✅ Complet |
| **purchaseOrders** | — | — | ❌ Manquant |
| **purchaseInvoices** | — | — | ❌ Manquant |
| **receipts** | — | — | ❌ Manquant |
| **openItems** | — | — | ❌ Manquant |
| **suppliers** | — | — | ❌ Manquant |
| **carriers** | — | — | ❌ Manquant |
| **bom** | (dans Articles) | getBOM | ⚠️ Partiel |
| **bomCommercial** | (dans Articles) | getBOMCommercial | ⚠️ Partiel |

## Entités candidates à la génération

Les entités suivantes ont des endpoints API mais pas de ThingShape dédié :

1. **purchaseOrders** - Commandes d'achat + purchaseOrdersLines + invoicingElements
2. **purchaseInvoices** - Factures d'achat + purchaseInvoicesLines
3. **receipts** - Réceptions + receiptsLines
4. **openItems** - Articles en attente
5. **suppliers** - Fournisseurs (même pattern que accounts) + addressLines + contactLines
6. **carriers** - Transporteurs (même pattern que accounts) + addressLines + contactLines

## Endpoints statistiques (datastats)

| Endpoint | Service TWX existant | Statut |
|----------|---------------------|--------|
| `POST /datastats/customers/quotesConversionRate` | getQuotesConversionRateGraph | ✅ |
| `POST /datastats/customers/quotesConversionValorisationRate` | — | ✅ |
| `POST /datastats/customers/quotesGlobalValorisationRate` | — | ✅ |
| `POST /datastats/customers/quotesQtyConversion` | — | ✅ |
| `POST /datastats/customers/topArticlesSoldQty` | — | ✅ |
| `POST /datastats/customers/topArticlesSoldAmount` | — | ✅ |
| `POST /datastats/customers/topArticlesForecastdAmount` | — | ✅ |
| `POST /datastats/customers/topArticlesForecastQty` | — | ✅ |
