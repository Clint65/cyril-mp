# Feature: Analyse ventes

**Feature ID:** F03 (within E02)
**Epic:** E02 - Ventes & Facturation
**Status:** Complete

## Description

Couvrir les questions d'analyse commerciale : évolution du CA dans le temps, taux de conversion devis→commandes, classement et analyse de portefeuille clients. C'est le coeur de valeur du skill.

## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-005 | CA et évolution temporelle | 5 | Complete | 2026-03-19 |
| US-006 | Taux de conversion devis→commandes | 3 | Complete | 2026-03-19 |
| US-007 | Top clients et analyse portefeuille | 3 | Complete | 2026-03-19 |

**Total:** 3 stories, 11 points

## Acceptance Criteria (Feature Level)

- [ ] Les questions de CA par période (mois, trimestre, année) retournent des résultats corrects
- [ ] Le taux de conversion est calculé selon la définition validée dans le glossaire
- [ ] Les classements clients sont fiables (avec résolution idcor)
- [ ] Exemples few-shot spécifiques au domaine ventes créés

## Dependencies

- **Requires:** E01/F01 (glossaire), E01/F02 (patterns de base)
- **Enables:** F04 (facturation), E04/F07 (cas avancés)
