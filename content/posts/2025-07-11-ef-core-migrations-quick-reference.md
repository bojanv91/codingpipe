---
title: "EF Core Migrations Quick Reference"
date: 2025-07-11
dateUpdated: Last Modified
permalink: /posts/ef-core-migrations-quick-reference/
tags:
  - EF Core
  - Reference
layout: layouts/post.njk
---

EF Core migration commands that I find myself looking up constantly.

Run these from the project directory containing your DbContext. Add `--project <ProjectName>` if running from solution root. Use `--output-dir Data/Migrations` to organize migrations in a specific folder.

```powershell
# Install EF Core CLI globally
dotnet tool install --global dotnet-ef

# Check migration status and list all migrations
dotnet ef migrations list

# Apply pending migrations to database
dotnet ef database update

# Generate new migration with descriptive name
dotnet ef migrations add <MigrationName>
dotnet ef migrations add <MigrationName> --output-dir Data/Migrations

# Drop database
dotnet ef database drop

# Remove all migrations
dotnet ef migrations remove --force

# View DbContext information and connection details
dotnet ef dbcontext info
```
