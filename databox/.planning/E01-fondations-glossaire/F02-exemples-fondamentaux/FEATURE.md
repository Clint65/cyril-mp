# Feature: Exemples fondamentaux NL→SQL

**Feature ID:** F02 (within E01)
**Epic:** E01 - Fondations & Glossaire métier
**Status:** Complete

## Description

Créer les premiers exemples few-shot NL→SQL qui couvrent les patterns de base Databox : version courante, résolution idcor, jointure article, agrégation temporelle. Documenter les pièges connus avec des exemples de requêtes fausses vs correctes.

## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-003 | Exemples NL→SQL de base | 3 | Complete | 2026-03-19 |
| US-004 | Documentation des pièges connus | 2 | Complete | 2026-03-19 |

**Total:** 2 stories, 5 points

## Acceptance Criteria (Feature Level)

- [ ] Au moins 6 exemples NL→SQL couvrant les patterns fondamentaux
- [ ] Chaque piège critique est documenté avec une requête fausse et sa correction
- [ ] Les exemples sont validés par exécution sur la base

## Dependencies

- **Requires:** F01 (glossaire métier pour les termes utilisés dans les exemples)
- **Enables:** E02, E03 (les exemples fondamentaux servent de base aux exemples par domaine)
