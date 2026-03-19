# Epic: Achats, Production & Projets

**Epic ID:** E03
**Status:** Complete
**Created:** 2026-03-19

## Value Proposition

**As a** responsable achats, production ou chef de projet
**I want** interroger les données fournisseurs, fabrication et projets en langage naturel
**So that** j'aie une vue complète de mon activité au-delà du cycle commercial

## Success Metrics

- [ ] Les questions sur les achats fournisseurs (volumes, montants, écarts) sont couvertes
- [ ] Le suivi des ordres de fabrication est fonctionnel
- [ ] Les projets/affaires avec leurs tâches sont interrogeables

## Features

| ID | Feature | Stories | Points | Status |
|----|---------|---------|--------|--------|
| F05 | Analyse achats | 2 | 5 | Not started |
| F06 | Production & projets | 2 | 5 | Not started |

**Total:** 2 features, 4 stories, 10 points

## Dependencies

- **Requires:** E01 (glossaire métier)
- **Enables:** E04 (cas avancés cross-domaines)

## Out of Scope

- Nomenclatures détaillées (bom_commercial, bom_subcontracting) — trop spécialisé pour un premier niveau
- Qualité (quality_events) — à évaluer selon le besoin

## Technical Notes

- Tables : purchase_orders, receipts, purchase_invoices, manufacturing_orders, projects
- Relations fournisseurs via idcor_supplier (même pattern que idcor_account)
