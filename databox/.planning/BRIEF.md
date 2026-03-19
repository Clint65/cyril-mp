# Skill NL→SQL Databox

**One-liner**: Skill Claude Code permettant d'interroger la base PostgreSQL Databox (ERP/CRM) en langage naturel français.

## Problem

Les utilisateurs métier veulent obtenir des réponses à des questions business (CA, taux de conversion, clients à risque…) sans connaître SQL ni les particularités du schéma Databox (versioning id_mapping, résolution idcor, invoices_sign, cast TEXT…). Chaque requête mal construite produit des résultats silencieusement faux.

## Success Criteria

- [ ] Le skill comprend une question en français et génère du SQL Databox correct
- [ ] Les 6 règles critiques sont systématiquement appliquées (end_date IS NULL, idcor, aliases, cast TEXT, invoices_sign, value_lists)
- [ ] Le skill couvre les domaines : ventes, achats, facturation, production, projets
- [ ] Les réponses sont formatées en langage naturel avec les données
- [ ] Un glossaire métier validé traduit les termes business en SQL

## Constraints

- PostgreSQL, schéma `databox`, accès lecture seule via MCP (`claude_readonly`)
- Documentation schéma existante : DB-SCHEMA.md, ID-MAPPING.md, VALUE-LISTS-ARCHITECTURE.md
- Définitions métier à co-construire par dialogue itératif avec l'utilisateur

## Out of Scope

- Port vers Flowise (projet séparé, le skill sert de référence)
- Requêtes en écriture (INSERT/UPDATE/DELETE)
- Visualisations graphiques (charts, dashboards)
- Support multilingue (français uniquement)
