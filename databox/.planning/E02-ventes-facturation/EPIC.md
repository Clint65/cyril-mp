# Epic: Ventes & Facturation

**Epic ID:** E02
**Status:** Complete
**Created:** 2026-03-19

## Value Proposition

**As a** responsable commercial ou dirigeant
**I want** poser des questions sur mon activité commerciale en langage naturel
**So that** j'obtienne instantanément des analyses fiables sur mon CA, mes conversions et mes clients

## Success Metrics

- [ ] Les questions d'évolution de CA (par mois, trimestre, année) produisent des résultats corrects
- [ ] Le taux de conversion devis→commandes est calculé selon la définition métier validée
- [ ] Les top clients, l'analyse par article et les encours sont couverts
- [ ] invoices_sign est systématiquement appliqué dans les agrégations de facturation

## Features

| ID | Feature | Stories | Points | Status |
|----|---------|---------|--------|--------|
| F03 | Analyse ventes | 3 | 11 | Not started |
| F04 | Facturation & encours | 3 | 11 | Not started |

**Total:** 2 features, 6 stories, 22 points

## Dependencies

- **Requires:** E01 (glossaire métier validé)
- **Enables:** E04 (robustesse et cas avancés)

## Out of Scope

- Multi-devises avancé (E04)
- Analyse de clients à risque (E04)

## Technical Notes

- Tables principales : quotes, sales_orders, invoices, deliveries + leurs lignes
- Relations clés : idcor_account, idcor_quote, idcor_sales_rep
- Attention : invoices_sign pour les avoirs, quotes_quote_status pour la conversion
