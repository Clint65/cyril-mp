# Epic: Robustesse & Polish

**Epic ID:** E04
**Status:** In Progress
**Created:** 2026-03-19

## Value Proposition

**As a** utilisateur métier
**I want** que le skill gère correctement les cas complexes (multi-devises, décodage statuts, alertes clients)
**So that** je puisse faire confiance aux résultats même sur des questions avancées

## Success Metrics

- [ ] Les montants multi-devises sont correctement ramenés en devise de référence
- [ ] Les colonnes codées sont décodées en libellés lisibles via le système de listes de valeurs
- [ ] Les alertes clients (inactifs, baisse CA) sont fonctionnelles
- [ ] 15-20 exemples few-shot validés couvrent l'ensemble des domaines
- [ ] Les tests de non-régression vérifient les 6 règles critiques
- [ ] SKILL.md final créé via /skill-creator, opérationnel sur les 4 questions cibles de la spec

## Features

| ID | Feature | Stories | Points | Status |
|----|---------|---------|--------|--------|
| F07 | Cas avancés | 3 | 11 | Not started |
| F08 | Qualité & complétude | 3 | 10 | Not started |

**Total:** 2 features, 6 stories, 21 points

## Dependencies

- **Requires:** E02, E03 (domaines fonctionnels en place)
- **Enables:** Port vers Flowise (projet séparé)

## Technical Notes

- Multi-devises : colonnes currency_rate dans quotes, sales_orders, invoices
- Listes de valeurs : 5 tables (value_list, value_list_entry, value_list_label, value_list_system_mapping, value_list_column_config)
- Clients à risque : croisement open_items, customer_due_dates, historique commandes/factures
