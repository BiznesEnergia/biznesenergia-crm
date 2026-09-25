# biznesenergia-crm

Warstwa biznesowa i integracyjna dla CRM Biznes Energia, oparta na [Twenty CRM](https://github.com/twentyhq/twenty).

Repozytorium źródłowe Twenty: [`BiznesEnergia/twenty`](https://github.com/BiznesEnergia/twenty).

Po podłączeniu źródło Twenty będzie dostępne w tym repozytorium jako submodule `packages/twenty`.

---

# Środowiska

| Środowisko | URL                             | Branch      | Przeznaczenie    |
| ---------- | ------------------------------- | ----------- | ---------------- |
| Local      | http://localhost:3000           | `feature/*` | Development      |
| Staging    | https://staging.crm.twojprad.pl | `develop`   | Testy            |
| Production | https://crm.twojprad.pl         | `main`      | Dane produkcyjne |

> Adresy środowisk staging i production są wartościami roboczymi i wymagają potwierdzenia przed wdrożeniem.

### Local

Każdy developer posiada własne środowisko lokalne.

### Staging

Środowisko testowe zbliżone do produkcji.

Nie korzysta z produkcyjnej bazy ani produkcyjnych sekretów.

### Production

Środowisko produkcyjne z rzeczywistymi danymi.

Deployment odbywa się przez CI/CD.

---

# Stack

## Twenty

- Node.js w wersji z `packages/twenty/.nvmrc`
- Yarn w wersji z `packageManager`
- Nx
- TypeScript
- PostgreSQL
- Redis
- Docker
- Docker Compose

## Biznes Energia

- Node.js
- TypeScript
- pnpm
- Caddy
- GitHub Actions
- GitHub Container Registry
- restic

Twenty pozostaje w swoim workspace Yarn/Nx. Własne pakiety Biznes Energia są oddzielnym workspace pnpm.

---

# Struktura docelowa

```text
biznesenergia-crm/
│
├── packages/
│   ├── twenty/
│   ├── biznesenergia-api/
│   ├── biznesenergia-integrations/
│   └── biznesenergia-shared/
│
├── infrastructure/
│   ├── docker/
│   │   ├── compose.local.yml
│   │   ├── compose.staging.yml
│   │   └── compose.production.yml
│   │
│   ├── caddy/
│   │   └── Caddyfile
│   │
│   └── scripts/
│       ├── deploy.sh
│       ├── backup.sh
│       └── restore.sh
│
├── database/
│   ├── migrations/
│   └── seeds/
│
├── docs/
│   ├── architecture.md
│   ├── development.md
│   ├── deployment.md
│   └── integrations/
│
├── tests/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── staging.yml
│       └── production.yml
│
├── .env.example
├── .gitignore
├── .gitmodules
├── package.json
├── pnpm-workspace.yaml
├── pnpm-lock.yaml
└── README.md
```

`packages/twenty` jest źródłem Twenty i nie należy do workspace pnpm. Pozostałe katalogi w `packages/` są oddzielnymi pakietami Biznes Energia.

# Stan projektu

Repozytorium znajduje się na etapie przygotowania fundamentu. Własne API, integracje i biblioteki będą rozwijane w `packages/biznesenergia-*`, a Twenty pozostanie w osobnym forku.

---

# Branching

```text
main
develop
feature/*
```

### `main`

Production.

### `develop`

Staging.

### `feature/*`

Nowe funkcje.

Przykłady:

```text
feature/ksef-integration
feature/customer-import
feature/email-sync
feature/erp-sync
```

---

# Workflow

Bezpośredni push do `main` i `develop` jest zabroniony.

```text
feature/*
   │
   ▼
Pull Request
   │
   ▼
CI
   │
   ▼
Code Review
   │
   ▼
develop
   │
   ▼
STAGING
   │
   ▼
TESTY
   │
   ▼
Pull Request → main
   │
   ▼
PRODUCTION
```

Każdy PR wymaga minimum jednego review.

---

# CI

Pull Request musi przejść:

```text
lint
typecheck
unit tests
integration tests
docker build
```

---

# Deployment

## Staging

Merge do `develop` uruchamia automatyczny deployment.

```text
develop
   ↓
CI
   ↓
Docker build
   ↓
GHCR
   ↓
Staging VPS
   ↓
Health check
```

## Production

Merge do `main` uruchamia przygotowanie deploymentu produkcyjnego.

```text
main
   ↓
CI
   ↓
Docker image
   ↓
Backup
   ↓
Production
   ↓
Health check
   ↓
Smoke tests
```

Production deployment wymaga ręcznego zatwierdzenia.

---

# Twenty

Twenty jest utrzymywane w osobnym forku:

- [`BiznesEnergia/twenty`](https://github.com/BiznesEnergia/twenty)
- fork upstream: [`twentyhq/twenty`](https://github.com/twentyhq/twenty)

`biznesenergia-crm` korzysta z forka jako submodule:

```text
BiznesEnergia/twenty
        │
        │ submodule: packages/twenty
        ▼
BiznesEnergia/biznesenergia-crm
```

Fork zawiera zmiany wymagane przez Biznes Energia, w tym customizacje UI, zmiany backendu i poprawki Twenty. Kod specyficzny dla Biznes Energia pozostaje poza submodulem i obejmuje:

- API,
- integracje,
- automatyzacje,
- worker,
- wspólne biblioteki,
- infrastrukturę.

Twenty ma własny workspace Yarn/Nx i pozostaje niezależny od workspace pnpm głównego repozytorium. Preferujemy rozszerzenia przez API, Apps i integracje; zmiany w core Twenty stosujemy tylko wtedy, gdy rozszerzenie systemu nie wystarcza.

# Development

## Wymagania

- WSL2 z Ubuntu
- Git
- Docker i Docker Compose
- Node.js w wersji wskazanej przez `packages/twenty/.nvmrc`
- Yarn i Corepack dla Twenty
- pnpm dla własnych pakietów

## Windows i długie ścieżki

Preferowanym środowiskiem dla Twenty jest WSL2. Przy bezpośrednim klonowaniu Twenty w Git for Windows mogą wystąpić błędy `Filename too long`. W takim przypadku włącz obsługę długich ścieżek:

```bash
git config --global core.longpaths true
```

## Pierwsze pobranie

Po podłączeniu submodule:

```bash
git clone --recurse-submodules https://github.com/BiznesEnergia/biznesenergia-crm.git
cd biznesenergia-crm
git submodule update --init --recursive
```

Jeżeli repozytorium zostało pobrane wcześniej:

```bash
git submodule update --init --recursive
```

## Uruchomienie Twenty

Twenty uruchamiamy z katalogu `packages/twenty`, zgodnie z instrukcją Twenty dotyczącą lokalnego środowiska:

- [Local setup](https://docs.twenty.com/developers/contribute/capabilities/local-setup#windows-wsl)
- [Self-hosting](https://docs.twenty.com/developers/self-host/capabilities/docker-compose)

W Twenty używamy wersji Node.js z `.nvmrc` oraz Yarn z `packageManager`. Nie należy uruchamiać Twenty z workspace pnpm głównego repozytorium.

## Własne pakiety

Własne pakiety Biznes Energia będą instalowane i uruchamiane z workspace pnpm w katalogu głównym repozytorium. `packages/twenty` jest celowo wykluczone z tego workspace:

```yaml
packages:
  - 'packages/*'
  - '!packages/twenty'
```

# Aktualizacja Twenty

W klonie forka `origin` wskazuje `BiznesEnergia/twenty`, a `upstream` wskazuje `twentyhq/twenty`. Aktualizacja odbywa się w repozytorium forka:

```bash
cd packages/twenty
git remote add upstream https://github.com/twentyhq/twenty.git
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
```

Następnie w repozytorium CRM aktualizujemy wskazanie submodule:

```bash
cd ../..
git submodule update --remote packages/twenty
```

PR ze zmianą wskaźnika submodule przechodzi przez review i testy Twenty przed wdrożeniem. W razie konfliktów najpierw aktualizujemy fork, a następnie ponownie wykonujemy aktualizację submodule.

---

# Integracje

Integracje znajdują się w:

```text
packages/biznesenergia-integrations/
```

Przykłady:

```text
packages/biznesenergia-integrations/
├── ksef/
├── erp/
└── email/
```

Integracje powinny być niezależnymi modułami i posiadać własne testy. Każda integracja komunikuje się z Twenty przez jego API lub warstwę integracyjną, zamiast bezpośrednio modyfikować kod submodule.

---

# Worker

Ciężkie operacje wykonywane są asynchronicznie:

```text
Twenty
   ↓
API
   ↓
Redis
   ↓
Worker
   ↓
Integracja
```

Webhooki i zadania integracyjne powinny być idempotentne.

---

# Secrets

Sekrety nie są przechowywane w Git.

Do repozytorium trafia:

```text
.env.example
```

Nie commitujemy:

```text
.env
.env.local
.env.staging
.env.production
*.key
*.pem
```

Sekrety Local, Staging i Production są rozdzielone.

---

# Database

PostgreSQL.

Zmiany schematu wykonujemy przez migracje:

```text
database/migrations/
```

Nie wykonujemy ręcznych zmian schematu produkcyjnej bazy.

---

# Backup

Backup produkcji obejmuje:

- PostgreSQL
- Twenty storage

Backup przechowywany jest poza głównym VPS.

Retencja:

```text
14 × daily
8 × weekly
12 × monthly
```

Przed deploymentem produkcyjnym wykonywany jest dodatkowy backup.

Restore backupu jest okresowo testowany.

---

# Infrastructure

Nie modyfikujemy ręcznie kodu produkcyjnego.

Nie:

```text
SSH → edycja kodu → restart
```

Tak:

```text
Git
 ↓
Pull Request
 ↓
CI
 ↓
Docker image
 ↓
Deployment
```

---

# Documentation

Dokumentacja znajduje się w:

```text
docs/
```

```text
docs/
├── architecture.md
├── development.md
├── deployment.md
└── integrations/
```

# Licencja

Twenty jest utrzymywane w osobnym forku i podlega własnym zasadom licencyjnym. Kod Twenty jest oznaczony licencjami AGPL oraz, w części plików, licencjami komercyjnymi Enterprise. Przy aktualizacji forka należy zachować pliki `LICENSE`, nagłówki licencyjne i wymagane warunki dystrybucji.

Przed użyciem komercyjnym, publikacją obrazu albo dystrybucją zmian należy zweryfikować obowiązujące licencje i zgodność z polityką Biznes Energia.

---

# Zasada

> Kod → Git
> Zmiany → Pull Request
> Testy → CI
> Staging → `develop`
> Production → `main`
> Deployment → CI/CD
> Dane → PostgreSQL / storage
> Backup → offsite
> Sekrety → poza Git
