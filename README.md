# biznesenergia-crm

Customizowany [Twenty CRM](https://github.com/twentyhq/twenty) dla Biznes Energia.

---

# Środowiska

| Środowisko | URL                             | Branch      | Przeznaczenie      |
| ---------- | ------------------------------- | ----------- | ------------------ |
| Local      | http://localhost:3000           | `feature/*` | Development        |
| Staging    | https://staging.crm.twojprad.pl | `develop`   | Testy i integracje |
| Production | https://crm.twojprad.pl         | `main`      | Dane produkcyjne   |

### Local

Każdy developer posiada własne lokalne środowisko:

- Twenty
- PostgreSQL
- Redis
- API
- integracje

Lokalna baza danych nie jest współdzielona pomiędzy developerami.

### Staging

Środowisko testowe możliwie zbliżone do produkcji.

Staging nie korzysta z produkcyjnej bazy danych ani produkcyjnych sekretów.

### Production

Środowisko produkcyjne zawierające rzeczywiste dane biznesowe.

Zmiany w produkcji wykonywane są wyłącznie przez proces CI/CD.

---

# Stack technologiczny

## Core

- Twenty CRM
- PostgreSQL
- Redis
- Docker
- Docker Compose
- Caddy

## Development

- Node.js
- TypeScript
- pnpm

## CI/CD

- GitHub
- GitHub Actions
- GitHub Container Registry (GHCR)

## Backup

- PostgreSQL dump
- Twenty storage
- restic
- offsite storage

---

# Docelowa struktura

```text
biznesenergia-crm/
│
├── apps/
│   ├── integrations/
│   └── api/
│
├── packages/
│   ├── shared/
│   └── twenty-client/
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

# Branching strategy

Główne branche:

```text
main
develop
feature/*
```

## `main`

Wersja produkcyjna.

```text
main = production
```

Bez bezpośredniego pushowania.

## `develop`

Aktualna wersja stagingowa.

```text
develop = staging
```

## `feature/*`

Branch roboczy dla konkretnej funkcji.

Przykłady:

```text
feature/ksef-integration
feature/customer-import
feature/new-crm-fields
feature/email-sync
feature/erp-sync
```

Dodatkowo:

```text
fix/*
hotfix/*
chore/*
```

dla poprawek i zadań technicznych.

---

# Zasada pracy developera

Bezpośredni push do:

```text
main
develop
```

jest zabroniony.

Proces:

```text
feature/ksef-integration
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

---

# Pull Request

Każda zmiana przechodzi przez Pull Request.

PR powinien zawierać:

- opis zmiany
- powód zmiany
- sposób testowania
- informację o migracjach DB
- informację o zmianach konfiguracji
- informację o zmianach integracji

Przykład:

```text
## Co zostało zmienione?

Dodano synchronizację klientów z ERP.

## Dlaczego?

Potrzebujemy automatycznej synchronizacji danych klientów.

## Testy

- [x] Unit tests
- [x] Integration tests
- [x] Test na staging

## Database

- [ ] Brak zmian
- [ ] Dodano migrację

## Configuration

- [ ] Brak zmian
- [ ] Dodano nowe zmienne środowiskowe
```

---

# Code Review

Minimalnie **1 osoba musi zaakceptować PR** przed merge.

Developer nie powinien samodzielnie zatwierdzać i mergować własnej zmiany.

GitHub pozwala wymusić review oraz wymagane status checks na chronionych branchach.

---

# CI

Każdy Pull Request powinien przejść:

```text
lint
typecheck
unit tests
integration tests
docker build
```

Docelowo:

```text
PR
 │
 ├── lint              ✅
 ├── typecheck         ✅
 ├── unit tests        ✅
 ├── integration tests ✅
 └── docker build      ✅
```

PR nie może zostać zmergowany, jeżeli wymagane checki nie przejdą.

---

# Deployment

## Staging

Merge do:

```text
develop
```

uruchamia automatyczny deployment:

```text
develop
   │
   ▼
GitHub Actions
   │
   ▼
Docker build
   │
   ▼
GHCR
   │
   ▼
Staging VPS
   │
   ▼
Health check
```

## Production

Merge do:

```text
main
```

przygotowuje deployment produkcyjny.

Proces:

```text
main
 │
 ▼
CI
 │
 ▼
Docker image
 │
 ▼
Backup
 │
 ▼
Production deployment
 │
 ▼
Health check
 │
 ▼
Smoke tests
```

Production deployment powinien być początkowo ręcznie zatwierdzany.

---

# Docker images

Obrazy produkcyjne nie powinny być uruchamiane wyłącznie przez tag:

```text
latest
```

Każdy deployment powinien wskazywać konkretną wersję/commit:

```text
ghcr.io/OWNER/biznesenergia-crm:<commit-sha>
```

Dzięki temu możliwy jest jednoznaczny rollback.

---

# Secrets

Sekrety nie mogą znajdować się w repozytorium.

Nigdy nie commitujemy:

```text
.env
.env.local
.env.staging
.env.production
*.key
*.pem
```

Do repozytorium trafia wyłącznie:

```text
.env.example
```

Przykładowe sekrety:

```text
DATABASE_URL
REDIS_URL
ENCRYPTION_KEY
APP_SECRET
KSEF_TOKEN
ERP_API_KEY
GOOGLE_CLIENT_SECRET
```

Sekrety Local, Staging i Production muszą być rozdzielone.

---

# Twenty CRM

Twenty jest głównym systemem CRM.

Preferowane rozszerzanie Twenty:

```text
API
Webhooks
Apps
Integracje
Custom code
```

Modyfikowanie core Twenty powinno być ostatecznością.

Konfiguracja CRM powinna być dokumentowana w:

```text
docs/twenty/
```

np.:

```text
docs/twenty/
├── objects.md
├── fields.md
├── pipelines.md
├── permissions.md
├── workflows.md
└── configuration.md
```

Twenty domyślnie może przechowywać część zmiennych konfiguracyjnych w bazie danych, dlatego konfigurację wykonywaną przez panel administracyjny trzeba traktować jako część konfiguracji systemu, a nie zakładać, że wszystko będzie widoczne w Git.

---

# Integracje

Integracje znajdują się w:

```text
apps/integrations/
```

Każda integracja powinna być odseparowanym modułem.

Przykład:

```text
apps/integrations/
│
├── ksef/
│   ├── client.ts
│   ├── mapper.ts
│   ├── sync.ts
│   ├── webhook.ts
│   └── tests/
│
├── erp/
│   ├── client.ts
│   ├── mapper.ts
│   ├── sync.ts
│   └── tests/
│
└── email/
    ├── client.ts
    ├── sync.ts
    └── tests/
```

---

# Webhooki i eventy

Każdy webhook musi być odporny na wielokrotne dostarczenie tego samego eventu.

Przykład:

```text
event_id
    │
    ▼
sprawdzenie czy przetworzony
    │
 ┌──┴──┐
 │     │
 TAK   NIE
 │     │
stop   process
       │
       ▼
     save event
```

Operacje integracyjne powinny być idempotentne.

---

# Worker

Ciężkie operacje nie powinny blokować API.

Docelowy przepływ:

```text
Twenty
   │
   ▼
API
   │
   ▼
Redis Queue
   │
   ▼
Worker
   │
   ├── ERP
   ├── KSeF
   ├── Email
   └── inne integracje
```

---

# Database

PostgreSQL jest główną bazą danych.

Zmiany schematu muszą być wykonywane przez migracje.

Migracje:

```text
database/migrations/
```

Nie wykonujemy ręcznych zmian schematu produkcyjnej bazy bez zapisania ich w procesie migracji.

---

# Backup

Backup produkcji obejmuje:

```text
PostgreSQL
Twenty storage
```

Backup jest przechowywany poza głównym VPS.

Minimalna retencja:

```text
14 × daily
8 × weekly
12 × monthly
```

Przed deploymentem produkcyjnym wykonywany jest dodatkowy backup.

---

# Restore

Backup musi być okresowo testowany.

Minimum:

```text
backup
   │
   ▼
restore
   │
   ▼
test PostgreSQL
   │
   ▼
test Twenty
```

Test restore powinien być wykonywany cyklicznie.

---

# Monitoring

Monitorowane powinny być minimum:

```text
CPU
RAM
Disk
SSL
HTTP
PostgreSQL
Redis
Backup
Twenty
API
Worker
Integrations
```

---

# Logging

Logi powinny zawierać identyfikatory umożliwiające śledzenie operacji:

```text
request_id
job_id
event_id
integration
status
duration
```

Nigdy nie logujemy:

```text
password
API keys
tokens
OAuth secrets
ENCRYPTION_KEY
danych uwierzytelniających
```

---

# Zasada infrastruktury

Nie wykonujemy ręcznych zmian produkcyjnego kodu.

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

# Zasada danych

Kod i infrastruktura są wersjonowane w Git.

Dane biznesowe nie są wersjonowane w Git.

```text
GIT
├── kod
├── konfiguracja infrastruktury
├── Docker
├── CI/CD
├── migracje
└── dokumentacja

PRODUCTION
├── dane CRM
├── PostgreSQL
├── pliki
└── sekrety
```

---

# Dokumentacja

Dokumentacja projektu znajduje się w:

```text
docs/
```

Minimalny zestaw:

```text
docs/
├── architecture.md
├── development.md
├── deployment.md
├── backup.md
├── security.md
└── integrations/
```

Każda istotna integracja powinna posiadać własną dokumentację.

---

# Zasada ogólna

> Kod → Git
> Review → Pull Request
> Testy → CI
> Staging → `develop`
> Production → `main`
> Deployment → CI/CD
> Dane → PostgreSQL / storage
> Backup → offsite
> Sekrety → secret management
