# Roadmap: Skill NL→SQL Databox

## Overview

Créer un skill Claude Code permettant d'interroger la base PostgreSQL Databox (ERP/CRM) en langage naturel français. Le skill traduit les questions métier en SQL correct (avec toutes les particularités du schéma Databox) et retourne les résultats en langage naturel.

## Epics

- [x] **E01: Fondations & Glossaire métier** - Co-construire les définitions métier et les exemples NL→SQL de base
- [x] **E02: Ventes & Facturation** - Couvrir le cycle commercial complet (devis, commandes, factures, CA, clients)
- [x] **E03: Achats, Production & Projets** - Étendre aux domaines fournisseurs, fabrication et projets
- [ ] **E04: Robustesse & Polish** - Multi-devises, listes de valeurs, clients à risque, tests

## Epic Details

### E01: Fondations & Glossaire métier

**Value:** Base solide sans laquelle aucune requête métier ne peut être fiable
**Depends on:** Rien (premier epic)
**Features:** 2
**Stories:** 4

Features:
- [ ] F01: Glossaire métier (2 stories, 8 pts)
- [ ] F02: Exemples fondamentaux (2 stories, 5 pts)

### E02: Ventes & Facturation

**Value:** Répondre aux questions business les plus fréquentes (CA, conversion, top clients)
**Depends on:** E01
**Features:** 2
**Stories:** 6

Features:
- [ ] F03: Analyse ventes (3 stories, 11 pts)
- [ ] F04: Facturation & encours (3 stories, 11 pts)

### E03: Achats, Production & Projets

**Value:** Couverture complète des domaines ERP secondaires
**Depends on:** E01
**Features:** 2
**Stories:** 4

Features:
- [ ] F05: Analyse achats (2 stories, 5 pts)
- [ ] F06: Production & projets (2 stories, 5 pts)

### E04: Robustesse & Polish

**Value:** Fiabiliser le skill sur les cas complexes et garantir la qualité
**Depends on:** E02, E03
**Features:** 2
**Stories:** 6

Features:
- [ ] F07: Cas avancés (3 stories, 11 pts)
- [ ] F08: Qualité & complétude (3 stories, 10 pts)

## Progress

| Epic | Features | Stories | Points | Status | Completed |
|------|----------|---------|--------|--------|-----------|
| E01: Fondations & Glossaire | 2 | 4/4 | 13/13 | Complete | 2026-03-19 |
| E02: Ventes & Facturation | 2 | 6/6 | 22/22 | Complete | 2026-03-19 |
| E03: Achats, Production & Projets | 2 | 4/4 | 10/10 | Complete | 2026-03-19 |
| E04: Robustesse & Polish | 2 | 5/6 | 16/21 | In progress | - |

**Total:** 4 epics, 8 features, 20 stories, 66 points
