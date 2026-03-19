# Feature: Cas avancés

**Feature ID:** F07 (within E04)
**Epic:** E04 - Robustesse & Polish
**Status:** Complete

## Description

Gérer les cas complexes qui rendent les résultats faux s'ils sont ignorés : multi-devises (taux de change), décodage des colonnes codées via le système de listes de valeurs, et détection des clients à risque (inactifs, en baisse, en retard de paiement).

## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-015 | Multi-devises et taux de change | 3 | Complete | 2026-03-19 |
| US-016 | Décodage listes de valeurs | 3 | Complete | 2026-03-19 |
| US-017 | Clients à risque et alertes | 5 | Complete | 2026-03-19 |

**Total:** 3 stories, 11 points

## Acceptance Criteria (Feature Level)

- [ ] Les montants multi-devises sont ramenés en devise de référence via currency_rate
- [ ] Les statuts, types et catégories codés sont décodés en libellés français
- [ ] Les clients inactifs, en baisse de CA ou en retard sont identifiables
- [ ] Exemples few-shot pour chaque cas avancé

## Dependencies

- **Requires:** E02/F03-F04 (ventes/facturation), E03/F05 (achats pour multi-devises)
- **Enables:** F08 (validation finale)
