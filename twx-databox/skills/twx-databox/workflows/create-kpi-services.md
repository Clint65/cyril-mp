# Workflow : Créer des services KPI DataOps (stats + graphiques)

## Objectif
Créer les services ThingWorx qui récupèrent des statistiques depuis l'API REST et génèrent les options ECharts pour les tiles Metric.

## Architecture des services KPI

Chaque indicateur KPI DataOps nécessite **2 services** dans le ThingShape `DataBox_TS_Dash_Comptes` :

```
Service Data  (appel API /datastats/)  →  Données brutes (JSON)
    ↓
Service Graph (transformation ECharts)  →  Options ECharts (JSON pour FCADECharts)
```

## Processus

### Étape 1 : Identifier le KPI et l'endpoint stats

Endpoints stats disponibles dans `databox-data-access/src/stats-databox/` :

| Endpoint | Description |
|----------|-------------|
| `POST /datastats/customers/quotesConversionRate` | Taux de conversion devis → commandes |
| `POST /datastats/customers/quotesConversionValorisationRate` | Valorisation de la conversion |
| `POST /datastats/customers/quotesGlobalValorisationRate` | Valorisation globale des devis |
| `POST /datastats/customers/quotesQtyConversion` | Conversion en quantité |
| `POST /datastats/customers/topArticlesSoldQty` | Top articles vendus (quantité) |
| `POST /datastats/customers/topArticlesSoldAmount` | Top articles vendus (montant) |
| `POST /datastats/customers/topArticlesForecastdAmount` | Top prévisionnel (montant) |
| `POST /datastats/customers/topArticlesForecastQty` | Top prévisionnel (quantité) |

### Étape 2 : Créer le service Data

**Signature type :**
```
get[Metric](startDate: DATETIME, endDate: DATETIME, selectedObjects: INFOTABLE[DS_Identification]) → JSON
```

**Pattern code JavaScript :**
```javascript
if (selectedObjects && selectedObjects.length > 0) {
    let ids = selectedObjects.rows.toArray().map((row) => row.idCorrespondence);
    let body = {
        ids: ids,
        startDate: startDate.getTime(),
        endDate: endDate.getTime()
    };
    let rate = Things["DataBox_Helper_Platform"].PostStats({
        api: "customers/[endpointName]",
        body: body
    });
    result = rate;
} else result = {};
```

### Étape 3 : Créer le service Graph

**Signature type :**
```
get[Metric]Graph(startDate: DATETIME, endDate: DATETIME, selectedObjects: INFOTABLE[DS_Identification]) → JSON
```

**Pattern code JavaScript :**
```javascript
let rate = me.get[Metric]({startDate: startDate, endDate: endDate, selectedObjects: selectedObjects});

let data = [];
let categories = [];

if (rate.array) {
    rate.array.forEach(row => {
        data.push(row.rate);  // ou row.value, row.amount selon le KPI
        categories.push(row.productCode ? ((row.productCode).split("|"))[2] : "???");
    });
}

result = {
    tooltip: {
        trigger: 'axis',
        axisPointer: { type: 'shadow' }
    },
    grid: {
        left: '3%', right: '4%', bottom: '3%', top: '3%',
        containLabel: true
    },
    yAxis: [{ type: 'category', data: categories }],
    xAxis: [{ type: 'value' }],
    series: [{
        name: '[MetricName]',
        type: 'bar',
        emphasis: { focus: 'series' },
        data: data
    }]
};
```

### Étape 4 : Créer le service Synth (optionnel)

Pour les tiles synthétiques qui combinent compteurs + graphique :

```javascript
let quotes = me.getDocumentsForSelected({table: "quotes/forAccounts", selectedObjects, startDate, endDate});
let so = me.getDocumentsForSelected({table: "salesOrders/forAccounts", selectedObjects, startDate, endDate});
let invoices = me.getDocumentsForSelected({table: "invoices/forAccounts", selectedObjects, startDate, endDate});

result = {
    quotesCount: quotes.rows.length,
    soCount: so.rows.length,
    invoicesCount: invoices.rows.length,
    graph: {
        // Options ECharts pour graphique comparatif
        yAxis: [{type: 'category', data: ['Montant TTC', 'Montant HT']}],
        xAxis: [{type: 'value'}],
        series: [
            {name: 'Devis', type: 'bar', data: [totalQuotesTTC, totalQuotesHT]},
            {name: 'Commandes', type: 'bar', data: [totalSOTTC, totalSOHT]},
            {name: 'Factures', type: 'bar', data: [totalInvTTC, totalInvHT]}
        ]
    }
};
```

### Étape 5 : Consulter la doc ECharts si besoin

Pour des types de graphiques avancés (heatmap, treemap, sankey, radar, etc.), consulter la documentation ECharts via Context7 :
```
mcp__plugin_context7_context7__query-docs(libraryId: "/apache/echarts-doc", query: "type de graphique souhaité")
```

### Étape 6 : Où placer les services

- **KPIs Accounts (Dash_Comptes)** : dans `DataBox_TS_Dash_Comptes`
- **Nouveau ThingShape KPI** : créer `DataBox_TS_Dash_[Domain]` si c'est un nouveau domaine

### Étape 7 : Référence existante

Lire le ThingShape existant pour le pattern exact :
```bash
cat 08-SourceControl/DataBox/ThingShapes/DataBox_TS_Dash_Comptes.xml
```

### Checklist de validation

**Service Data :**
- [ ] Signature : `get[Metric](startDate: DATETIME, endDate: DATETIME, selectedObjects: INFOTABLE[DS_Identification]) → JSON`
- [ ] Le code vérifie `selectedObjects && selectedObjects.length > 0` avant d'appeler l'API
- [ ] L'appel utilise `Things["DataBox_Helper_Platform"].PostStats({api: "customers/...", body: ...})`
- [ ] Les dates sont converties avec `.getTime()` (millisecondes) dans le body
- [ ] Le retour par défaut est `{}` si pas de sélection

**Service Graph :**
- [ ] Signature : `get[Metric]Graph(startDate, endDate, selectedObjects) → JSON`
- [ ] Appelle le service Data correspondant (`me.get[Metric]({...})`)
- [ ] Transforme les données en options ECharts valides
- [ ] Le JSON contient au minimum : `tooltip`, `grid`, `xAxis`/`yAxis`, `series`

**XML :**
- [ ] ServiceDefinition et ServiceImplementation ont le même `name`
- [ ] Les paramètres startDate, endDate sont de type DATETIME
- [ ] Le paramètre selectedObjects a `aspect.dataShape="DataBox_DS_Identification"`
- [ ] Le type de retour est JSON (pas INFOTABLE)

