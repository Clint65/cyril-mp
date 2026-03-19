# Feature: Facturation & encours

**Feature ID:** F04 (within E02)
**Epic:** E02 - Ventes & Facturation
**Status:** Complete

## Description

Couvrir la facturation (avec gestion des avoirs via invoices_sign), les encours et échéances clients, et l'analyse croisée par article/famille sur le cycle vente complet (devis→commande→facture).

## User Stories

| ID | Story | Points | Status | Completed |
|----|-------|--------|--------|-----------|
| US-008 | CA facturé avec gestion des avoirs | 3 | Complete | 2026-03-19 |
| US-009 | Encours clients et échéances | 5 | Complete | 2026-03-19 |
| US-010 | Analyse par article/famille | 3 | Complete | 2026-03-19 |

**Total:** 3 stories, 11 points

## Acceptance Criteria (Feature Level)

- [ ] Le CA facturé déduit correctement les avoirs (invoices_sign = -1)
- [ ] Les encours et échéances sont exploitables (open_items, customer_due_dates)
- [ ] L'analyse par article fonctionne avec le cast TEXT sur id_product
- [ ] Exemples few-shot spécifiques au domaine facturation créés

## Dependencies

- **Requires:** E01/F01 (glossaire), F03 (analyse ventes pour cohérence)
- **Enables:** E04/F07 (multi-devises, clients à risque)
