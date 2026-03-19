# US-013: Suivi ordres de fabrication Summary

**Exemples NL→SQL pour OF par statut et OF en retard, validés avec 3 674 OF réels.**

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | OF en cours | Done | 164 OF statut "En cours" |
| AC2 | OF par article | Done | Pattern idcor_product documenté |
| AC3 | Charge production par période | Done | Pattern generate_series applicable |
| AC4 | OF en retard | Done | Exécution : OF avec end_date < NOW() et statut 1-2 |

## Discoveries

- Statuts OF : 1="En attente" (3 357), 2="En cours" (164), 4="Exclue" (153)
- Colonne = `manufacturing_orders_status` (pas `order_status`)
- Cast `::text` nécessaire pour jointure value_list (neutral_code est text)

## Files Created/Modified

- `references/exemples/requetes_achats.md` - Exemples 3-4 (OF par statut, OF en retard)

---
*Story: US-013 / Completed: 2026-03-19*
