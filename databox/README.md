# Databox Query — Skill NL->SQL

Skill Claude Code pour interroger la base PostgreSQL **Databox** (ERP/CRM) en langage naturel francais.

## Prerequis

- PostgreSQL avec le schema `databox`
- Node.js / npx disponible
- Variable d'environnement `DATABOX_DATABASE_URL`

## Installation

```bash
/plugin marketplace add Clint65/cyril-mp
/plugin install databox-query@cyril-mp
```

## Configuration

Le serveur MCP PostgreSQL est integre au plugin. Il suffit de definir la variable d'environnement avec votre connection string :

```bash
export DATABOX_DATABASE_URL="postgresql://claude_readonly:MOT_DE_PASSE@HOTE:5432/NOM_BASE"
```

Ajoutez cette ligne a votre `~/.zshrc`, `~/.bashrc` ou fichier de secrets equivalent.

### Creer un utilisateur PostgreSQL lecture seule

```sql
CREATE USER claude_readonly WITH PASSWORD 'votre_mot_de_passe';
GRANT CONNECT ON DATABASE votre_base TO claude_readonly;
GRANT USAGE ON SCHEMA databox TO claude_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA databox TO claude_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA databox GRANT SELECT ON TABLES TO claude_readonly;
```

### Verifier la connexion

Lancez Claude puis :

```
Liste les tables disponibles dans le schema databox.
```

Le serveur MCP doit apparaitre comme **connected** dans `/mcp`.

## Utilisation

Posez vos questions en francais :

- "Quel est mon CA facture par mois cette annee ?"
- "Taux de conversion devis/commandes par commercial"
- "Top 10 clients par chiffre d'affaires"
- "Quels clients n'ont pas commande depuis 3 mois ?"

### Commande schema

```
/databox-query:databox-schema           # Schema complet
/databox-query:databox-schema ventes    # Filtre sur un domaine
```

## Documentation technique

- `docs/DB-SCHEMA.md` — Structure complete des tables
- `docs/ID-MAPPING.md` — Systeme de versioning (SCD Type 2)
- `docs/VALUE-LISTS-ARCHITECTURE.md` — Decodage des colonnes codees
