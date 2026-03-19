# Databox Query — Skill NL→SQL

Skill Claude Code pour interroger la base PostgreSQL **Databox** (ERP/CRM) en langage naturel français.

## Prérequis

- PostgreSQL avec le schéma `databox`
- Node.js / npx disponible

## Installation du serveur MCP

Le skill nécessite un accès lecture seule à la base Databox via un serveur MCP PostgreSQL.

### 1. Créer un utilisateur PostgreSQL lecture seule

```sql
CREATE USER claude_readonly WITH PASSWORD 'votre_mot_de_passe';
GRANT CONNECT ON DATABASE votre_base TO claude_readonly;
GRANT USAGE ON SCHEMA databox TO claude_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA databox TO claude_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA databox GRANT SELECT ON TABLES TO claude_readonly;
```

### 2. Créer le fichier `.mcp.json` dans votre projet

Dans le dossier où vous lancez Claude, créez un fichier `.mcp.json` :

```json
{
  "mcpServers": {
    "databox-db": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://claude_readonly:MOT_DE_PASSE@HOTE:5432/NOM_BASE"
      ]
    }
  }
}
```

Remplacer `MOT_DE_PASSE`, `HOTE` et `NOM_BASE` par vos valeurs.

> **Ne jamais commiter `.mcp.json`** — ajoutez-le à votre `.gitignore`.

### 3. Vérifier la connexion

Lancez Claude dans le dossier contenant `.mcp.json`, puis :

```
Liste les tables disponibles dans le schéma databox.
```

Le serveur MCP doit apparaître comme **connected** dans `/mcp`.

## Utilisation

Une fois le MCP configuré, posez vos questions en français :

- "Quel est mon CA facturé par mois cette année ?"
- "Taux de conversion devis/commandes par commercial"
- "Top 10 clients par chiffre d'affaires"
- "Quels clients n'ont pas commandé depuis 3 mois ?"

## Documentation technique

- `docs/DB-SCHEMA.md` — Structure complète des tables
- `docs/ID-MAPPING.md` — Système de versioning (SCD Type 2)
- `docs/VALUE-LISTS-ARCHITECTURE.md` — Décodage des colonnes codées
