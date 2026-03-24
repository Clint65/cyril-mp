# Patterns de Services DataOps - ThingShapes

## Fichier de référence
Analysé depuis : `08-SourceControl/DataBox/ThingShapes/DataBox_TS_Accounts.xml`

## Règle critique : préfixer les noms de services

Toutes les ThingShapes DataOps sont implémentées par le **même Thing** (`DataBox_Helper_Platform`). Si deux ThingShapes ont un service avec le même nom, il y a **conflit**.

**Règle : toujours préfixer les noms de services avec le nom de l'entité parente.**

| Mauvais (conflit) | Bon (préfixé) | Pourquoi |
|-------------------|---------------|----------|
| `getAddressLines` | `getSupplierAddressLines` | Conflit avec `getAddressLines` de DataBox_TS_Accounts |
| `getContactLines` | `getCarrierContactLines` | Conflit avec `getContactLines` de DataBox_TS_Accounts |
| `getAddressLine` | `getSupplierAddressLine` | Conflit avec `getAddressLine` de DataBox_TS_Accounts |
| `getContactLine` | `getSupplierContactLine` | Conflit avec `getContactLine` de DataBox_TS_Accounts |
| `getAddressLinesGridConfig` | `getSupplierAddressLinesGridConfig` | Conflit aussi |
| `setSelectedAccount` | `setSelectedSupplier` | Chaque entité a son propre passthrough |

**Quand préfixer :**
- Dès qu'une sous-entité (addressLines, contactLines, etc.) est partagée entre plusieurs entités parentes (Accounts, Suppliers, Carriers)
- Pour les services passthrough (`setSelected[Entity]`)
- Pour les GridConfig de sous-entités partagées

**Le service principal de l'entité** (`getSupplier`, `getCarrier`) n'a pas besoin de préfixe supplémentaire car le nom de l'entité est déjà unique.

## Quand utiliser quel pattern ?

| Situation | Pattern à utiliser | Exemple |
|-----------|-------------------|---------|
| Afficher le détail d'une entité (clic sur une ligne) | **GET par ID/IDCorrespondence** | `getAccount(id, idcor)` |
| Afficher les documents liés à une entité (onglets dans Details) | **GET sous-entités** | `getAccountQuotes(id)`, `getAccountInvoices(id)` |
| Afficher les lignes d'un document (lignes de devis, de commande) | **GET lignes** | `getQuoteLines(id)`, `getAddressLines(idAccount)` |
| Configurer les colonnes d'une grille FCADDataGrid dans un ThingShape | **GridConfig** (avec small/pagination) | `getAccountsGridConfig(small, pagination)` |
| Configurer les colonnes d'une grille pour des lignes de sous-entité | **GridConfig colonnes spécifiques** | `getAddressLinesGridConfig(small, pagination)` |
| Propager la sélection d'une entité entre tiles | **setSelected** (passthrough) | `setSelectedAccount(selectedAccount)` |
| Lister toutes les entités dans une tile Grid (pagination serveur) | Pas dans le ThingShape — utiliser `DataBox_Helper_Platform.FetchData` ou `.GetInfotable` directement dans le mashup | Binding dans le JSON mashup |

## Structure d'un ThingShape

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Entities majorVersion="9" minorVersion="7" universal="password">
    <ThingShapes>
        <ThingShape className="" description="" documentationContent=""
         homeMashup="" name="DataBox_TS_[EntityName]" projectName="DataBox" tags="">
            <PropertyDefinitions></PropertyDefinitions>
            <ServiceDefinitions>
                <!-- Déclarations des services (signature) -->
            </ServiceDefinitions>
            <EventDefinitions></EventDefinitions>
            <ServiceMappings></ServiceMappings>
            <ServiceImplementations>
                <!-- Implémentations JavaScript des services -->
            </ServiceImplementations>
            <!-- Permissions, etc. -->
        </ThingShape>
    </ThingShapes>
</Entities>
```

## Types de services

### 1. Service GET par ID/IDCorrespondence

**Déclaration :**
```xml
<ServiceDefinition aspect.isAsync="false" category="" description=""
 isAllowOverride="false" isLocalOnly="false" isOpen="false" isPrivate="false"
 name="get[Entity]">
    <ResultType aspect.dataShape="DataBox_DS_Identification" baseType="INFOTABLE"
     description="" name="result" ordinal="0"></ResultType>
    <ParameterDefinitions>
        <FieldDefinition baseType="STRING" description="" name="id" ordinal="1"></FieldDefinition>
        <FieldDefinition baseType="STRING" description="" name="idcor" ordinal="2"></FieldDefinition>
    </ParameterDefinitions>
</ServiceDefinition>
```

**Implémentation :**
```javascript
if(id){
    result = Things["DataBox_Helper_Platform"].GetInfotable({
        api: "[entity_path]/"+id
    });
}
else if (idcor){
    result = Things["DataBox_Helper_Platform"].GetInfotable({
        api: "[entity_path]/idCorrespondence/"+idcor
    });
}
else{
    result = DataShapes["DataBox_DS_Identification"].CreateValues();
}
```

### 2. Service GET sous-entités (relations)

**Déclaration :**
```xml
<ServiceDefinition aspect.isAsync="false" category="" description=""
 isAllowOverride="false" isLocalOnly="false" isOpen="false" isPrivate="false"
 name="get[Entity][SubEntity]">
    <ResultType aspect.dataShape="DataBox_DS_Identification" baseType="INFOTABLE"
     description="" name="result" ordinal="0"></ResultType>
    <ParameterDefinitions>
        <FieldDefinition baseType="STRING" description="" name="id" ordinal="1"></FieldDefinition>
    </ParameterDefinitions>
