# biznesenergia-crm

Warstwa biznesowa i integracyjna dla CRM Biznes Energia, oparta na [Twenty CRM].

Repozytorium Twenty:

`BiznesEnergia/twenty`

---

# Środowiska

| Środowisko | URL                             | Branch      | Przeznaczenie    |
| ---------- | ------------------------------- | ----------- | ---------------- |
| Local      | http://localhost:3000           | `feature/*` | Development      |
| Staging    | https://staging.crm.twojprad.pl | `develop`   | Testy            |
| Production | https://crm.twojprad.pl         | `main`      | Dane produkcyjne |

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

- Twenty CRM
- PostgreSQL
- Redis
- Node.js
- TypeScript
- pnpm
- Docker
- Docker Compose
- Caddy
- GitHub Actions
- GitHub Container Registry
- restic

---

# Struktura

```text
biznesenergia-crm/
│
├── apps/
│   ├── integrations/
│   └── api/
│
├── packages/
│   └── shared/
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
├── package.json
├── pnpm-lock.yaml
└── README.md
```

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

Twenty jest utrzymywane w osobnym repozytorium:

```text
BiznesEnergia/twenty
```

Repozytorium `BiznesEnergia/twenty` zawiera:

- fork Twenty,
- zmiany core,
- customizacje UI,
- zmiany backendu,
- poprawki wymagające modyfikacji Twenty.

`biznesenergia-crm` zawiera kod specyficzny dla Biznes Energia:

- integracje,
- API,
- automatyzacje,
- worker,
- wspólne biblioteki,
- infrastrukturę.

Preferowany model:

```text
BiznesEnergia/twenty
        │
        │ Docker image
        ▼
BiznesEnergia/biznesenergia-crm
        │
        ▼
   Staging / Production
```

Twenty samo wspiera self-hosting przez Docker Compose oraz rozszerzanie CRM przez Apps, API i kod, dlatego zmiany core powinny być stosowane tylko wtedy, gdy rozszerzenie systemu nie wystarcza.

---

# Integracje

Integracje znajdują się w:

```text
apps/integrations/
```

Przykłady:

```text
apps/integrations/
├── ksef/
├── erp/
└── email/
```

Integracje powinny być niezależnymi modułami i posiadać własne testy.

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
