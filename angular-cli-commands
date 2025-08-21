# Angular CLI Commands 

## Install & Workspace

```bash
npm i -g @angular/cli             # Install CLI
ng version                        # Show versions
ng new my-app --standalone --style=scss --routing --ssr
ng new my-workspace --create-application=false
```

## Serve & Build

```bash
ng serve                          # Dev server at http://localhost:4200
ng serve -o --port 4300 -c development
ng build                          # Production build uses default prod config
ng build -c production --output-path=dist/app --base-href=/subpath/
ng build --watch                  # Rebuild on file change
```

## Add & Update

```bash
ng add @angular/material          # Install & configure a library
ng add @angular/pwa               # Turn app into a PWA
ng update @angular/core @angular/cli  # Update Angular & CLI
```

## Generate (aliases: `g`, `s`, `c`, etc.)

```bash
ng g application app2                         # New app in workspace
ng g library shared-ui                        # New library

# Standalone components (Angular 15+)
ng g c features/users --standalone --change-detection OnPush --skip-tests
ng g c shared/ui/button --inline-style --inline-template

# Other schematics
ng g s core/api                               # Service (providedIn: 'root' by default)
ng g d shared/directives/auto-focus           # Directive
ng g p shared/pipes/currency                  # Pipe
ng g i models/user                            # Interface
ng g e models/role                            # Enum
ng g guard features/auth/auth --functional    # Route guard (functional)
ng g interceptor core/http/auth               # HTTP interceptor
ng g resolver features/users/user             # Route resolver

# Router helpers (standalone routes)
ng g @angular/router:routes users --standalone --flat
```

## Testing & Linting

```bash
ng test                           # Unit tests (watch by default)
ng test --code-coverage           # Generate coverage
ng lint                           # With @angular-eslint configured
```

## i18n

```bash
ng extract-i18n                   # Generate messages.xlf
ng extract-i18n --output-path=src/locale --format=xlf
```

## Deploy (requires a deploy builder)

```bash
ng add angular-cli-ghpages
ng deploy                         # e.g., deploy to GitHub Pages
```

## Config & Schematics

```bash
ng config                         # Read/write angular.json
ng config schematics.@schematics/angular:component.style scss
ng config cli.cache.enabled true
```

## Cache, Analytics, Help

```bash
ng cache clean                    # Clear build cache
ng analytics                      # Show analytics status
ng analytics enable|disable
ng help                           # Global help
ng help build|serve|generate      # Command-specific help
ng doc component                  # Open docs for a keyword
```

## Common Flags (mix & match)

* `-c, --configuration <name>` – choose a named config (e.g., `production`, `development`)
* `--project <name>` – target a specific project in a workspace
* `--dry-run` – show what will happen without changing files
* `--force` – bypass certain safety checks
* `--flat` – place files without creating a subfolder
* `--path <dir>` – output to a specific path
* `--skip-tests` – don’t create spec files
* `--standalone` – generate standalone components/routes

---

Want this as a printable one-pager PDF or tailored for Angular v20 options (signals/standalone by default)? I can convert it instantly.
