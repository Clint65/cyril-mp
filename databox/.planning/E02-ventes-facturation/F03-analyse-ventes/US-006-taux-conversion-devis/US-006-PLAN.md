---
story: US-006
feature: F03
epic: E02
type: execute
---

<objective>
Taux de conversion devis→commandes — global, par commercial, par période, montants convertis/perdus.
Output: Exemples ajoutés dans references/exemples/requetes_ventes.md
</objective>

<acceptance_criteria>
AC1: Taux de conversion global
AC2: Taux par commercial
AC3: Taux par période (mois par mois)
AC4: Montant des devis convertis vs perdus
</acceptance_criteria>

<tasks>
<task id="1" type="auto" maps-to="AC1,AC2,AC3,AC4">
Construire, exécuter et documenter les requêtes de taux de conversion.
</task>
</tasks>

<output>Create US-006-SUMMARY.md in this directory.</output>
