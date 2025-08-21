# EF Core CLI – Commands

Works with EF Core 7-9. Replace the provider package (e.g., `Microsoft.EntityFrameworkCore.SqlServer`) as needed.

---

# Setup & Tooling

```bash
# Install as a global tool
dotnet tool install --global dotnet-ef
dotnet tool update  --global dotnet-ef

# Or as a local tool (per repo)
dotnet new tool-manifest
dotnet tool install dotnet-ef
dotnet tool update  dotnet-ef

# Verify
dotnet ef --version
```

**Project prerequisites:** Add `Microsoft.EntityFrameworkCore.Design` to the project that contains your `DbContext`.

---

# Common Flags You’ll Use Everywhere

* `-p|--project <PATH>`: Project containing the `DbContext` (e.g., Infrastructure).
* `-s|--startup-project <PATH>`: ASP.NET Core startup (where Program.cs is).
* `-c|--context <NAME>`: Specific `DbContext` when multiple exist.
* `-f|--framework <TFM>`: Target framework (e.g., `net9.0`).
* `--configuration <CFG>`: Build config (e.g., `Release`).
* `--no-build`: Skip building.
* `--verbose`: Extra diagnostics.

**Examples**

```bash
dotnet ef migrations add Init -p src/Infra -s src/Api -c AppDbContext
dotnet ef database update -p src/Infra -s src/Api
```

---

# DbContext Utilities

```bash
# List discovered DbContexts
dotnet ef dbcontext list -p src/Infra -s src/Api

# Show model info for a DbContext
dotnet ef dbcontext info -c AppDbContext -p src/Infra -s src/Api

# Generate compiled/optimized model (improves startup)
dotnet ef dbcontext optimize -c AppDbContext -o Generated/CompiledModel -n MyApp.CompiledModel \
  -p src/Infra -s src/Api
```

---

# Migrations Lifecycle

```bash
# Create a migration
dotnet ef migrations add Init -o Migrations -c AppDbContext -p src/Infra -s src/Api

# List migrations
dotnet ef migrations list -p src/Infra -s src/Api

# Remove the last (unapplied) migration
dotnet ef migrations remove -p src/Infra -s src/Api

# Update DB to latest (apply all pending)
dotnet ef database update -p src/Infra -s src/Api

# Update DB to a specific migration
dotnet ef database update Init -p src/Infra -s src/Api

# Roll back ALL migrations (to 0)
dotnet ef database update 0 -p src/Infra -s src/Api
```

---

# SQL Scripts (for DBAs / CI)

```bash
# Generate script from initial (0) to latest
dotnet ef migrations script -p src/Infra -s src/Api -o sql/full.sql

# Idempotent script (safe to run multiple times)
dotnet ef migrations script --idempotent -p src/Infra -s src/Api -o sql/idempotent.sql

# Script a range of migrations
dotnet ef migrations script Init AddStudents -p src/Infra -s src/Api -o sql/range.sql
```

---

# Migration Bundle (Self-Contained Migrator EXE)

```bash
# Produce a single file you can run on servers/containers
dotnet ef migrations bundle \
  --self-contained \
  --target-runtime linux-x64 \
  --configuration Release \
  --project src/Infra \
  --startup-project src/Api \
  --output ./publish/migrate

# Run it (optionally pass connection)
./publish/migrate --connection "Server=...;Database=...;User Id=...;Password=...;TrustServerCertificate=True"
```

---

# Database Commands

```bash
# Apply migrations to database
dotnet ef database update -p src/Infra -s src/Api

# Drop database (force/no prompt)
dotnet ef database drop --force -p src/Infra -s src/Api
```

---

# Reverse Engineering (Scaffold From Existing DB)

```bash
# Basic scaffold (SQL Server provider shown)
dotnet ef dbcontext scaffold "Server=.;Database=UniDb;Trusted_Connection=True;TrustServerCertificate=True" \
  Microsoft.EntityFrameworkCore.SqlServer \
  --context AppDbContext \
  --context-dir Data \
  --output-dir Models \
  --data-annotations \
  --use-database-names \
  -p src/Infra -s src/Api

# Target specific schemas/tables
dotnet ef dbcontext scaffold "<conn>" Microsoft.EntityFrameworkCore.SqlServer \
  --schema dbo --table Students --table Enrollments \
  -p src/Infra -s src/Api

# Overwrite existing scaffolded files
dotnet ef dbcontext scaffold "<conn>" Microsoft.EntityFrameworkCore.SqlServer --force \
  -p src/Infra -s src/Api
```

**Tip:** For secrets, prefer `dotnet user-secrets` or environment variables over hardcoding the connection string.

---

# Environment & Configuration

```bash
# Bash
ASPNETCORE_ENVIRONMENT=Development dotnet ef database update -p src/Infra -s src/Api

# PowerShell
$env:ASPNETCORE_ENVIRONMENT="Development"
dotnet ef migrations add AddCourse -p src/Infra -s src/Api
```

EF uses your **startup project’s** configuration (appsettings, user-secrets, Key Vault, etc.) to resolve the connection.

---

# Quick Recipes

**Monorepo with clean architecture**

```bash
dotnet ef migrations add Init -p src/Infrastructure -s src/Api -c UniversityDbContext
dotnet ef database update    -p src/Infrastructure -s src/Api
```

**Multiple DbContexts**

```bash
dotnet ef migrations add InitStudents -c StudentsDbContext -o Migrations/Students -p src/Infra -s src/Api
dotnet ef migrations add InitBilling  -c BillingDbContext  -o Migrations/Billing  -p src/Infra -s src/Api
```

**Rebuild from scratch (local dev)**

```bash
dotnet ef database drop --force -p src/Infra -s src/Api
dotnet ef database update      -p src/Infra -s src/Api
```

**Generate idempotent script for deployment**

```bash
dotnet ef migrations script --idempotent -p src/Infra -s src/Api -o artifacts/migrate.sql
```

---

# Troubleshooting Tips

* If CLI can’t find your `DbContext`, ensure the **startup project** builds and can create it at design time (or implement `IDesignTimeDbContextFactory<T>`).
* Use `--verbose` to see assembly loading and configuration issues.
* Cross-platform path issues? Prefer forward slashes or quote paths with spaces.
* Targeting multiple TFMs? Add `-f net9.0` (or appropriate TFM).

---
