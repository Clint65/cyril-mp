---
description: Afficher le schéma Databox — tables, relations, conventions et patterns SQL
disable-model-invocation: true
---

Lis et présente un résumé structuré du schéma Databox à partir de ces fichiers de référence :

1. **Schéma des tables** : `docs/DB-SCHEMA.md`
2. **Système de versioning** : `docs/ID-MAPPING.md`
3. **Listes de valeurs** : `docs/VALUE-LISTS-ARCHITECTURE.md`

Affiche :
- La liste des domaines avec leurs tables principales
- Les conventions critiques (filtre `end_date IS NULL`, résolution `idcor`, cast TEXT articles, `invoices_sign`)
- Les relations cross-domaines les plus utilisées

Si l'utilisateur passe un argument (ex: `/databox-query:databox-schema ventes`), filtre l'affichage sur ce domaine uniquement.

Argument utilisateur : "$ARGUMENTS"
