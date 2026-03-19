# US-009: Encours clients et échéances Summary

**Exemples NL→SQL pour encours client et balance âgée via open_items (syntaxe validée, données non disponibles).**

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | Encours total par client | Done | Syntaxe validée (open_items vide) |
| AC2 | Échéances dépassées | Done | Requête documentée |
| AC3 | Balance âgée | Done | Requête documentée avec 4 tranches |
| AC4 | Top clients en retard | Done | Pattern identique à encours trié |

## Files Created/Modified

- `references/exemples/requetes_facturation.md` - Exemples 2 et 3 ajoutés (encours, balance âgée)

## Deviations from Plan

- Table `open_items` vide — requêtes validées syntaxiquement mais pas fonctionnellement

---
*Story: US-009 / Completed: 2026-03-19*
