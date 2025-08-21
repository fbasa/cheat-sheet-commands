# Essentials .NET CLI commands

```bash
dotnet --info                 # SDK/runtime details
dotnet --list-sdks            # Installed SDKs
dotnet --list-runtimes        # Installed runtimes
dotnet help                   # Top-level help
dotnet <command> --help       # Help for a specific command
dotnet sdk check              # Check for SDK/Workload updates
```

# Solutions & Projects

```bash
dotnet new sln -n MySolution
dotnet new webapi -n Api -f net9.0
dotnet new classlib -n Core -f net9.0
dotnet new xunit -n Tests -f net9.0

dotnet sln MySolution.sln add src/Api/Api.csproj src/Core/Core.csproj tests/Tests/Tests.csproj
dotnet sln MySolution.sln list
```

# Dependencies (NuGet & Project Refs)

```bash
# Project-to-project
dotnet add src/Api/Api.csproj reference src/Core/Core.csproj
dotnet list src/Api/Api.csproj reference
dotnet remove src/Api/Api.csproj reference src/Core/Core.csproj

# Packages
dotnet add src/Core/Core.csproj package FluentValidation --version 11.9.*
dotnet add src/Core/Core.csproj package Serilog.AspNetCore --prerelease
dotnet remove src/Core/Core.csproj package FluentValidation
dotnet list src/Core/Core.csproj package
dotnet list src/Core/Core.csproj package --outdated

# Feeds & cache
dotnet nuget add source https://api.nuget.org/v3/index.json -n nuget.org
dotnet nuget remove source nuget.org
dotnet nuget locals all --list
dotnet nuget locals all --clear
```

# Restore • Build • Run • Watch

```bash
dotnet restore
dotnet build -c Release -f net9.0 --no-restore
dotnet run --project src/Api/Api.csproj -- --urls=http://localhost:5005

# Hot reload / watch
dotnet watch run --project src/Api/Api.csproj
dotnet watch test --project tests/Tests/Tests.csproj
```

# Testing

```bash
dotnet test -c Release
dotnet test --filter "TestCategory=Unit"
dotnet test --logger "trx;LogFileName=test-results.trx"
dotnet test --collect "XPlat Code Coverage"
```

# Publish (runtime images, trimming, single-file)

```bash
# Framework-dependent
dotnet publish src/Api/Api.csproj -c Release -f net9.0 -o out

# Self-contained (include runtime)
dotnet publish src/Api/Api.csproj -c Release -r linux-x64 --self-contained true -o out

# Single-file + trimming + ReadyToRun (tune per app)
dotnet publish src/Api/Api.csproj -c Release -r win-x64 -p:PublishSingleFile=true \
  -p:PublishTrimmed=true -p:TrimMode=partial -p:PublishReadyToRun=true -o out
```

# Packing & Publishing Libraries

```bash
dotnet pack src/Core/Core.csproj -c Release -o ./nupkgs /p:PackageVersion=1.2.3
dotnet nuget push ./nupkgs/*.nupkg -k <API_KEY> -s https://api.nuget.org/v3/index.json
```

# dotnet tool (global/local)

```bash
# Global tools
dotnet tool install -g dotnet-ef
dotnet tool update -g dotnet-ef
dotnet tool list -g
dotnet tool uninstall -g dotnet-ef

# Local tools (per-repo)
dotnet new tool-manifest
dotnet tool install dotnet-ef
dotnet tool run dotnet-ef --help
```

# Entity Framework Core (via dotnet-ef tool)

```bash
dotnet ef migrations add Init --project src/Infra/Infra.csproj --startup-project src/Api/Api.csproj
dotnet ef database update --project src/Infra/Infra.csproj --startup-project src/Api/Api.csproj
dotnet ef dbcontext list --project src/Infra/Infra.csproj --startup-project src/Api/Api.csproj
```

# Workloads (MAUI, WebAssembly AOT, etc.)

```bash
dotnet workload list
dotnet workload search wasm
dotnet workload install wasm-tools
dotnet workload update
```

# Formatting & Analyzers

```bash
dotnet format                          # All (style, analyzers, whitespace)
dotnet format style
dotnet format analyzers
dotnet format whitespace
```

# User Secrets (dev-time)

```bash
dotnet user-secrets init --project src/Api/Api.csproj
dotnet user-secrets set "MySettings:ApiKey" "xyz" --project src/Api/Api.csproj
dotnet user-secrets list --project src/Api/Api.csproj
dotnet user-secrets clear --project src/Api/Api.csproj
```

# HTTPS Dev Certificates

```bash
dotnet dev-certs https --check
dotnet dev-certs https --trust         # (may prompt OS trust dialog)
dotnet dev-certs https --clean
```

# Diagnosing & Performance (built-in & tools)

```bash
# Built-in
dotnet --info
dotnet --version

# Install runtime diagnostics (as global tools)
dotnet tool install -g dotnet-counters
dotnet tool install -g dotnet-trace
dotnet tool install -g dotnet-dump
dotnet tool install -g dotnet-gcdump

# Examples:
dotnet-counters monitor System.Runtime --process-id <pid>
dotnet-trace collect --process-id <pid> --buffersize 256 --duration 00:02:00
dotnet-dump collect --process-id <pid>
```

# MSBuild Properties (quick reference)

You can pass properties to most commands:

```bash
dotnet build -c Release -p:ContinuousIntegrationBuild=true
dotnet publish -r linux-x64 -p:SelfContained=false -p:PublishSingleFile=true
dotnet test -p:CollectCoverage=true -p:CoverletOutput=./coverage/ -p:CoverletOutputFormat=cobertura
```

Common flags:

* `-c|--configuration` → `Debug` (default) or `Release`
* `-f|--framework` → target TFMs (`net9.0`, `net8.0`, …)
* `-r|--runtime` → RID (`win-x64`, `linux-x64`, `linux-musl-x64`, `osx-arm64`, …)
* `-o|--output` → output directory
* `--no-restore` / `--no-build` → skip steps for CI speed
* `--verbosity quiet|minimal|normal|detailed|diagnostic`

# Pin an SDK (per-repo)

```bash
dotnet new globaljson --sdk-version 9.0.100
# or edit global.json manually to lock SDK for CI reproducibility
```

# Typical Day-to-Day Sequences

**New service (API + tests):**

```bash
dotnet new sln -n MySvc
dotnet new webapi -n MySvc.Api -f net9.0
dotnet new xunit -n MySvc.Tests -f net9.0
dotnet sln MySvc.sln add src/MySvc.Api/MySvc.Api.csproj tests/MySvc.Tests/MySvc.Tests.csproj
dotnet add tests/MySvc.Tests/MySvc.Tests.csproj reference src/MySvc.Api/MySvc.Api.csproj
```

**CI-style build & test (fast):**

```bash
dotnet restore
dotnet build -c Release --no-restore
dotnet test -c Release --no-build --logger "trx;LogFileName=test.trx"
```

**Publish for Linux container (single-file, trimmed):**

```bash
dotnet publish src/MySvc.Api/MySvc.Api.csproj -c Release -r linux-x64 \
  -p:PublishSingleFile=true -p:PublishTrimmed=true -p:TrimMode=partial -o out
```

---
