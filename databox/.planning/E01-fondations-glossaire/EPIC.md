# Epic: Fondations & Glossaire métier

**Epic ID:** E01
**Status:** Complete
**Created:** 2026-03-19

## Value Proposition

**As a** utilisateur métier
**I want** que le skill connaisse les définitions exactes de mes indicateurs (CA, taux de conversion, client actif…)
**So that** les requêtes SQL générées reflètent fidèlement la réalité de mon entreprise

## Success Metrics

- [ ] Un glossaire métier validé traduit chaque terme business en colonnes/filtres SQL Databox
- [ ] Les KPIs sont définis avec précision (formules, filtres, périodes)
- [ ] Au moins 6 exemples NL→SQL fondamentaux couvrent les patterns de base
- [ ] Les pièges spécifiques Databox sont documentés avec exemples de requêtes fausses/correctes

## Features

| ID | Feature | Stories | Points | Status |
|----|---------|---------|--------|--------|
| F01 | Glossaire métier | 2 | 8 | Not started |
| F02 | Exemples fondamentaux | 2 | 5 | Not started |

**Total:** 2 features, 4 stories, 13 points

## Dependencies

- **Requires:** Accès MCP PostgreSQL fonctionnel, documentation schéma (DB-SCHEMA.md, ID-MAPPING.md, VALUE-LISTS-ARCHITECTURE.md)
- **Enables:** E02, E03, E04 — tous les domaines fonctionnels dépendent du glossaire

## Out of Scope

- Implémentation du SKILL.md (sera fait avec /skill-creator)
- Requêtes spécifiques par domaine (E02-E04)

## Technical Notes

- Le glossaire se construit par dialogue itératif : Claude propose des définitions, l'utilisateur valide ou corrige
- Les documents de référence du schéma existent déjà dans le dossier de travail
