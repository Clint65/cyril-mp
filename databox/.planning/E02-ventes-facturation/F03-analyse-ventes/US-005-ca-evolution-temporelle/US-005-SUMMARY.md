# US-005: CA et évolution temporelle Summary

**4 exemples NL→SQL couvrant l'évolution CA par mois, trimestre, comparaison N/N-1, et par commercial.**

## Story Completed

**As a** dirigeant ou responsable commercial
**I want** demander l'évolution de mon chiffre d'affaires sur une période en langage naturel
**So that** je puisse suivre la tendance de mon activité sans écrire de SQL

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | CA par mois avec generate_series | Done | Exécution : 12 mois dont mois à 0 |
| AC2 | CA par trimestre | Done | Exécution : 4 trimestres 2024 |
| AC3 | Comparaison N/N-1 | Done | Exécution : 2023 vs 2024, +530% |
| AC4 | CA par commercial | Done | Exécution avec résolution idcor_sales_rep |
| AC5 | 3+ exemples dans requetes_ventes.md | Done | 4 exemples créés |

## Files Created/Modified

- `references/exemples/requetes_ventes.md` - 4 exemples NL→SQL ventes (CA trimestre, comparaison N/N-1, par commercial, CA commandé par mois)

## Deviations from Plan

None

---
*Story: US-005*
*Completed: 2026-03-19*
