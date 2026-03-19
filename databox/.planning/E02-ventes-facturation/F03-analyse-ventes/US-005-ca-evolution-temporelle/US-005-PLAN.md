---
story: US-005
feature: F03
epic: E02
type: execute
---

<objective>
CA et évolution temporelle

**As a** dirigeant ou responsable commercial
**I want** demander l'évolution de mon chiffre d'affaires sur une période en langage naturel
**So that** je puisse suivre la tendance de mon activité sans écrire de SQL

Purpose: Couvrir les questions d'évolution CA (par mois, trimestre, comparaison N/N-1, par commercial)
Output: Exemples NL→SQL dans references/exemples/requetes_ventes.md
</objective>

<context>
@references/metier/glossaire.md
@references/metier/kpis.md
@DB-SCHEMA.md
</context>

<acceptance_criteria>
AC1: CA par mois avec generate_series (mois à zéro inclus)
AC2: CA par trimestre avec DATE_TRUNC('quarter')
AC3: Comparaison N/N-1 avec pourcentage d'évolution
AC4: CA par commercial avec résolution idcor_sales_rep
AC5: Au moins 3 paires NL→SQL dans requetes_ventes.md
</acceptance_criteria>

<tasks>
<task id="1" type="auto" maps-to="AC1,AC2,AC3,AC4,AC5">
Construire, exécuter et documenter les requêtes d'évolution CA puis écrire requetes_ventes.md.
</task>
</tasks>

<output>Create US-005-SUMMARY.md in this directory.</output>
