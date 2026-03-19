# Feature: Glossaire métier

**Feature ID:** F01 (within E01)
**Epic:** E01 - Fondations & Glossaire métier
**Status:** Complete

## Description

Co-construire par dialogue itératif un glossaire qui traduit chaque terme métier (CA, taux de conversion, client actif, marge…) en colonnes, tables et filtres SQL Databox. Définir les KPIs avec leurs formules exactes. Ce glossaire est la source de vérité pour toutes les requêtes générées par le skill.

## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-001 | Définitions métier fondamentales | 5 | Complete | 2026-03-19 |
| US-002 | KPIs validés avec formules SQL | 3 | Complete | 2026-03-19 |

**Total:** 2 stories, 8 points

## Acceptance Criteria (Feature Level)

- [ ] Chaque terme métier courant a une définition validée par l'utilisateur
- [ ] Chaque KPI a une formule SQL précise avec les filtres Databox appropriés
- [ ] Le glossaire est stocké dans references/metier/glossaire.md
- [ ] Les KPIs sont stockés dans references/metier/kpis.md
- [ ] All user stories pass their individual acceptance criteria

## Technical Notes

- Processus itératif : Claude propose, l'utilisateur valide
- Les définitions doivent référencer les colonnes exactes du schéma DB-SCHEMA.md

## Dependencies

- **Requires:** DB-SCHEMA.md, documentation schéma
- **Enables:** F02 (exemples), F03-F08 (tous les domaines)
