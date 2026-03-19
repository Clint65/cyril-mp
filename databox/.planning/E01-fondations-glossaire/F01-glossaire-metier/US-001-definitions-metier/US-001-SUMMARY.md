# US-001: Définitions métier fondamentales Summary

**13 termes métier co-construits par dialogue itératif et traduits en SQL Databox avec exemples.**

## Story Completed

**As a** utilisateur métier interrogeant Databox
**I want** que chaque terme business courant soit traduit en colonnes et filtres SQL précis
**So that** le skill génère du SQL qui reflète fidèlement ce que je mesure dans mon activité

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | Terme "Chiffre d'affaires" défini | Done | CA facturé (invoices * invoices_sign) + CA commandé (sales_orders, status IN 1,2) |
| AC2 | Terme "Client actif" défini | Done | Commande dans les 12 derniers mois, basé sur sales_orders |
| AC3 | Terme "Devis converti" défini | Done | quotes_quote_status IN ('2','3'), variante granulaire par lignes |
| AC4 | Au moins 10 termes validés | Done | 13 termes validés par dialogue itératif |
| AC5 | Glossaire au format structuré | Done | references/metier/glossaire.md créé (tableau + détails + exemples SQL) |

## Accomplishments

- 13 termes métier définis et validés : CA facturé, CA commandé, marge, client actif, client inactif, devis converti, avoir, encours client, taux de conversion, panier moyen, délai de paiement, commande en retard, commercial
- Chaque terme inclut : définition, table(s), colonne(s), filtres SQL, exemple SQL minimal
- Ambiguïtés métier résolues par dialogue (CA facturé vs commandé, seuil 12 mois, statut 0 exclu, open_items vs customer_due_dates)
- Découverte via requête : statuts commandes (0=brouillon/annulé, 1=non soldée, 2=soldée) et libellés value_list

## Decisions Made

- **CA :** deux termes distincts (facturé et commandé) selon le contexte
- **Statut commande 0 :** exclu des calculs (brouillon/annulé, 23 occurrences)
- **Client inactif :** `accounts_is_active = false` surcharge la règle des 12 mois
- **Encours :** table `open_items` uniquement (`customer_due_dates` abandonnée)
- **Panier moyen :** basé sur les commandes
- **Avoir :** `invoices_sign = -1`, pourra être affiné avec `invoices_invoice_type`

## Files Created/Modified

- `references/metier/glossaire.md` - Glossaire complet (13 termes, tableau + détails + exemples SQL)

## Deviations from Plan

- 13 termes au lieu de 12 prévus (ajout du terme "Commercial" pour documenter le pattern de résolution idcor_sales_rep)

---
*Story: US-001*
*Completed: 2026-03-19*
