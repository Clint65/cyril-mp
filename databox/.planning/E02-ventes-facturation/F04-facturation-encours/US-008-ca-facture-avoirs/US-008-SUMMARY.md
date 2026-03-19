# US-008: CA facturé avec gestion des avoirs Summary

**Exemples NL→SQL couvrant le détail factures/avoirs par client et l'évolution mensuelle nette.**

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | CA net avec invoices_sign | Done | Validé US-003/US-005 + requetes_facturation.md ex.1 |
| AC2 | Détail factures vs avoirs | Done | Exécution : VELOLAND 6 factures, 4 avoirs |
| AC3 | CA par client avec avoirs déduits | Done | Exécution : top 5 clients avec détail |
| AC4 | Évolution facturation mensuelle | Done | Validé US-005 ex.4 (generate_series + invoices_sign) |

## Files Created/Modified

- `references/exemples/requetes_facturation.md` - Créé avec exemple 1 (détail factures/avoirs)

---
*Story: US-008 / Completed: 2026-03-19*
