# US-018: Exemples few-shot complets Summary

**31 exemples NL→SQL validés couvrant tous les domaines et tous les patterns critiques.**

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | Couverture domaines | Done | Ventes 10, Facturation 5, Achats 5, Avancés 5, Fondamentaux 6 |
| AC2 | Couverture patterns SQL | Done | 7 patterns couverts (end_date, idcor, cast TEXT, generate_series, invoices_sign, value_lists, multi-devises) |
| AC3 | Validation par exécution | Done | Toutes requêtes exécutées (sauf tables vides : syntaxe validée) |
| AC4 | Organisation par fichier | Done | 5 fichiers dans references/exemples/ |

## Bilan

| Fichier | Nb exemples | Domaine |
|---------|-------------|---------|
| requetes_fondamentales.md | 6 | Patterns de base |
| requetes_ventes.md | 10 | Devis, commandes, clients |
| requetes_facturation.md | 5 | Factures, encours, articles |
| requetes_achats.md | 5 | Fournisseurs, OF, projets |
| requetes_avancees.md | 5 | Multi-devises, value_lists, risque |
| **Total** | **31** | **Tous domaines** |

---
*Story: US-018 / Completed: 2026-03-19*
