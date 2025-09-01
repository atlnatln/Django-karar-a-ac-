```markdown
# Test Altyapısı — Ev Hayvanları Pazaryeri (MVP → Ölçeklenebilir)

📜 Açıklık (Clarity)
- Problem: Monorepo (Django backend + Next.js frontend) için güvenilir, hızlı, sürdürülebilir ve tekrarlanabilir bir test altyapısı kurmak. Amaç: hataları erkenden yakalamak, regresyonları önlemek ve production riskini azaltmak.
- Hedefler:
  - Test seviyeleri: unit, integration, contract, e2e (Playwright).
  - TDD destekli geliştirme akışı: önce test, sonra implementasyon.
  - CI/CD entegrasyonu: lint → unit → integration/contract → build → e2e (staging).
  - Ölçeklenebilir, modüler test düzeni ve external servislerin güvenli mocklanması.

🔎 Çözümleme (Analysis)
- Test seviyeleri ve sorumlulukları
  - Unit Tests (Hızlı, izole): modellerin business logic'i, serializer/validator, yardımcı fonksiyonlar, util'ler.
  - Integration Tests (DB+cache+celery vb.): Django ORM, view/endpoint (DRF) behavior, template rendering, Channels consumers (WebSocket), Celery tasks (async flows), storage pipeline (S3 sign flow with moto).
  - Contract Tests (API sözleşmeleri): OpenAPI / Pact ile backend ↔ frontend sözleşme doğrulaması; contract-first akış destekleri.
  - E2E Tests (Kullanıcı senaryoları): Playwright ile kullanıcı akışı (signup → create listing with images → buy package → message seller).
  - Static & Security: linters, bandit/Snyk, type checks (mypy/pyright), a11y (axe) ve dependency scanning.
- Data & State Management
  - Test DB: pytest-django + transactional DB (Postgres in Docker) veya SQLite for super-fast unit-only flows.
  - Fixtures/factories: factory_boy + faker kullan (deterministic seed). Test veri izolasyonu için pytest fixtures scope kontrolü.
  - External services: Redis, MinIO (S3), Elasticsearch/OpenSearch, Stripe, SendGrid; CI için ephemeral docker-compose veya test doubles.
- Mocking ve bağımlılık enjeksiyonu
  - HTTP: responses, httpx_mock veya vcrpy (3rd-party API'ler için deterministik kayıt).
  - AWS S3: moto veya MinIO local integration.
  - Stripe: stripe-mock (lokalde) veya responses ile webhook simülasyonu.
  - Mail: local SMTP dev server veya mailhog; alternatif olarak django-mailer ile test backend.
  - Celery: celery-testing (eager mode) ve pytest fixtures ile task runner kontrolü.
  - Channels: channels.testing.WebsocketCommunicator + pytest-asyncio.
  - Frontend: MSW (Development) / mocked fetch in unit tests; Pact/OpenAPI contract tests for api compatibility.
- Test performansı ve güvenilirlik
  - Unit tests < 200ms/test ortalama.
  - Integration tests izolasyonlu, paralel çalıştırılabilir (pytest-xdist).
  - Flaky test yönetimi: retry (pytest-rerunfailures) ve `@pytest.mark.flaky` ile karantinaya alma.
  - Test paralelliği ve DB: transactional test pattern + database per worker (pytest-django + django-db-blocker patterns veya testcontainers).

📐 Düzenlilik (Order) — Uygulanacak adımlar (basitten karmaşığa)
1. Temel kurulum (Sprint 0)
   - tools: pytest, pytest-django, factory_boy, pytest-asyncio, pytest-cov, responses, moto, pytest-mock.
   - frontend tools: vitest/jest + React Testing Library, MSW, Playwright.
   - CI: GitHub Actions yapılandırması (workflow: ci.yml).
   - pre-commit: black, isort, flake8, ruff; test çalıştırma kısa komutları (Makefile / scripts).
2. Unit test kapsama alanı (her app — sprint 1)
   - Backend: services/backend/apps/*/tests/unit/  
     - users: password policies, serializer validation, permissions
     - listings: price calculation, publish rules, slug generation
     - payments: local helper functions, billing math
     - messaging: Conversation helpers, unread counters
     - media: thumbnail generation helpers
   - Frontend: components ve hooks için unit tests (RTL/vitest)
   - Kural: her PR için yeni kod +20% test örnekleri; hedef minimum coverage %80 modül bazında.
3. Integration tests (sprint 2)
   - DB-backed endpoint tests: services/backend/tests/integration/
     - Model ↔ Serializer ↔ View tam akış testleri (pytest-django client).
     - Celery task integration (eager veya test worker).
     - Channels consumer tests (WebSocket lifecycle).
     - Storage: signed upload flow with moto or MinIO test instance.
   - Contract tests: OpenAPI-validate responses against spec; consumer-driven Pact for critical flows (payments, listings).
4. Contract & Security tests (sprint 3)
   - OpenAPI schema validation job in CI: ensure responses follow spec.
   - Security scans: bandit/Snyk container; dependency pinning check.
5. E2E tests & CI integration (sprint 4)
   - Playwright tests located at /tests/e2e/playwright/
   - E2E runs on staging (not on every PR); smoke e2e runs in PR for critical paths (signup, create listing).
   - Use test user accounts, seeded fixture DB snapshot for deterministic flows.
6. Observability & Flaky mitigation (ongoing)
   - Artifacts: junit XML, coverage reports, Playwright traces/screenshots uploaded as artifacts.
   - Flaky test detection, quarantine and incident review process.

Dosya / Dizin önerisi (monorepo örneği)
```
repo-root/
├─ services/
│  ├─ backend/
│  │  ├─ apps/
│  │  │  ├─ users/tests/unit/
│  │  │  ├─ users/tests/integration/
│  │  │  ├─ listings/tests/unit/
│  │  │  └─ ...
│  │  ├─ tests/
│  │  │  ├─ unit/                # cross-app utilities
│  │  │  ├─ integration/         # DB+redis+celery tests
│  │  │  └─ contract/            # openapi/pact tests
│  │  ├─ pytest.ini
│  │  ├─ conftest.py             # shared fixtures (db, client, factories)
│  │  └─ tests_requirements.txt
│  └─ frontend/
│     ├─ src/
│     │  └─ tests/               # unit+integration (RTL / vitest)
│     └─ tests/e2e/              # Playwright (if frontend-focused)
├─ tests/
│  ├─ e2e/playwright/            # monorepo-level E2E
│  └─ contracts/                 # consumer/provider artifacts
├─ infra/
│  └─ test-compose/             # docker-compose for CI services (pg, redis, minio)
└─ .github/workflows/ci.yml
```

Her modül için test kapsamı önerisi (kısa)
- users
  - unit: validators, profile business rules, token rotation logic
  - integration: registration flow, email verification, password reset
  - e2e: signup+login+profile edit
- listings
  - unit: price logic, slug, tag parsing
  - integration: create/list/update/delete endpoints with media signed upload
  - e2e: create listing with images (upload → webhook → DB)
- payments
  - unit: invoice calculations, coupon application
  - integration: Stripe webhook handling (signature verification), DB side-effects
  - contract: provider (backend) exposes OpenAPI for checkout endpoints
  - e2e: buy package (using Stripe test card flows on staging)
- messaging
  - unit: model helpers, unread counters
  - integration: WebSocket consumer message send/receive, persisted messages
  - e2e: message exchange between two users (connect, send, receive)
- media
  - unit: image processing helpers
  - integration: signed upload + post-processing task (celery) with moto/MinIO
- moderation
  - unit: rule evaluation
  - integration: async moderation task invoking 3rd-party API (mocked)
- search
  - integration: index syncing behavior with Postgres FTS or OpenSearch test cluster
- notifications
  - unit: templating, payload creation
  - integration: delivery via mail provider (mocked) + in-app notification DB entries

Mocking & Test Doubles — Pratik kurallar
- External 3rd-party HTTP çağrılarını doğrudan çalıştırmayın; kullan:
  - responses / httpx_mock (unit/integration)
  - VCRpy only for non-deterministic legacy APIs (careful)
- AWS: moto veya MinIO; seçim: moto lightweight unit, MinIO daha gerçekçi integration.
- Stripe: kullanıyorsanız stripe-mock local runner + verify webhook signatures in integration.
- Replace long-lived credentials with CI secrets and ephemeral test keys.

CI/CD Entegrasyonu (GitHub Actions — öneri akış)
- Workflow: .github/workflows/ci.yml
  - jobs:
    1. lint (black/isort/ruff/flake8)
    2. unit-tests-backend (pytest -m "unit") — cache pip/venv, coverage upload
    3. unit-tests-frontend (vitest/jest)
    4. integration-tests (requires docker-compose services up: pg, redis, minio) — run in parallel matrix (postgres versions)
    5. contract-validation (openapi validator / pact verifications)
    6. e2e-smoke (Playwright quick smoke) — optional in PR; full e2e on staging deployment job
  - Artifacts: junit.xml, coverage.xml, playwright traces/screenshots, pact contracts.
  - Gate: merge blocked unless lint + unit + critical integration + contract pass. E2E on staging before release.

TDD & Geliştirici Pratikleri
- PR workflow: developer yazmadan önce test tasarlayıp failing test üzerinden ilerlesin.
- Small commits: "red → green → refactor".
- Conftest fixtures'ı ile shared factory ve override pattern kullanın (ör. override_settings, monkeypatch).
- Service classes & factories kullanarak DI-benzeri yapı: gerçek external client yerine interface/adapter pattern, tests override ile fake adapter injekte edin.

Test Kalitesi & Metrikler
- Hedefler:
  - Unit coverage: ≥ 80% (kritik modüller için ≥ 90%).
  - Integration coverage: kritik paths 100% (webhook processing, payments).
  - Test süreleri: PR için toplam unit+lint < 10 dakika (parallel jobs).
  - Flaky rate: <%1; tespit edilen flaky tests quarantine listesine alınsın.
- İzleme:
  - Test coverage badge, weekly flaky report, and CI time trends.

✅ Eksiksizlik (Completeness)
- Idea: Test stratejisi, seviyeler, araçlar ve dizin yapısı tanımlandı.
- Test: Her seviyede hangi testler yazılacağı, hangi kütüphanelerin kullanılacağı ve mocking stratejileri açıklandı.
- Dokümantasyon: Bu dosya repository içinde tests/README.md veya docs/TESTING.md olarak yer almalı; CI workflow örnekleri, nasıl lokal çalıştırılacağı, Docker Compose test stack komutları ve test veri seed script'leri dahil edilmeli.

Checklist (quick)
- [ ] pytest + pytest-django kurulumu & konfigürasyonu (pytest.ini)
- [ ] conftest.py: DB, client, factories, async fixtures
- [ ] factory_boy + deterministic seed
- [ ] MSW + vitest/jest frontend unit tests
- [ ] Playwright e2e testleri + test senaryolar
- [ ] OpenAPI / Pact contract validation job
- [ ] docker-compose test stack (pg, redis, minio, elasticsearch)
- [ ] CI pipeline: lint → unit → integration → contract → e2e(staging)
- [ ] Test dokümantasyonu (how-to run locally, debug tips)
- [ ] Pre-commit hooks ve test run shortcuts (Makefile)

Hızlı komut örnekleri
- Backend unit: poetry run pytest -m "unit" --maxfail=1 -q
- Backend integration: poetry run pytest tests/integration/ --durations=10
- Full tests (local docker services up): ./infra/test-compose/up.sh && pytest
- Frontend unit: pnpm test
- Playwright e2e (local staging): npx playwright test --project=chromium

Son not — Descartes ilkeleriyle bağlama
- Açıklık: test hedefleri ve kapsam net tanımlandı.
- Çözümleme: modül bazlı ayrıştırma, bağımlılıkların mocklanması ve veri akışı gösterildi.
- Düzenlilik: basitten karmaşığa adım adım uygulanacak plan verildi.
- Eksiksizlik: idea + test + dokümantasyon + CI entegrasyonu kapsandı.

İsterseniz bir sonraki adım olarak:
- Bu stratejiye uygun bir başlangıç PR'sı (pytest config, conftest, örnek unit+integration test, GitHub Actions ci.yml skeleton) hazırlayıp açabilirim.
```