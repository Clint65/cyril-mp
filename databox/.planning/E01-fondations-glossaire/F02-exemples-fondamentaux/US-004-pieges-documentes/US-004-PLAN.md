---
story: US-004
feature: F02
epic: E01
type: execute
---

<objective>
Documentation des pièges connus

**As a** skill NL→SQL
**I want** connaître les pièges spécifiques du schéma Databox avec des exemples concrets
**So that** je ne génère jamais de SQL silencieusement faux

Purpose: Documenter les erreurs silencieuses pour que le skill les évite systématiquement
Output: references/exemples/pieges_connus.md avec 6 pièges documentés (requête fausse vs correcte)
</objective>

<execution_context>
Execute this plan with: /run-us .planning/E01-fondations-glossaire/F02-exemples-fondamentaux/US-004-pieges-documentes/US-004-PLAN.md
</execution_context>

<context>
@.planning/BRIEF.md
@.planning/E01-fondations-glossaire/F02-exemples-fondamentaux/FEATURE.md
@.planning/E01-fondations-glossaire/F02-exemples-fondamentaux/US-004-pieges-documentes/US-004.md
@references/metier/glossaire.md
@DB-SCHEMA.md
</context>

<acceptance_criteria>
AC1: Piège end_date — requête fausse (résultats multipliés), correcte, impact chiffré
AC2: Piège aliases id_mapping — erreur SQL et correction
AC3: Piège cast TEXT — jointure vide et correction
AC4: Piège invoices_sign — CA gonflé vs correct
AC5: Les 6 pièges critiques couverts avec avant/après
</acceptance_criteria>

<tasks>
<task id="1" type="auto" maps-to="AC1,AC2,AC3,AC4,AC5">
Construire et exécuter les 6 pièges (faux vs correct) sur la base, puis écrire pieges_connus.md.
</task>
</tasks>

<output>
Create US-004-SUMMARY.md in this directory.
</output>
