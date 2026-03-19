# US-011: Achats par fournisseur Summary

**Exemples NL→SQL pour top fournisseurs et achats (syntaxe validée, purchase_orders vide).**

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | Top fournisseurs par montant | Done | Syntaxe validée, noms colonnes corrigés |
| AC2 | Évolution achats par période | Done | Pattern generate_series documenté |
| AC3 | Achats par article | Done | Pattern idcor_product documenté |
| AC4 | Factures fournisseurs | Done | Pattern idcor_supplier documenté |

## Deviations from Plan

- Noms de colonnes différents de DB-SCHEMA.md : `purchase_orders_code`, `purchase_orders_total_amount_excl_vat`, `suppliers_name`
- Table `purchase_orders` vide

## Files Created/Modified

- `references/exemples/requetes_achats.md` - Exemples 1-2 (top fournisseurs, écarts)

---
*Story: US-011 / Completed: 2026-03-19*
