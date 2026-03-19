# US-017: Clients à risque et alertes Summary

**2 exemples NL→SQL pour détection clients en baisse et score de risque combiné.**

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | Clients inactifs | Done | Validé US-007 ex.9 (clients sans commande) |
| AC2 | Clients en baisse CA | Done | Exécution : XNS Suisse -97,76%, AERONEF -52,52% |
| AC3 | Clients en retard paiement | Done | Pattern documenté (open_items vide) |
| AC4 | Score combiné | Done | Requête CTE avec 2 signaux (extensible à 3) |

## Files Created/Modified

- `references/exemples/requetes_avancees.md` - Exemples 4-5

---
*Story: US-017 / Completed: 2026-03-19*
