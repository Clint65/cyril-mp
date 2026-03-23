# Patterns de Services IIOT

Sources : `DataBox_TS_Devices.xml`, `DataBox_TS_TRS.xml`, `DataBox_Helper_Devices.xml`

## Table des matières

1. [Requêtes PostgreSQL](#1-requêtes-postgresql-arrêts-activités) (arrêts, activités)
2. [Modèle de données IIOT](#modèle-de-données-iiot) (tables, causes, schéma ER)
3. [Calcul TRS](#2-calcul-trs-norme-nf-e60-182) (norme NF E60-182)
4. [Graphique Gauge TRS](#3-graphique-gauge-trs-code-réel)
5. [Conversion durée](#4-conversion-durée)
6. [Simulation de données](#5-simulation-de-données-testdémo)

---

## Modèle de données IIOT

### Schéma ER (tables PostgreSQL dans le schéma DATABOX)

```
DATABOX.CAUSE                          DATABOX.ACTIVITY
┌─────────────────────────┐            ┌──────────────────────────────┐
│ cause_id (PK, text)     │◄───────────│ activity_cause_id (FK, text) │
│ cause_name (text)       │            │ activity_id (PK, BIGSERIAL)  │
│ cause_token_name (text) │            │ activity_type (text)         │
│ cause_color (text)      │            │ activity_start (TIMESTAMPTZ) │
│ cause_parent_id (FK→self)│           │ activity_end (TIMESTAMPTZ)   │
│ cause_tree_type (enum)  │            │ activity_division (text)     │
│ cause_visible (bool)    │            │ activity_comment (text)      │
│ cause_root_cause_id (text)│          │ activity_of (text)           │
│ cause_root_cause_name    │           │ activity_data_custom (JSONB) │
│ cause_data_custom (JSONB)│           └──────────────────────────────┘
└─────────────────────────┘
     │ cause_parent_id
     └──→ (self-reference = arbre hiérarchique)
```

### Valeurs de cause_root_cause_id

L'arbre des causes est hiérarchique. Chaque cause feuille appartient à une catégorie racine :

| root_cause_id | Signification | Impact sur TRS |
|---------------|---------------|----------------|
| `running` | Fonctionnement normal (production) | Temps de fonctionnement |
| `closing` | Fermeture (site fermé, week-end, nuit) | Réduit le temps d'ouverture |
| `scheduled` | Arrêt planifié (maintenance, nettoyage, affûtage) | Réduit le temps requis |
| `unscheduled` | Arrêt non planifié (panne, manque matière, absence) | Réduit la disponibilité opérationnelle |

### Valeurs de cause_tree_type (enum)

| Valeur | Description |
|--------|-------------|
| `fcad_ROOT` | Racine de l'arbre |
| `fcad_BRANCH` | Branche intermédiaire (catégorie) |
| `fcad_LEAF` | Feuille (cause sélectionnable) |

### Valeurs de activity_type

| Valeur | Description |
|--------|-------------|
| `REEL` | Activité réelle (production, arrêt constaté) |
| `PLANIFIE` | Activité planifiée (calendrier de production) |

### Clé de jointure

- `activity_division` = `ThingName` du device/workstation concerné
- Les requêtes filtrent toujours par `activity_division = '${thingName}'`

---

## 1. Requêtes PostgreSQL (arrêts, activités)

### Pattern : runQuery (SELECT)

```javascript
let query = `
    SELECT ...
    FROM DATABOX.ACTIVITY a
    LEFT JOIN DATABOX.CAUSE c ON a.activity_cause_id = c.cause_id
    WHERE a.activity_division = '${thingName}'
    AND a.activity_start >= TIMESTAMP '${startDate.toISOString()}'
    AND a.activity_end <= TIMESTAMP '${endDate.toISOString()}'
`;
let result = Things["DataBox_DB"].runQuery({query: query});
```

### Pattern : runCommand (INSERT/UPDATE)

```javascript
// INSERT
let sql = `INSERT INTO Databox.activity (activity_type, activity_start, activity_cause_id, activity_division, activity_comment)
    VALUES ('REEL', TO_TIMESTAMP(${startDate.getTime()/1000}), '${causeId}', '${division}', '')
    RETURNING activity_id`;
let result = Things["DataBox_DB"].runQuery({query: sql}).activity_id;

// UPDATE
let sql = `UPDATE Databox.activity SET activity_end = TO_TIMESTAMP(${stopDate.getTime()/1000})
    WHERE activity_id=${activityId}`;
Things["DataBox_DB"].runCommand({query: sql});
```

### Requête Pareto par occurrence (code réel)

```javascript
let query = `
    WITH base AS (
      SELECT c.cause_id, c.cause_name, c.cause_color, c.cause_root_cause_id, c.cause_root_cause_name,
             count(a.activity_id) AS occurence
      FROM DATABOX.ACTIVITY a
      JOIN DATABOX.CAUSE c ON a.activity_cause_id = c.cause_id
      WHERE a.activity_type='REEL' AND a.activity_division='${thingName}'
        AND a.activity_start >= TIMESTAMP '${startDate.toISOString()}'
        AND a.activity_end <= TIMESTAMP '${endDate.toISOString()}'
      GROUP BY c.cause_id, c.cause_name
    ),
    ordered AS (
      SELECT *, SUM(occurence) OVER () AS total_all,
             SUM(occurence) OVER (ORDER BY occurence DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_count
      FROM base
    )
    SELECT cause_id, cause_name, cause_color, cause_root_cause_id, cause_root_cause_name, occurence,
           ROUND((occurence / total_all) * 100, 2) AS percent,
           ROUND((cumulative_count / total_all) * 100, 2) AS cumulative_percent
    FROM ordered ORDER BY occurence DESC;
`;
let result = Things["DataBox_DB"].runQuery({query: query});
```

### Requête arrêts par catégorie avec groupBy (code réel)

```javascript
let query = `
    WITH time_slots AS (
      SELECT gs AS slot_start, gs + INTERVAL '1 ${groupBy}' AS slot_end
      FROM generate_series(
        date_trunc('${groupBy}', TIMESTAMP '${startDate.toISOString()}'),
        date_trunc('${groupBy}', TIMESTAMP '${endDate.toISOString()}') - INTERVAL '1 ${groupBy}',
        INTERVAL '1 ${groupBy}'
      ) AS gs
    ),
    activity_slices AS (
      SELECT ts.slot_start, ts.slot_end, c.cause_root_cause_id, a.activity_id,
             GREATEST(a.activity_start, ts.slot_start) AS overlap_start,
             LEAST(a.activity_end, ts.slot_end) AS overlap_end
      FROM time_slots ts
      JOIN DATABOX.ACTIVITY a ON a.activity_end > ts.slot_start AND a.activity_start < ts.slot_end
      JOIN DATABOX.CAUSE c ON a.activity_cause_id = c.cause_id
      WHERE a.activity_type = 'REEL' AND a.activity_division = '${thingName}'
        AND c.cause_root_cause_id IN ('closing','scheduled','unscheduled')
    )
    SELECT slot_start, slot_end,
      SUM(CASE WHEN cause_root_cause_id = 'closing' THEN EXTRACT(EPOCH FROM (overlap_end - overlap_start)) ELSE 0 END) AS closing,
      SUM(CASE WHEN cause_root_cause_id = 'scheduled' THEN EXTRACT(EPOCH FROM (overlap_end - overlap_start)) ELSE 0 END) AS scheduled,
      SUM(CASE WHEN cause_root_cause_id = 'unscheduled' THEN EXTRACT(EPOCH FROM (overlap_end - overlap_start)) ELSE 0 END) AS unscheduled
    FROM activity_slices WHERE overlap_end > overlap_start
    GROUP BY slot_start, slot_end ORDER BY slot_start;
`;
let result = Things["DataBox_DB"].runQuery({query: query});
```

## 2. Calcul TRS (norme NF E60-182)

### Architecture du calcul

```
calculcateTRSIndicators(thingName, startDate, endDate, groupBy)
    → getDowntimesByCategory() [SQL] → closing, scheduled, unscheduled (en secondes)
    → computeTRS(downTimesByType) → indicateurs TRS
```

### Indicateurs calculés par computeTRS

| Indicateur | Formule | Description |
|-----------|---------|-------------|
| tempsTotal | endDate - startDate | Temps total de la période |
| tempsFermeture | closing | Temps de fermeture |
| tempsOuverture | tempsTotal - tempsFermeture | Temps d'ouverture |
| tempsRequis | tempsOuverture - arretsPlanifies | Temps requis |
| tempsFonctionnement | tempsRequis - arretsNonPlanifies | Temps de fonctionnement |
| tempsNet | tempsFonctionnement - ecartsCadence | Temps net (cadence = 0 par défaut) |
| tempsUtile | tempsNet - tempsNonQualite | Temps utile (qualité = 0 par défaut) |
| Disponibilité | tempsFonctionnement / tempsRequis × 100 | % |
| Performance | tempsNet / tempsFonctionnement × 100 | % |
| Qualité | tempsUtile / tempsNet × 100 | % |
| TRS | tempsUtile / tempsRequis × 100 | Taux de Rendement Synthétique |
| TRG | tempsUtile / tempsOuverture × 100 | Taux de Rendement Global |
| TRE | tempsUtile / tempsTotal × 100 | Taux de Rendement Économique |

### Format de retour computeTRS

```javascript
result = {
    thingName: thingName,
    startDate: ..., endDate: ...,
    indicators: {
        tempsTotal: {value: 86400, unit: "secondes", valueString: "1j 00h00m00s", description: "..."},
        disponibiliteOperationnelle: {value: 85.5, unit: "%", description: "..."},
        TRS: {value: 72.3, unit: "%", description: "..."},
        // ... tous les indicateurs
    }
};
```

## 3. Graphique Gauge TRS (code réel)

```javascript
let trs = me.getTRSOverviewGraph({thingName: thingName, startDate: startDate, endDate: endDate}).TRS.value;
result = {
    series: [{
        type: "gauge",
        radius: '80%',
        axisLine: {
            lineStyle: {
                width: 10,
                color: [
                    [0.3, "#f44336"],   // rouge < 30%
                    [0.7, "#2196f3"],   // bleu 30-70%
                    [1, "#4caf50"],     // vert > 70%
                ]
            }
        },
        pointer: {
            icon: "path://M2.9,0.7L2.9,0.7c1.4,0,2.6,1.2,2.6,2.6v115c0,1.4-1.2,2.6-2.6,2.6l0,0c-1.4,0-2.6-1.2-2.6-2.6V3.3C0.3,1.9,1.4,0.7,2.9,0.7z",
            width: 4,
            itemStyle: {color: "auto"}
        },
        anchor: {show: true, showAbove: true, size: 10, itemStyle: {color: "auto"}},
        axisTick: "none",
        splitLine: {distance: -10, length: 15, lineStyle: {color: "#fff", width: 2}},
        axisLabel: "none",
        detail: {valueAnimation: true, formatter: "{value} %", color: "inherit", fontSize: '0.7rem', offsetCenter: [0, '85%']},
        data: [{value: trs}]
    }]
};
```

## 4. Conversion durée

```javascript
function secondsToDhms(value) {
    if (value && value > 0) {
        var d = Math.floor(value / 86400);
        var h = Math.floor(value % 86400 / 3600);
        var m = Math.floor(value % 3600 / 60);
        var s = Math.floor(value % 3600 % 60);
        var dDisplay = d > 0 ? d + "j " : "";
        var hDisplay = h > 0 ? (h < 10 ? "0" + h : h) + "h" : "";
        var mDisplay = m > 0 ? (m < 10 ? "0" + m : m) + "m" : "";
        var sDisplay = s > 0 ? (s < 10 ? "0" + s : s) + "s" : "";
        return dDisplay + hDisplay + mDisplay + sDisplay;
    } else {
        return "0 Jour";
    }
}
```

## 5. Simulation de données (test/démo)

```javascript
// Service simulattion dans DataBox_Helper_Devices
const randomFactor = (Math.random() - 0.5) * 2;
const volatileChange = randomFactor * volatility;
const change = volatileChange * maxChange;
let newTemp = previous + change;
newTemp = Math.max(min, Math.min(max, newTemp));
Things[thing][property] = Math.round(newTemp * 10) / 10;
```