</ServiceDefinition>
```

**Implémentation :**
```javascript
result = Things["DataBox_Helper_Platform"].GetInfotable({
    api: "[entity_path]/[subEntity]/"+id
});
```

**Exemples réels :**
- `getAccountQuotes` → `api: "accounts/quotes/"+id`
- `getAccountInvoices` → `api: "accounts/invoices/"+id`
- `getAccountDeliveries` → `api: "accounts/deliveries/"+id`
- `getAccountSalesOrders` → `api: "accounts/salesOrders/"+id`

### 3. Service GET lignes (sous-ressources)

**Déclaration :** Même que GET sous-entités, avec paramètre `idAccount` ou `id[Entity]`

**Implémentation :**
```javascript
let result = Things["DataBox_Helper_Platform"].GetInfotable({
    api: "[entity_path]/[linesPath]/"+idAccount
});
```

**Exemples réels :**
- `getAddressLines` → `api: "accounts/addressLines/"+idAccount`
- `getContactLines` → `api: "accounts/contactLines/"+idAccount`

### 4. Service GridConfig

**Déclaration :**
```xml
<ServiceDefinition aspect.isAsync="false" category="" description=""
 isAllowOverride="false" isLocalOnly="false" isOpen="false" isPrivate="false"
 name="get[Entity]GridConfig">
    <ResultType baseType="JSON" description="" name="result" ordinal="0"></ResultType>
    <ParameterDefinitions>
        <FieldDefinition baseType="BOOLEAN" description="" name="pagination" ordinal="2"></FieldDefinition>
        <FieldDefinition baseType="BOOLEAN" description="" name="small" ordinal="1"></FieldDefinition>
    </ParameterDefinitions>
</ServiceDefinition>
```

**Implémentation :**
```javascript
if (small){
    result = me.GetGridConfig({table:"[api_table_name]", pagination:pagination, columns: []});
}
else {
    result = me.GetGridConfigPagination({table:"[api_table_name]"});
}
```

### 5. Service GridConfig avec colonnes spécifiques (pour lignes)

**Implémentation :**
```javascript
if (small){
    let columns = [
        "[entity]Lines[Field1]",
        "[entity]Lines[Field2]",
        "[entity]Lines[Field3]",
    ]
    result = me.GetGridConfig({table:"[api_table_name]", pagination:pagination, columns: columns})
}
else {
    result = me.GetGridConfigPagination({table:"[api_table_name]"});
}
```

### 6. Service setSelected (passthrough)

**Implémentation :**
```javascript
result = selected[Entity];
```

## Structure d'une ServiceImplementation

```xml
<ServiceImplementation description="" handlerName="Script" name="[serviceName]">
    <ConfigurationTables>
        <ConfigurationTable dataShapeName="" description="" isMultiRow="false"
         name="Script" ordinal="0">
            <DataShape>
                <FieldDefinitions>
                    <FieldDefinition baseType="STRING" description="code"
                     name="code" ordinal="0"></FieldDefinition>
                </FieldDefinitions>
            </DataShape>
            <Rows>
                <Row>
                    <code>
                    <![CDATA[
                    // JavaScript code here
                    ]]>
                    </code>
                </Row>
            </Rows>
        </ConfigurationTable>
    </ConfigurationTables>
</ServiceImplementation>
```

## Attention : noms de routes API irréguliers

Les noms de sous-routes dans l'API NestJS ne suivent **aucun pattern régulier**. Il faut toujours les copier depuis le controller, jamais les deviner.

| Entité | Route lignes dans le controller | Piège |
|--------|-------------------------------|-------|
| purchaseOrders | `purchaseOrdersLines` | Avec le **s** de Orders |
| salesOrders | `salesOrderLines` | **Sans** le s |
| invoices | `invoiceLines` | **Singulier** |
| quotes | `quoteLines` | **Singulier** |
| deliveries | `deliveryLines` | **Singulier** |
| serviceContracts | `serviceContractLines` | **Singulier** |
| customerAssets | `customerAssetLines` | **Singulier** |
| purchaseInvoices | `purchaseInvoicesLines` | Avec le **s** |
| receipts | `receiptsLines` | Avec le **s** |

**Règle absolue** : lire le `@Get('...')` du controller et copier le nom exact.

## API Helper utilisé

Tous les services DataOps utilisent `Things["DataBox_Helper_Platform"]` avec :
- `.GetInfotable({api: "..."})` pour les requêtes GET retournant INFOTABLE
- `.Get({api: "..."})` pour les requêtes GET retournant JSON
- `.Post({api: "...", body: JSON.stringify({...})})` pour les requêtes POST

L'URL API complète est construite comme : `api_base_url + "/databox/" + api_path`
