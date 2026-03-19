# US-004: Documentation des pièges connus Summary

**6 pièges critiques documentés avec requêtes fausses/correctes et impact chiffré mesuré sur la base.**

## Story Completed

**As a** skill NL→SQL
**I want** connaître les pièges spécifiques du schéma Databox avec des exemples concrets
**So that** je ne génère jamais de SQL silencieusement faux

## Acceptance Criteria Status

| AC | Description | Status | Verified By |
|----|-------------|--------|-------------|
| AC1 | Piège end_date documenté avec impact chiffré | Done | Devis : 659 vs 327 (×2 sans filtre) |
| AC2 | Piège aliases id_mapping documenté | Done | Erreur SQL confirmée par exécution |
| AC3 | Piège cast TEXT documenté | Done | Erreur type confirmée par exécution |
| AC4 | Piège invoices_sign documenté | Done | CA 17 730 vs 14 091 (+25% sans sign) |
| AC5 | 6 pièges critiques couverts | Done | 6 pièges avec avant/après + 2 bonus (idcor, value_lists) |

## Accomplishments

- 6 pièges documentés avec requête fausse, requête correcte et impact chiffré
- Impacts mesurés par exécution réelle : ×2 sur les devis, 82% de jointures perdues, +25% sur le CA
- Découverte : les libellés statuts devis dans value_list diffèrent de DB-SCHEMA.md ('2'="Partiellement commandé", '3'="Totalement commandé")
- Convention d'alias id_mapping documentée (doc_im, acc_im, rep_im, sup_im, art_im)

## Decisions Made

- Statuts devis corrigés dans le glossaire : value_list est la source de vérité (pas DB-SCHEMA.md)

## Files Created/Modified

- `references/exemples/pieges_connus.md` - 6 pièges avec avant/après et impact
- `references/metier/glossaire.md` - Correction statuts devis (value_list = source de vérité)

## Deviations from Plan

- **Bug fix :** Statuts devis dans DB-SCHEMA.md inexacts ('2'=commandé, '3'=partiellement commandé) vs value_list réels ('2'=partiellement commandé, '3'=totalement commandé). Glossaire corrigé.

---
*Story: US-004*
*Completed: 2026-03-19*
