---
story: US-003
feature: F02
epic: E01
type: execute
---

<objective>
Exemples NL→SQL de base

**As a** skill NL→SQL
**I want** disposer d'exemples few-shot couvrant les patterns fondamentaux Databox
**So that** je puisse m'appuyer sur des modèles validés pour générer du SQL correct

Purpose: Constituer la base d'exemples few-shot qui servira de référence au skill
Output: references/exemples/requetes_fondamentales.md contenant 6+ paires NL→SQL validées par exécution
</objective>

<execution_context>
Execute this plan with: /run-us .planning/E01-fondations-glossaire/F02-exemples-fondamentaux/US-003-exemples-nl-sql-base/US-003-PLAN.md
The run-us skill handles execution protocol, deviation rules, and summary creation.
</execution_context>

<context>
@.planning/BRIEF.md
@.planning/E01-fondations-glossaire/EPIC.md
@.planning/E01-fondations-glossaire/F02-exemples-fondamentaux/FEATURE.md
@.planning/E01-fondations-glossaire/F02-exemples-fondamentaux/US-003-exemples-nl-sql-base/US-003.md
@references/metier/glossaire.md
@references/metier/kpis.md
@DB-SCHEMA.md
</context>

<acceptance_criteria>
AC1: Exemple version courante — SQL inclut INNER JOIN id_mapping avec end_date IS NULL
AC2: Exemple résolution idcor — SQL résout invoices_idcor_account via id_mapping avec alias acc_im
AC3: Exemple jointure article — SQL joint via id_product = articles_id_databox::text
AC4: Exemple agrégation temporelle — SQL utilise generate_series pour inclure les mois sans données
AC5: Exemple avec invoices_sign — SQL multiplie les montants par invoices_sign dans le SUM
AC6: Au moins 6 exemples validés par exécution sans erreur SQL
</acceptance_criteria>

<tasks>

<task id="1" type="auto" maps-to="AC1,AC2,AC3,AC4,AC5,AC6">
## Construire et valider les exemples NL→SQL

Pour chaque pattern fondamental, construire un exemple complet (question NL + SQL + explication) et l'exécuter sur la base via mcp__databox-db__query.

**Exemples à produire (minimum 6) :**

1. **Version courante** (AC1)
   - Question : "Liste tous les clients actifs"
   - Pattern : INNER JOIN id_mapping + end_date IS NULL + définition client actif du glossaire (commande dans les 12 derniers mois)

2. **Résolution idcor** (AC2)
   - Question : "Quelles sont les factures du client VELOLAND CHOLET ?"
   - Pattern : Résolution invoices_idcor_account → id_mapping (alias acc_im) → accounts

3. **Jointure article avec cast TEXT** (AC3)
   - Question : "Quel article se vend le plus en quantité ?"
   - Pattern : sales_orders_lines_id_product = art.articles_id_databox::text

4. **Agrégation temporelle avec generate_series** (AC4)
   - Question : "Quel est mon CA facturé par mois en 2024 ?"
   - Pattern : generate_series + LEFT JOIN + COALESCE pour les mois vides
   - Note : Utiliser la plage 2024 (données disponibles de 2015 à jan 2025)

5. **invoices_sign dans les agrégations** (AC5)
   - Question : "Quel est le CA net facturé en 2024 ?"
   - Pattern : SUM(invoices_amount_lines_excl_vat * invoices_sign)

6. **Aliases multiples id_mapping** (AC1+AC2 combiné)
   - Question : "CA facturé par client en 2024, avec nom du commercial"
   - Pattern : 3 jointures id_mapping avec aliases distincts (doc_im, acc_im, rep_im)

**Pour chaque exemple :**
1. Écrire la question en français
2. Écrire le SQL complet
3. Exécuter via mcp__databox-db__query
4. Vérifier : pas d'erreur SQL, résultats cohérents
5. Noter les points clés (quel pattern est illustré, quelles règles Databox appliquées)
</task>

<task id="2" type="auto" maps-to="AC6">
## Écrire references/exemples/requetes_fondamentales.md

Créer le fichier avec tous les exemples validés dans la tâche 1.

**Structure par exemple :**

```markdown
### Exemple N : [Titre — pattern illustré]

**Question :** "[question en français]"

**SQL :**
```sql
[requête complète]
```

**Points clés :**
- [Pattern Databox illustré]
- [Règle(s) critique(s) appliquée(s)]

**Résultat (extrait) :**
[2-3 lignes de résultat pour illustrer]
```

**Actions :**
1. Créer le dossier `references/exemples/` s'il n'existe pas
2. Écrire requetes_fondamentales.md avec les 6+ exemples
3. Ajouter un en-tête récapitulant les patterns couverts
</task>

</tasks>

<verification>
- [ ] 6+ exemples NL→SQL complets
- [ ] Chaque exemple exécuté avec succès sur la base
- [ ] AC1 (version courante), AC2 (idcor), AC3 (cast TEXT), AC4 (generate_series), AC5 (invoices_sign) explicitement couverts
- [ ] Fichier references/exemples/requetes_fondamentales.md créé
- [ ] Toutes les règles Databox respectées dans chaque exemple
</verification>

<success_criteria>
- AC1: vérifié par présence de INNER JOIN id_mapping + end_date IS NULL dans l'exemple
- AC2: vérifié par présence de la résolution idcor avec alias acc_im
- AC3: vérifié par présence de ::text dans la jointure article
- AC4: vérifié par présence de generate_series avec périodes vides
- AC5: vérifié par présence de * invoices_sign dans le SUM
- AC6: vérifié par exécution réussie de chaque requête sur la base
</success_criteria>

<output>
Create US-003-SUMMARY.md in this directory.
</output>
