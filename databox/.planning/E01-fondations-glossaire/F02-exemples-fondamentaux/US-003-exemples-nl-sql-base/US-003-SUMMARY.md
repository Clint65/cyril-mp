# US-003: Exemples NL→SQL de base Summary

**6 paires NL→SQL couvrant les 6 patterns fondamentaux Databox, toutes validées par exécution.**

## Story Completed

**As a** skill NL→SQL
**I want** disposer d'exemples few-shot couvrant les patterns fondamentaux Databox
**So that** je puisse m'appuyer sur des modèles validés pour générer du SQL correct

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | Exemple version courante (end_date IS NULL) | Done | Exécution : 5 clients actifs retournés |
| AC2 | Exemple résolution idcor (alias acc_im) | Done | Exécution : 14 factures VELOLAND CHOLET |
| AC3 | Exemple jointure article (cast TEXT) | Done | Exécution : top 5 articles par quantité |
| AC4 | Exemple agrégation temporelle (generate_series) | Done | Exécution : 12 mois dont mois à 0 |
| AC5 | Exemple invoices_sign dans SUM | Done | Exécution : CA brut 17 730 vs net 14 091 |
| AC6 | 6+ exemples validés par exécution | Done | 6 exemples, 0 erreur SQL |

## Accomplishments

- 6 exemples NL→SQL couvrant chaque pattern critique : version courante, résolution idcor, cast TEXT, generate_series, invoices_sign, aliases multiples
- Chaque exemple inclut : question en français, SQL complet, points clés, extrait de résultat
- Tableau récapitulatif des patterns en en-tête du fichier
- Résultats réels capturés pour illustration

## Files Created/Modified

- `references/exemples/requetes_fondamentales.md` - 6 exemples NL→SQL fondamentaux

## Deviations from Plan

None

---
*Story: US-003*
*Completed: 2026-03-19*
