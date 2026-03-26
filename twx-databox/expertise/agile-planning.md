# Expertise : ThingWorx DataBox Development

Domain expertise for planning ThingWorx DataBox projects with the agile-planning skill.
This file is loaded by `/create-plans-agile` to enrich story creation with DataBox-specific constraints.

## Skill reference

The implementation skill is `twx-databox` — it handles ThingShapes XML, mashup generation, tiles, services, and API discovery.
When writing US-PLAN.md tasks, the executor will have access to this skill's workflows, references, and templates.

## Two domains

DataBox has two distinct domains with different patterns:

| Domain | Scope | Data sources |
|--------|-------|-------------|
| **DataOps** | ERP/CRM/PLM — Accounts, Quotes, Orders, Articles, etc. | API REST NestJS → PostgreSQL |
| **IIOT** | Sites, Lines, Workstations, Devices, TRS/OEE | API REST NestJS → PostgreSQL + InfluxDB |

Stories should clearly identify which domain they target — patterns and services differ significantly.

## Scope estimation adjustments

| Factor | Impact on estimation |
|--------|---------------------|
| New entity (ThingShape + Dashboard + Tiles) | 5-8 points — full pipeline, consider splitting |
| New ThingShape only (services for existing entity) | 2-3 points |
| New Dashboard (mashup XML) | 2-3 points |
| New Tile (Grid, Synth, or Metric) | 1-2 points per tile |
| New IIOT service (InfluxDB + PostgreSQL) | 2-3 points — dual data source complexity |
| Add services to existing ThingShape | 1-2 points |
| API endpoint discovery | 1 point — research task |
| KPI/ECharts service | 2-3 points — JavaScript logic + ECharts config |
| TRS/OEE calculation | 3-5 points — complex temporal logic |

**Rule of thumb:** A single tile = 1-2 points. A complete entity (ThingShape + Dashboard + 3-4 tiles) = 5-8 points (split into stories).

## Key acceptance criteria patterns

### ThingShape XML validity
```
Given a ThingShape XML is generated
When imported into ThingWorx
Then the XML is valid and the entity is created without errors
And all services are callable from mashups
```

### Service data integrity
```
Given a service calls the NestJS API
When the API returns data
Then the service returns a properly formatted InfoTable
And column names match the DataShape definition
```

### Dashboard layout (GridStack)
```
Given a dashboard mashup is generated
When loaded in ThingWorx at different screen sizes
Then tiles are arranged in a responsive GridStack layout
And each tile renders its data correctly
```

### API coverage
```
Given a new entity is added
When the ThingShape services are implemented
Then all relevant API endpoints for the entity are covered
And the coverage is documented in references/api-twx-mapping.md
```

### IIOT temporal queries
```
Given an IIOT service queries InfluxDB
When a time range is specified
Then the query uses proper InfluxQL/Flux syntax
And results are aggregated at the correct granularity
```

## Story splitting guidance

### For new entities (the most common pattern)
- **Story 1:** API discovery — explore NestJS controllers, document available endpoints
- **Story 2:** ThingShape — create XML with services calling discovered endpoints
- **Story 3:** Dashboard mashup — GridStack layout with tile placeholders
- **Story 4:** Tiles — one story per tile type (Grid, Synth, Metric) or group small tiles

### For IIOT features
- **Story 1:** PostgreSQL services (device metadata, configuration)
- **Story 2:** InfluxDB services (temporal data, aggregations)
- **Story 3:** Dashboard + Tiles (visualization)
- **Story 4:** KPI/TRS calculations (if applicable)

### Do NOT split
- Adding 1-2 services to an existing ThingShape
- Modifying a single tile
- Updating API mappings

## Architecture constraints for plans

When creating US-PLAN.md tasks, the executor must know:

1. **Read before write** — Every task that generates XML MUST first read an existing example (ThingShape, Mashup, or Tile) from `08-SourceControl/DataBox/`. The skill enforces pattern matching with existing code.

2. **XML structure is critical** — ThingWorx XML has strict structure requirements. Generated XML must exactly match the patterns in existing files (entity headers, service definitions, property bindings).

3. **Services are JavaScript** — ThingWorx services are written in JavaScript (not TypeScript), embedded in XML `<ServiceImplementation>` blocks. They use `Things["name"].method()` syntax.

4. **InfoTable return format** — All services returning data must create proper InfoTable objects with DataShape definitions.

5. **Tile naming convention** — `DataBox_Tile_{Entity}_{DataType}_{TileType}` (e.g., `DataBox_Tile_Accounts_Grid`, `DataBox_Tile_DevicePropertyGraph`).

6. **Dashboard naming convention** — `DataBox_Dashboard_{Entity}` for list views, `DataBox_Dashboard_{Entity}Details` for detail views.

7. **ThingShape naming convention** — `DataBox_TS_{Entity}` for DataOps, `DataBox_TS_{Domain}` for IIOT (e.g., `DataBox_TS_Devices`, `DataBox_TS_TRS`).

## Project paths for context in plans

When writing `@file` references in US-PLAN.md:

| Resource | Path |
|----------|------|
| ThingWorx project | `/Users/cyril/Documents/01_Sources/01_Databox/databox-twx-dashboard/` |
| XML source control | `08-SourceControl/DataBox/` (ThingShapes, Mashups, DataShapes) |
| NestJS API | `/Users/cyril/Documents/01_Sources/01_Databox/databox-data-access/` |
| API controllers | `databox-data-access/src/data-databox/*/` |
| API-TWX mapping | `references/api-twx-mapping.md` (in skill) |
