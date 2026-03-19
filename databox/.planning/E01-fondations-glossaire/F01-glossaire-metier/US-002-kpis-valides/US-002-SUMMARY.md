# US-002: KPIs validés avec formules SQL Summary

**7 KPIs avec formules SQL complètes, validés par exécution sur la base Databox.**

## Story Completed

**As a** utilisateur métier
**I want** que chaque KPI ait une formule SQL Databox précise et validée
**So that** les calculs soient reproductibles et cohérents d'une question à l'autre

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | Évolution CA avec generate_series, granularité, périodes vides | Done | Exécution sur base (mois + trimestre) |
| AC2 | Taux de conversion avec ventilation par commercial | Done | Exécution avec résolution idcor_sales_rep |
| AC3 | Marge absolue et %, traitement avoirs | Done | Exécution : 9 403,28 / 19,40% |
| AC4 | Fichier kpis.md structuré | Done | 7 KPIs dans references/metier/kpis.md |

## Accomplishments

- 7 KPIs documentés : Évolution CA facturé, Évolution CA commandé, Taux de conversion, Marge brute, Panier moyen, Top N clients, Délai moyen de paiement
- Chaque KPI inclut : description, source glossaire, paramètres, formule SQL complète, exemple de résultat, règles Databox appliquées
- 6/7 KPIs validés par exécution réelle sur la base (délai de paiement non testable car open_items vide)
- Exemples de résultats réels capturés (CA 2024, marge 19,40%, panier moyen 443,01€, taux conversion 48,15%)

## Decisions Made

- Nom de colonne corrigé : `accounts_company_name_1` (avec underscore, pas `accounts_company_name1` comme dans DB-SCHEMA.md)
- Jointure open_items→invoices via `document_number = invoice_code` : à vérifier quand les données seront disponibles

## Files Created/Modified

- `references/metier/kpis.md` - 7 KPIs avec formules SQL complètes
- `references/metier/glossaire.md` - Correction `accounts_company_name_1`

## Deviations from Plan

- **Bug fix :** Nom de colonne `accounts_company_name1` → `accounts_company_name_1` corrigé dans le glossaire (DB-SCHEMA.md inexact)
- **Note :** Table `open_items` vide dans le jeu de données — KPI délai de paiement validé syntaxiquement mais pas fonctionnellement

---
*Story: US-002*
*Completed: 2026-03-19*
