# Workflow : Créer des services IIOT (collecte de données + graphiques)

## Objectif
Créer les services ThingWorx qui collectent des données IIOT (PostgreSQL pour arrêts/activités, InfluxDB pour séries temporelles) et les services qui génèrent les options ECharts pour les graphiques.

## Architecture des services IIOT

Chaque indicateur IIOT nécessite typiquement **2 services** :

```
Service Data (collecte)          →  Données brutes (JSON ou INFOTABLE)
    ↓
Service Graph (visualisation)    →  Options ECharts (JSON pour FCADECharts)
```

## Types de services à créer

### Type 1 : Requête PostgreSQL (arrêts, activités, causes)

**Pattern :** Utilise `Things["DataBox_DB"].runQuery()`

```javascript
// Exemple : Récupérer les arrêts d'un device sur une période
let timezone = Things["DataBox_Configuration"].timezone || 'Europe/Paris';
let sql = `
    SELECT a.activity_id, a.activity_start AT TIME ZONE '${timezone}' as activity_start,
           a.activity_end AT TIME ZONE '${timezone}' as activity_end,
           a.activity_cause_id, c.cause_name, c.cause_color,
           a.activity_comment, a.activity_type,
           EXTRACT(EPOCH FROM (COALESCE(a.activity_end, NOW()) - a.activity_start)) as duration_seconds
    FROM DATABOX.ACTIVITY a
    LEFT JOIN DATABOX.CAUSE c ON a.activity_cause_id = c.cause_id
    WHERE a.activity_division = '${thingName}'
    AND a.activity_start >= '${startDate.toISOString()}'
    AND a.activity_start <= '${endDate.toISOString()}'
    ORDER BY a.activity_start DESC
`;

result = Things["DataBox_DB"].runQuery({
    sql: sql,
    maxItems: 500,
    timeout: 30
});
```

**Tables disponibles :**
- `DATABOX.ACTIVITY` : Arrêts et activités (start, end, cause, division, comment)
- `DATABOX.CAUSE` : Arbre des causes (id, name, color, parent, tree_type)

### Type 2 : Requête InfluxDB (séries temporelles)

**Pattern :** Utilise `Things["DataBox_DB_Influx"]` avec ses services spécialisés.

```javascript
// Comptage et durée d'une propriété booléenne
let data = Things["DataBox_DB_Influx"].getCountAndDurationBoolean({
    thing: thingName,       // THINGNAME
    property: propertyName, // STRING
    startDate: startDate,   // DATETIME
    endDate: endDate        // DATETIME
});
result = data;
```

**Services InfluxDB disponibles :**

| Service | Pour type | Retourne |
|---------|-----------|----------|
| `getCountBoolean(thing, property, start, end)` | BOOLEAN | Nombre de changements |
| `getDurationBoolean(thing, property, start, end)` | BOOLEAN | Durée true/false |
| `getCountAndDurationBoolean(thing, property, start, end)` | BOOLEAN | Count + durée combinés |
| `getCountString(thing, property, start, end)` | STRING | Nombre par valeur |
| `getDurationString(thing, property, start, end)` | STRING | Durée par valeur |
| `getPeriodCountAndDurationString(thing, property, start, end)` | STRING | Count + durée par période |
| `getCountNumber(thing, property, start, end)` | NUMBER | Comptage |
| `getPeriodCountConditionNumber(thing, property, start, end)` | NUMBER | Comptage par période avec condition |
| `getPeriodDurationCondtionNumber(thing, property, start, end)` | NUMBER | Durée par période avec condition |
| `getNumberPropertyHistory(thing, property, start, end)` | NUMBER | Historique valeurs |
| `getStringPropertyHistory(thing, property, start, end)` | STRING | Historique valeurs |

### Type 3 : Service Graph (génération ECharts)

**Pattern :** Appelle un service Data, puis transforme en options ECharts.

```javascript
// 1. Récupérer les données brutes
let data = me.getStopsData({thingName: thingName, startDate: startDate, endDate: endDate});

// 2. Transformer en séries ECharts
let categories = [];
let durations = [];
let colors = [];

data.rows.toArray().forEach(row => {
    categories.push(row.cause_name);
    durations.push(row.duration_seconds);
    colors.push(row.cause_color);
});

// 3. Construire les options ECharts
result = {
    tooltip: {
        trigger: 'axis',
        axisPointer: { type: 'shadow' },
        valueFormatter: "secondString"  // Raccourci durée → "Xd HH:MM:SS"
    },
    grid: {
        left: '3%', right: '4%', bottom: '3%', top: '3%',
        containLabel: true
    },
    yAxis: [{
        type: 'category',
        data: categories
    }],
    xAxis: [{
        type: 'value'
    }],
    series: [{
        name: 'Durée',
        type: 'bar',
        data: durations,
        itemStyle: { color: function(params) { return colors[params.dataIndex]; } }
    }]
};
```

