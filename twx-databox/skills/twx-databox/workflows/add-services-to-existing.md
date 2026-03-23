# Workflow : Ajouter des services à un ThingShape existant

## Objectif
Ajouter de nouveaux services à un ThingShape existant quand de nouveaux endpoints API apparaissent.

## Processus

### Étape 1 : Identifier le ThingShape cible et les services manquants

1. Lire le ThingShape existant :
   ```bash
   cat 08-SourceControl/DataBox/ThingShapes/DataBox_TS_[EntityName].xml
   ```
2. Lister les services actuels
3. Scanner le controller NestJS pour identifier les nouveaux endpoints :
   ```bash
   cat databox-data-access/src/data-databox/[module]/*.controller.ts
   ```
4. Identifier les endpoints non couverts

### Étape 2 : Lire le fichier XML existant en entier

Lire intégralement le ThingShape pour comprendre :
- Les ServiceDefinitions existantes (section déclaration)
- Les ServiceImplementations existantes (section implémentation)
- L'ordre des services
- Les patterns utilisés (nommage des paramètres, types de retour)

### Étape 3 : Ajouter les nouvelles ServiceDefinitions

Insérer les nouvelles déclarations **avant** la balise `</ServiceDefinitions>`, en suivant le pattern des services existants du même ThingShape.

```xml
<ServiceDefinition aspect.isAsync="false" category="" description=""
 isAllowOverride="false" isLocalOnly="false" isOpen="false" isPrivate="false"
 name="[newServiceName]">
    <ResultType ... />
    <ParameterDefinitions>
        <FieldDefinition ... />
    </ParameterDefinitions>
</ServiceDefinition>
```

### Étape 4 : Ajouter les nouvelles ServiceImplementations

Insérer les nouvelles implémentations **avant** la balise `</ServiceImplementations>`, en utilisant le même style de code JavaScript que les services existants.

### Étape 5 : Valider la cohérence

Vérifier que :
- Chaque ServiceDefinition a une ServiceImplementation correspondante
- Les noms de paramètres sont cohérents avec les services existants
- Les types de retour suivent les conventions (INFOTABLE avec DataBox_DS_Identification ou JSON)
- Le code JavaScript utilise le même helper (GetInfotable, Get, Post) que les services voisins

### Étape 6 : Checklist de validation

- [ ] Chaque nouvelle ServiceDefinition a une ServiceImplementation correspondante
- [ ] Les noms de services suivent les conventions existantes du ThingShape (même style de nommage)
- [ ] Les paramètres utilisent les mêmes noms que les services voisins (ex: `id` et `idcor`, pas `identifier` et `correspondenceId`)
- [ ] Le code JavaScript utilise le même Helper que les services existants du ThingShape
- [ ] Les balises XML sont correctement fermées
- [ ] L'ordre des services dans ServiceDefinitions correspond à l'ordre dans ServiceImplementations

### Étape 7 : Résumé

Présenter :
- Services ajoutés (nom, endpoint API correspondant)
- Fichier modifié
- Rappel : l'utilisateur doit recharger dans ThingWorx
