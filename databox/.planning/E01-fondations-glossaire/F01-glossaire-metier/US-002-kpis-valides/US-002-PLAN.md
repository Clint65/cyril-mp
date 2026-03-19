---
story: US-002
feature: F01
epic: E01
type: execute
---

<objective>
KPIs validés avec formules SQL

**As a** utilisateur métier
**I want** que chaque KPI ait une formule SQL Databox précise et validée
**So that** les calculs soient reproductibles et cohérents d'une question à l'autre

Purpose: Compléter le socle sémantique avec les formules SQL des indicateurs de performance
Output: references/metier/kpis.md contenant 5+ KPIs avec formules SQL complètes et validées par exécution
</objective>

<execution_context>
Execute this plan with: /run-us .planning/E01-fondations-glossaire/F01-glossaire-metier/US-002-kpis-valides/US-002-PLAN.md
The run-us skill handles execution protocol, deviation rules, and summary creation.
</execution_context>

<context>
@.planning/BRIEF.md
@.planning/E01-fondations-glossaire/EPIC.md
@.planning/E01-fondations-glossaire/F01-glossaire-metier/FEATURE.md
@.planning/E01-fondations-glossaire/F01-glossaire-metier/US-002-kpis-valides/US-002.md
@.planning/E01-fondations-glossaire/F01-glossaire-metier/US-001-definitions-metier/US-001-SUMMARY.md
@references/metier/glossaire.md
@DB-SCHEMA.md
</context>

<acceptance_criteria>
AC1: KPI "Évolution CA" inclut formule SQL avec generate_series, granularité (mois/trimestre/année), traitement des périodes sans données
AC2: KPI "Taux de conversion" inclut formule (nb convertis / nb total * 100), filtres de période, ventilation par commercial
AC3: KPI "Marge" précise colonne source, calcul (absolue vs %), traitement des avoirs
AC4: Fichier kpis.md structuré avec pour chaque KPI : nom, description, formule SQL complète, paramètres, exemple de résultat attendu
</acceptance_criteria>

<tasks>

<task id="1" type="auto" maps-to="AC1,AC2,AC3">
## Construire et valider les formules SQL de chaque KPI

S'appuyer sur le glossaire validé (references/metier/glossaire.md) pour construire les formules SQL complètes de chaque KPI. Exécuter chaque requête sur la base pour valider qu'elle fonctionne.

**KPIs à couvrir (minimum 5) :**

1. **Évolution CA (facturé)** — generate_series par mois/trimestre/année, SUM(montant * invoices_sign), LEFT JOIN pour inclure les périodes vides
2. **Évolution CA (commandé)** — Même pattern avec sales_orders, filtre order_status IN (1, 2)
3. **Taux de conversion devis→commandes** — COUNT FILTER sur quotes_quote_status IN ('2','3') / COUNT total, ventilable par commercial (résolution idcor_sales_rep)
4. **Marge brute** — Absolue : SUM(invoices_lines_line_margin * invoices_sign). Pourcentage : marge / CA * 100. Variante commandé avec sales_orders_total_margin
5. **Panier moyen** — SUM(sales_orders_amount_lines_excl_vat) / COUNT(DISTINCT sales_orders_id_databox), filtre order_status IN (1, 2)
6. **Top N clients par CA** — Agrégation par client avec résolution idcor_account, ORDER BY DESC LIMIT N
7. **Délai moyen de paiement** — AVG(open_items_payment_date - invoices_invoice_date), jointure via document_number

**Pour chaque KPI :**
1. Écrire la requête SQL complète avec toutes les règles Databox
2. Exécuter via mcp__databox-db__query pour vérifier la syntaxe et la cohérence des résultats
3. Noter les paramètres variables (période, granularité, filtres optionnels)
4. Capturer un exemple de résultat
</task>

<task id="2" type="auto" maps-to="AC4">
## Écrire references/metier/kpis.md

Créer le fichier avec les KPIs validés dans la tâche 1.

**Structure par KPI :**

```markdown
### [Nom du KPI]
- **Description :** [Ce que mesure le KPI]
- **Source glossaire :** [Terme(s) du glossaire utilisé(s)]
- **Paramètres :**
  - `période` : date début / date fin
  - `granularité` : mois / trimestre / année (si applicable)
  - `filtre` : par commercial, par client, etc. (optionnel)
- **Formule SQL :**
  ```sql
  [Requête complète, prête à exécuter]
  ```
- **Exemple de résultat :**
  | colonne | valeur |
  |---------|--------|
  | ... | ... |
- **Règles Databox appliquées :** [end_date IS NULL, invoices_sign, etc.]
```

Écrire dans `references/metier/kpis.md`.
</task>

</tasks>

<verification>
- [ ] Au moins 5 KPIs avec formules SQL complètes
- [ ] Chaque formule exécutée avec succès sur la base
- [ ] AC1 (évolution CA), AC2 (taux de conversion), AC3 (marge) explicitement couverts
- [ ] Fichier references/metier/kpis.md créé et structuré
- [ ] Les formules respectent toutes les règles Databox
</verification>

<success_criteria>
- AC1: vérifié par exécution de la requête generate_series + CA sur la base
- AC2: vérifié par exécution de la requête taux de conversion avec ventilation par commercial
- AC3: vérifié par exécution de la requête marge absolue et %
- AC4: vérifié par existence de references/metier/kpis.md au format structuré avec 5+ KPIs
</success_criteria>

<output>
Create US-002-SUMMARY.md in this directory.
</output>