## Processus complet

### Étape 1 : Identifier l'indicateur souhaité

Demander à l'utilisateur :
1. **Quel type de données ?** (arrêts PostgreSQL / propriété InfluxDB / combiné)
2. **Quel type de visualisation ?** (bar, line, pie/donut, gauge, pareto, timeline)
3. **Quels paramètres ?** (thingName, property, startDate, endDate)

### Étape 2 : Lire les patterns documentés et les services existants

**Référence documentée :** `references/service-patterns-iiot.md` contient le code JavaScript réel des requêtes SQL (pareto, arrêts groupBy, TRS), le calcul TRS norme NF E60-182, et les patterns de graphiques ECharts (gauge, bar, etc.)

```bash
# Si besoin de plus de détails, lire les sources complètes :
cat 08-SourceControl/DataBox/ThingShapes/DataBox_TS_Devices.xml
cat 08-SourceControl/DataBox/ThingShapes/DataBox_TS_TRS.xml
cat 08-SourceControl/DataBox/ThingShapes/DataBox_TS_Dash_Comptes.xml
```

### Étape 3 : Créer le service Data (collecte)

Ajouter au ThingShape cible (DataBox_TS_Devices ou le ThingShape du device) :
- ServiceDefinition avec paramètres (thingName, startDate, endDate, property si InfluxDB)
- ServiceImplementation avec la requête SQL ou l'appel InfluxDB

### Étape 4 : Créer le service Graph (visualisation)

Ajouter au même ThingShape :
- ServiceDefinition avec mêmes paramètres
- ServiceImplementation qui :
  1. Appelle le service Data
  2. Transforme les données en options ECharts
  3. Retourne le JSON résultat

### Étape 5 : Checklist de validation

**Service Data :**
- [ ] Les paramètres sont typés correctement (thingName: STRING, startDate/endDate: DATETIME, property: STRING)
- [ ] Les requêtes SQL utilisent des templates littéraux avec `${variable.toISOString()}` pour les dates
- [ ] Les requêtes SQL filtrent par `activity_division = '${thingName}'` pour les arrêts
- [ ] Le type de retour est cohérent (INFOTABLE pour runQuery, JSON pour les appels InfluxDB)
- [ ] Les requêtes SQL utilisent le schéma `DATABOX.` (majuscule)

**Service Graph :**
- [ ] Appelle le service Data (pas de requête directe)
- [ ] Produit un JSON ECharts valide avec au minimum `series` + axes
- [ ] Les raccourcis de formatage durée sont utilisés pour les valeurs en secondes (`"secondString"`)
- [ ] Les couleurs des causes sont extraites de `cause_color` quand disponible
- [ ] Le tooltip est configuré avec `trigger: 'axis'` ou `trigger: 'item'` selon le type

**XML :**
- [ ] ServiceDefinition et ServiceImplementation ont le même `name`
- [ ] Le code JavaScript est dans `<![CDATA[ ... ]]>`
- [ ] Les paramètres correspondent entre Definition et Implementation

## Documentation ECharts

Pour des types de graphiques non couverts ci-dessous, utiliser Context7 :
```
mcp__plugin_context7_context7__query-docs(libraryId: "/apache/echarts-doc", query: "...")
```

Exemples de requêtes utiles :
- `"gauge chart with color ranges"` → jauges TRS/OEE
- `"horizontal bar chart sorted"` → pareto
- `"pie donut chart"` → répartition arrêts
- `"line chart time series with dataZoom"` → historique propriétés
- `"scatter plot with custom symbols"` → corrélations
- `"heatmap with calendar"` → calendrier de production

## Patterns ECharts courants

### Bar chart horizontal (pareto arrêts)
```javascript
result = {
    yAxis: [{type: 'category', data: categories}],
    xAxis: [{type: 'value'}],
    series: [{type: 'bar', data: values}]
};
```

### Donut (répartition)
```javascript
result = {
    series: [{
        type: 'pie',
        radius: ['40%', '70%'],
        data: items.map(i => ({name: i.name, value: i.value, itemStyle: {color: i.color}}))
    }]
};
```

### Gauge (TRS/OEE)
```javascript
result = {
    series: [{
        type: 'gauge',
        min: 0, max: 100,
        data: [{value: trsValue, name: 'TRS'}],
        detail: {formatter: '{value}%'}
    }]
};
```

### Line chart temporel (historique propriété)
```javascript
result = {
    xAxis: [{type: 'time'}],
    yAxis: [{type: 'value'}],
    series: [{
        type: 'line',
        data: history.map(h => [h.timestamp, h.value]),
        smooth: true
    }],
    dataZoom: [{type: 'inside'}, {type: 'slider'}]
};
```
