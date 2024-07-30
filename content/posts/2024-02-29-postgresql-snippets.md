---
title: "PostgreSQL Query and Command Snippets"
date: 2024-02-29
dateUpdated: Last Modified
permalink: /posts/postgresql-cheatsheet/
tags:
  - Snippets
layout: layouts/post.njk
---

Set the PostgreSQL environment variables in your PowerShell session, so you won't need to add them to every single CLI command in that same session:
```powershell
# Export PostgreSQL variables
$env:PGHOST = "localhost"
$env:PGPORT = 5432
$env:PGUSER = "postgres"
$env:PGPASSWORD = "postgres"

# So now, you won't need to add this part in the commands:
# --host localhost --port 5432 --username "postgres"
```

List all databases
```powershell
psql --command \l
```

Create backup (database dump):
```powershell
pg_dump --format custom --no-owner --no-privileges --dbname mydb --file "C:/backups/mydb.backup"
```

Restore backup:
```powershell
pg_restore --format custom --no-owner --no-privileges --single-transaction --dbname mydb "C:/backups/mydb.backup"
```

Restore backup only in a specific schema:
```powershell
pg_restore --schema "schema_name_1" --format custom --no-owner --no-privileges --single-transaction --dbname mydb "C:/backups/mydb.backup"
```

Create user:
```powershell
psql --command "CREATE ROLE testuser NOSUPERUSER NOCREATEDB NOCREATEROLE NOINHERIT LOGIN NOREPLICATION NOBYPASSRLS PASSWORD 'testpsw';"
```

List users:
```powershell
psql --command \du
```

Create database:
```powershell
psql --command "CREATE DATABASE mydb;"

psql --command "CREATE DATABASE mydb OWNER testuser ENCODING 'UTF8';"
```

Create schema in database with authorization to another user
```powershell
psql --command "CREATE SCHEMA IF NOT EXISTS schema3 AUTHORIZATION testuser2;" mydb
```

List all schemas in database:
```powershell
psql --command \dn mydb
```

Iterate over all databases in PowerShell:
```powershell
$databases = psql --tuples-only --no-align --command "SELECT datname FROM pg_database WHERE datistemplate = false"

# Iterate over all databases
foreach ($db in $databases)
{
    if ($db) {
        Write-Host "Processing database: $db"
    }
}
```

## Additional References

- https://wiki.postgresql.org/wiki/Operations_cheat_sheet
- https://www.commandprompt.com/education/postgresql-basic-psql-commands/
- https://www.postgresqltutorial.com/postgresql-cheat-sheet/
- https://severalnines.com/blog/performance-cheat-sheet-postgresql
