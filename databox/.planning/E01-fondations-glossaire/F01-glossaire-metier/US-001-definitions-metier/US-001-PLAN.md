---
story: US-001
feature: F01
epic: E01
type: execute
---

<objective>
Définitions métier fondamentales

**As a** utilisateur métier interrogeant Databox
**I want** que chaque terme business courant soit traduit en colonnes et filtres SQL précis
**So that** le skill génère du SQL qui reflète fidèlement ce que je mesure dans mon activité

Purpose: Construire le socle sémantique sur lequel reposent toutes les requêtes NL→SQL
Output: references/metier/glossaire.md contenant 10+ termes métier avec leurs traductions SQL
</objective>

<execution_context>
Execute this plan with: /run-us .planning/E01-fondations-glossaire/F01-glossaire-metier/US-001-definitions-metier/US-001-PLAN.md
The run-us skill handles execution protocol, deviation rules, and summary creation.
</execution_context>

<context>
@.planning/BRIEF.md
@.planning/E01-fondations-glossaire/EPIC.md
@.planning/E01-fondations-glossaire/F01-glossaire-metier/FEATURE.md
@.planning/E01-fondations-glossaire/F01-glossaire-metier/US-001-definitions-metier/US-001.md
@DB-SCHEMA.md
@ID-MAPPING.md
@VALUE-LISTS-ARCHITECTURE.md
</context>

<acceptance_criteria>
AC1: Le terme "Chiffre d'affaires" est défini avec table source (invoices ou sales_orders), filtres (end_date IS NULL, invoices_sign), et colonne de montant
AC2: Le terme "Client actif" est défini avec critère temporel, table source et filtres
AC3: Le terme "Devis converti" est défini avec statuts (quotes_quote_status IN ('2','3')) et traitement du "partiellement commandé"
AC4: Au moins 10 termes métier validés par l'utilisateur avec définition, table(s), colonne(s), filtre(s) SQL
AC5: Le glossaire est stocké dans references/metier/glossaire.md au format structuré
</acceptance_criteria>

<tasks>

<task id="1" type="dialogue" maps-to="AC1,AC2,AC3,AC4">
## Dialogue itératif : définir les termes métier

Lire DB-SCHEMA.md pour identifier les colonnes pertinentes, puis proposer les définitions **une par une** à l'utilisateur pour validation.

**Termes à couvrir (minimum 10) :**

1. **Chiffre d'affaires (CA)** — Demander : basé sur invoices ou sales_orders ? Montant HT ou TTC ? Avoirs déduits (invoices_sign) ?
2. **CA commandé** — Distinction avec CA facturé si pertinent
3. **Marge** — Colonne total_margin ou line_margin ? Brute ou nette ? En % ou valeur absolue ?
4. **Client actif** — Seuil temporel (3 mois, 6 mois, 12 mois ?) ? Basé sur commandes ou factures ?
5. **Client inactif** — Inverse de client actif, ou critère différent ?
6. **Devis converti** — Statut '2' uniquement ou '2' + '3' ? Comptage en nombre ou montant ?
7. **Avoir** — invoices_sign = -1, comment le compter dans les analyses ?
8. **Encours client** — Tables open_items ou customer_due_dates ? Montant restant dû ?
9. **Taux de conversion** — Nb devis convertis / nb total ? Sur quelle période ? Commandes directes exclues ?
10. **Panier moyen** — CA / nb factures ou CA / nb commandes ?
11. **Délai de paiement** — Écart date facture vs date paiement ? Source de la donnée ?
12. **Commande en retard** — Basé sur asked_delivery_date vs shipment_date ?

**Méthode :**
- Proposer chaque définition avec une hypothèse par défaut
- Poser UNE question à la fois
- Attendre la validation avant de passer au terme suivant
- Noter les réponses pour la tâche 2

**checkpoint:human-verify** — L'utilisateur valide chaque définition au fil du dialogue. La tâche est terminée quand 10+ termes sont validés.
</task>

<task id="2" type="write" maps-to="AC5">
## Écrire le glossaire

Créer `references/metier/glossaire.md` avec toutes les définitions validées dans la tâche 1.

**Structure du fichier :**

```markdown
# Glossaire métier Databox

> Source de vérité pour la traduction des termes business en SQL Databox.
> Chaque définition a été validée par l'utilisateur.

## Termes

| Terme | Définition métier | Table(s) | Colonne(s) clé(s) | Filtres SQL obligatoires |
|-------|-------------------|----------|--------------------|-----------------------|
| Chiffre d'affaires | ... | invoices | invoices_amount_lines_excl_vat | end_date IS NULL, * invoices_sign |
| ... | ... | ... | ... | ... |

## Détails par terme

### Chiffre d'affaires
- **Définition :** [validée par l'utilisateur]
- **Table(s) :** [table(s) source]
- **Colonne(s) :** [colonne(s) de montant]
- **Filtres obligatoires :** [end_date IS NULL, invoices_sign, etc.]
- **Exemple SQL :**
  ```sql
  [requête minimale illustrative]
  ```
- **Ambiguïtés résolues :** [ex: "basé sur factures, pas commandes"]
```

**Actions :**
1. Créer le dossier `references/metier/` s'il n'existe pas
2. Écrire glossaire.md avec le tableau synthétique + les détails par terme
3. Inclure un exemple SQL minimal pour chaque terme
</task>

</tasks>

<verification>
- [ ] 10+ termes métier définis et validés par l'utilisateur
- [ ] Chaque terme a : définition, table(s), colonne(s), filtres SQL
- [ ] AC1 (CA), AC2 (client actif), AC3 (devis converti) explicitement couverts
- [ ] Fichier references/metier/glossaire.md créé et structuré
- [ ] Les exemples SQL respectent les règles Databox (end_date IS NULL, aliases, etc.)
</verification>

<success_criteria>
- AC1: vérifié par présence du terme "Chiffre d'affaires" dans glossaire.md avec table, colonne et filtres
- AC2: vérifié par présence du terme "Client actif" avec critère temporel et filtres
- AC3: vérifié par présence du terme "Devis converti" avec statuts et traitement du partiel
- AC4: vérifié par comptage des termes dans le tableau (>= 10)
- AC5: vérifié par existence de references/metier/glossaire.md au format tableau + détails
</success_criteria>

<output>
Create US-001-SUMMARY.md in this directory.
</output>
