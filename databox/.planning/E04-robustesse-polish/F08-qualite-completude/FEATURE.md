# Feature: Qualité & complétude

**Feature ID:** F08 (within E04)
**Epic:** E04 - Robustesse & Polish
**Status:** In Progress

## Description

Compléter les exemples few-shot pour atteindre 15-20 paires NL→SQL validées couvrant tous les domaines. Mettre en place des tests de non-régression vérifiant l'application systématique des 6 règles critiques Databox.

## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-018 | Exemples few-shot complets (15-20 paires) | 3 | Complete | 2026-03-19 |
| US-019 | Validation et tests de non-régression | 2 | Complete | 2026-03-19 |
| US-020 | Créer le SKILL.md final via /skill-creator | 5 | Not started | - |

**Total:** 3 stories, 10 points

## Acceptance Criteria (Feature Level)

- [ ] 15-20 exemples few-shot validés couvrant tous les domaines
- [ ] Tests vérifiant les 6 règles critiques (end_date, idcor, aliases, cast TEXT, invoices_sign, value_lists)
- [ ] Aucune régression sur les exemples existants
- [ ] SKILL.md final créé via /skill-creator et opérationnel sur les 4 questions cibles

## Dependencies

- **Requires:** F07 (cas avancés), E02, E03 (domaines)
- **Enables:** Port vers Flowise (projet séparé)
