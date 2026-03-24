# Workflow : Générer un mashup à partir d'un mashup existant (cp+sed+python)

## Objectif
Les gros mashups Details (>50Ko, >3000 lignes) dépassent la capacité de génération directe par un agent. L'approche correcte est : **copier un mashup existant similaire, puis le transformer** via sed et scripts Python.

## Quand utiliser ce workflow

- Mashup Dashboard Details (AccountsDetails → SuppliersDetails)
- Mashup avec sous-entités multiples (onglets, grilles liées)
- Tout fichier mashup > 50Ko

## Processus

### Étape 1 : Choisir le modèle source

Choisir le mashup existant le plus proche **par structure**, pas juste par nom :

| Pour créer | Bon modèle | Mauvais modèle |
|-----------|-----------|----------------|
| SuppliersDetails (2 sous-entités) | CustomerDueDatesDetails (simple) | AccountsDetails (4 sous-entités → doublons) |
| CarriersDetails (2 sous-entités) | CustomerDueDatesDetails | AccountsDetails |
| PurchaseOrdersDetails (1 sous-entité) | CustomerDueDatesDetails | AccountsDetails |
| EntityAddressDetails | AccountsAddressDetails | AccountsAddress (c'est la liste, pas le détail !) |

**Règle** : choisir un modèle avec le **même nombre de sous-entités** que la cible, ou moins. Jamais plus (sinon doublons de services, bindings, events).

### Étape 2 : Copier le fichier

```bash
cp 08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[Source].xml \
   08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[Target].xml
```

### Étape 3 : Remplacement global par sed

```bash
# Remplacer le nom de l'entité (ATTENTION à la casse)
sed -i '' \
  -e 's/[SourceEntity]/[TargetEntity]/g' \
  -e 's/[sourceEntity]/[targetEntity]/g' \
  -e 's/[source_entity]/[target_entity]/g' \
  08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[Target].xml
```

### Étape 4 : Régénérer tous les UUIDs

Les UUIDs doivent être uniques par mashup. Utiliser un script Python :

```python
import re, uuid

filepath = "08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[Target].xml"
with open(filepath, 'r') as f:
    content = f.read()

# Remplacer chaque UUID par un nouveau unique
uuid_pattern = r'[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}'
seen = {}
def replace_uuid(match):
    old = match.group(0)
    if old not in seen:
        seen[old] = str(uuid.uuid4())
    return seen[old]

content = re.sub(uuid_pattern, replace_uuid, content)

with open(filepath, 'w') as f:
    f.write(content)
```

### Étape 5 : Adapter les services au contexte métier

**C'est l'étape critique.** Le sed remplace les noms mais pas la logique métier.

Exemple : AccountsDetails a 4 sous-entités (quotes, salesOrders, invoices, deliveries).
Pour SuppliersDetails, il faut **remplacer** ces sous-entités par celles du fournisseur (purchaseOrders, purchaseInvoices, receipts).

Il faut modifier **manuellement** dans le JSON mashupContent :

1. **Section Data > Services** : remplacer les noms de services
   - `getAccountQuotes` → `getSupplierPurchaseOrders`
   - `getQuotesGridConfig` → `getPurchaseOrdersGridConfig`

2. **Section Events** : adapter les ServiceInvokeCompleted chains

3. **Section DataBindings** : adapter les SourceId/TargetId

4. **Section UI > Widgets > Tabs** : adapter Title, Icon pour chaque onglet

Si le modèle avait plus de sous-entités que la cible, **supprimer les doublons** avec un script Python :

```python
import json

# Charger le JSON mashupContent, dédupliquer les services dans Data
# Supprimer les bindings et events qui référencent des services supprimés
```

### Étape 6 : Adapter les sous-entités spécifiques

Les points souvent oubliés après sed :

| Champ JSON | Exemple de résidu | Correction |
|-----------|-------------------|------------|
| `"Title"` des onglets | "Devis", "Commandes" | Remplacer par les titres de la nouvelle entité |
| `"Icon"` des onglets | "fa-solid fa-tags" | Adapter l'icône au contexte |
| `"LabelText"` | "Date d'échéance client" | Remplacer le sous-titre |
| `"DisplayName"` | "gotoAccount" | Remplacer par "goto[Entity]" |
| `"api"` dans Parameters | "contacts/columns" | Remplacer par la bonne route API |
| `"table"` dans Parameters | "accounts" | Remplacer par le nom de table cible |
| `ColumnBaseUrls` | "accountsAccountCode" | Remplacer par les noms de colonnes de la nouvelle entité |
| `selectedAccounts` dans Session | | Remplacer par `selected[Entity]` |
| `setSelectedAccount` | | Remplacer par `setSelected[Entity]` |

### Étape 7 : Audit post-génération

**Obligatoire** avant de considérer le mashup comme terminé.

```bash
# 1. Vérifier 0 résidus de l'entité source
grep -ci "[SourceEntity]" 08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[Target].xml
# Doit retourner 0

# 2. Vérifier que tous les services référencés existent dans le ThingShape
grep -o '"Name": "[^"]*"' 08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[Target].xml | sort -u

# 3. Vérifier pas de doublons de services dans le JSON
python3 -c "
import json, re
with open('08-SourceControl/DataBox/Mashups/DataBox_Dashboard_[Target].xml') as f:
    content = f.read()
match = re.search(r'<mashupContent><!\[CDATA\[(.*?)\]\]></mashupContent>', content, re.DOTALL)
if match:
    data = json.loads(match.group(1))
    for section_name, section in data.get('Data', {}).items():
        services = [s['Name'] for s in section.get('Services', [])]
        if len(services) != len(set(services)):
            print(f'DOUBLONS dans {section_name}: {[s for s in services if services.count(s) > 1]}')
    print('Audit terminé')
"
```

### Checklist de validation post-cp+sed

- [ ] `grep -ci "[SourceEntity]"` retourne **0** (aucun résidu)
- [ ] Tous les services dans le JSON Data existent dans le ThingShape cible
- [ ] Pas de doublons de services dans la section Data
- [ ] Les noms de sous-entités correspondent au contexte métier (pas juste un remplacement textuel)
- [ ] Les Title/Icon des onglets sont adaptés
- [ ] Les LabelText/sous-titres sont remplacés
- [ ] Les DisplayName des navigationfunction sont remplacés
- [ ] Les chemins `api` dans les Parameters pointent vers les bonnes routes
- [ ] Les `table` dans les Parameters pointent vers les bonnes tables
- [ ] Les noms de colonnes dans ColumnBaseUrls sont ceux de la nouvelle entité
- [ ] Les Session properties (selected[Entity]) sont remplacées
- [ ] Les services passthrough (setSelected[Entity]) sont remplacés
- [ ] Les UUIDs sont tous uniques (pas de copie d'un autre mashup)
- [ ] Les DataBindings ne sont pas croisés entre grilles (vérifier SourceId ↔ TargetId)
