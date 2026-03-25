# Claude Code Plugin Marketplace

3 plugins Claude Code pour la planification agile, l'interrogation de données ERP/CRM et le développement ThingWorx.

## Installation

```bash
/plugin marketplace add Clint65/cyril-mp
```

## Plugins

### agile-planning

Planification agile avec structure Epic/Feature/User Story, optimisée pour le développement solo assisté par Claude.

```bash
/plugin install agile-planning@cyril-mp
```

| Skill | Commande | Description |
|-------|----------|-------------|
| create-plans-agile | `/agile-planning:create-plans-agile [projet]` | Planification agile complète (brief, roadmap, epics, features, stories) |
| plan-us | `/agile-planning:plan-us <story-path>` | Creer un plan d'execution (US-PLAN.md) pour une user story |
| run-us | `/agile-planning:run-us <plan-path>` | Executer un plan US-PLAN.md |
| test-us | `/agile-planning:test-us <story-path>` | Tester une user story completee |
| commit-us | `/agile-planning:commit-us <story-path>` | Commiter une user story avec format git conventionnel |
| qa-report | `/agile-planning:qa-report [scope]` | Generer un rapport QA sur la couverture des stories |

### databox-query

Interrogation NL->SQL de la base PostgreSQL Databox (ERP/CRM) en langage naturel francais.

```bash
/plugin install databox-query@cyril-mp
```

| Composant | Commande | Description |
|-----------|----------|-------------|
| Skill NL->SQL | *(auto-invoque)* | Traduit les questions business en SQL et retourne les resultats |
| Commande schema | `/databox-query:databox-schema [domaine]` | Affiche le schema Databox (tables, relations, conventions) |
| MCP PostgreSQL | *(integre)* | Serveur MCP PostgreSQL configure via `$DATABOX_DATABASE_URL` |

**Prerequis** : definir la variable d'environnement `DATABOX_DATABASE_URL` :

```bash
export DATABOX_DATABASE_URL="postgresql://user:password@host:5432/database"
```

### twx-databox

Developpement ThingWorx assiste pour le projet DataBox. Genere des services (ThingShapes XML), dashboards (GridStack) et tiles a partir des patterns existants.

```bash
/plugin install twx-databox@cyril-mp
```

## License

MIT
