```markdown
# Test Altyapısı Sabitleri ve Kurallar — kurallar.md

📜 Açıklık (Clarity)
- Amaç: Projede kullanılacak tüm test sabitlerini (env değişkenleri, portlar, zaman aşımı, dizinler, marker isimleri, CI artifact yolları, mock kuralları vb.) tek yerde standartlaştırmak.  
- Hedef: Testlerin tutarlılığı, tekrarlanabilirlik ve CI ile uyumluluğu sağlamak; geliştiricilerin "hangi sabiti kullanacağım?" sorusunu ortadan kaldırmak.

🔎 Çözümleme (Analysis)
- Kapsam: backend (Django + pytest), frontend (Next.js + Vitest/Jest), e2e (Playwright), contract tests (OpenAPI/Pact), infra test-stack (Docker Compose / testcontainers).
- Kurallar: environment-konfigürasyon, hizmet adları/portlar, test etiketleme (pytest marker), zaman aşımı/retry politikası, fixture/factory sabitleri, mocking tercihleri, CI output lokasyonları.
- Bağımlılıklar: PostgreSQL, Redis, MinIO (S3), OpenSearch (opsiyonel), Stripe (test), Mailhog, Celery (eager/worker).

📐 Düzenlilik (Order)
Aşağıda önce genel sabitler, sonra bileşen bazlı (DB, Redis, Celery, Storage, External APIs, Test araçları) sabitler yer alır. Her blokte ad ve önerilen değer/veri tipi verildi. Bu dosya source-of-truth olarak repo içinde tests/kurallar.md veya docs/testing/kurallar.md altında tutulmalı.

---

1) Genel kurallar
- DOSYA YERİ: tests/kurallar.md (bu dosya)
- ENV yükleme önceliği:
  - Local: .env.test (geliştirici)
  - CI: GitHub Actions secrets / environment
- Sürdürme: herhangi bir değer değiştiğinde ilgili ADR veya CHANGELOG güncellenecek.

2) Sabitler (anahtar = açıklama : örnek / öneri)

A. Environment / Docker / Test-stack
- TEST_COMPOSE_FILE : Docker Compose test dosyası : infra/test-compose/test-stack.yml
- TEST_STACK_SERVICES : postgres, redis, minio, mailhog, opensearch (opsiyonel)
- DOCKER_POSTGRES_IMAGE : Postgres image : postgres:15-alpine
- DOCKER_REDIS_IMAGE : Redis image : redis:7-alpine
- DOCKER_MINIO_IMAGE : MinIO image : minio/minio:RELEASE.2025-01-01T00-00-00Z (pinlenmiş)
- DOCKER_OPENSEARCH_IMAGE : opensearchproject/opensearch:2.8.0 (opsiyonel)

B. Database (Postgres)
- POSTGRES_HOST : postgres
- POSTGRES_PORT : 5432
- POSTGRES_USER : test
- POSTGRES_PASSWORD : test
- POSTGRES_DB : test_db
- TEST_DB_PREFIX : test_db_  (CI/xdist ile worker başına DB: test_db_worker1)
- DB_MIGRATE_ON_START : "true" (CI için seed+mig run flag)
- DB_MAX_CONNECTIONS_TEST : 20 (örnek limit)

C. Redis / Cache / Channels
- REDIS_HOST : redis
- REDIS_PORT : 6379
- REDIS_DB_CACHE : 0
- REDIS_DB_BROKER : 1
- REDIS_DB_CHANNELS : 2
- REDIS_URL_TEST : redis://redis:6379/1

D. Celery
- CELERY_ALWAYS_EAGER_TEST : true  (unit/integration default)
- CELERY_TASK_EAGER_PROPAGATES : true
- CELERY_BROKER_URL : redis://redis:6379/1
- CELERY_RESULT_BACKEND : cache+redis://redis:6379/0

E. Storage (S3 / MinIO / moto)
- STORAGE_PROVIDER_TEST : minio  # seçenek: moto|minio|local
- MINIO_ENDPOINT : minio:9000
- MINIO_ACCESS_KEY : minioadmin
- MINIO_SECRET_KEY : minioadmin
- S3_BUCKET_TEST : test-bucket
- SIGNED_UPLOAD_EXPIRE_SEC : 3600  # signed URL süre limiti testi

F. External APIs (mock sabitleri)
- STRIPE_API_BASE : https://api.stripe.com  (but use stripe-mock / mocked responses)
- STRIPE_TEST_KEY_PLACEHOLDER : sk_test_xxx
- STRIPE_WEBHOOK_SECRET_PLACEHOLDER : whsec_xxx
- SENDGRID_API_BASE : https://api.sendgrid.com  (mock)
- MAILHOG_SMTP_HOST : mailhog
- MAILHOG_SMTP_PORT : 1025
- MODERATION_API_URL : http://moderation-mock.local  (dev/test mock)

G. Search
- OPENSEARCH_HOST : opensearch:9200
- SEARCH_INDEX_PREFIX : test_idx_

H. Test davranış & zaman aşımı / retry
- PYTEST_UNIT_TIMEOUT_SEC : 5
- PYTEST_INTEGRATION_TIMEOUT_SEC : 30
- PYTEST_E2E_TIMEOUT_SEC : 120
- PLAYWRIGHT_DEFAULT_TIMEOUT_MS : 30000
- PLAYWRIGHT_ACTION_TIMEOUT_MS : 120000
- PYTEST_RERUN_FAILURES : 2   # pytest-rerunfailures
- PYTEST_XDIST_WORKERS_DEFAULT : max(1, cpu_count()-1)  (CI matrisi => explicit 4)

I. Pytest marker & naming
- PYTEST_MARKERS :
  - unit : hızlı, izole testler (no DB/IO ideally)
  - integration : DB/redis/minio veya celery vs gerçek infra kullanan testler
  - e2e : Playwright tests (sistem seviyesi)
  - contract : OpenAPI / Pact doğrulayan testler
  - flaky : karantinaya alınmış dalgalı testler
- TEST_FILE_PATTERNS :
  - backend unit : services/backend/**/tests/unit/test_*.py
  - backend integration : services/backend/**/tests/integration/test_*.py
  - frontend unit : services/frontend/src/**/__tests__/**/*.(test|spec).(ts|tsx|js|jsx)
  - e2e : tests/e2e/playwright/**/*.spec.ts

J. Factories & Fixtures
- FACTORY_SEED : 1337  # deterministic factory_boy seed
- FACTORY_LOCALE : tr_TR
- DEFAULT_FIXTURE_SCOPE : function
- SHARED_FIXTURE_MODULE : services/backend/conftest.py
- FACTORY_BOY_DEFAULT_STRATEGY : 'create' (integration), 'build' for unit when possible

K. Test data & limits
- SAMPLE_IMAGE_DIM : 1024x1024
- SAMPLE_IMAGE_FORMAT : JPEG
- THUMBNAIL_SIZES : [200, 400]
- MAX_TEST_LISTINGS_PER_RUN : 50
- DEFAULT_PAGINATION_SIZE_TEST : 10

L. Frontend test sabitleri
- VITEST_TIMEOUT_MS : 10000
- RTL_ASYNC_UTILS_TIMEOUT_MS : 4000
- MSW_INTEGRATION : true  # frontend unit/integration mock server
- STORYBOOK_TEST_TOKEN : storybook-test

M. Playwright / E2E
- PLAYWRIGHT_PROJECTS_SMOKE : chromium
- PLAYWRIGHT_TRACE_ON_FAIL : on-first-retry
- PLAYWRIGHT_SCREENSHOT_ON_FAIL : true
- E2E_TEST_USER :
  - EMAIL : e2e+user@example.test
  - PASSWORD : Password123!
  - STRIPE_TEST_CARD : 4242 4242 4242 4242 (test card)
- E2E_SEED_DB_SNAPSHOT : tests/fixtures/e2e_seed.dump  (staging snapshot optional)

N. Contract tests
- OPENAPI_SPEC_PATH : docs/API/openapi.yaml
- PACT_BROKER_URL : (CI secret) or file-based contracts in tests/contracts/
- CONTRACT_VERIFY_TIMEOUT : 30

O. CI / Artifacts / Reports
- CI_JUNIT_ARTIFACT_BACKEND : junit-backend.xml
- CI_JUNIT_ARTIFACT_FRONTEND : junit-frontend.xml
- CI_COVERAGE_REPORT : coverage.xml
- CI_PLAYWRIGHT_TRACES_DIR : artifacts/playwright/traces/
- CI_TEST_LOG_DIR : artifacts/test-logs/
- CI_CACHE_KEY_PIP : pip-cache-${{ hashFiles('**/poetry.lock') }} (example, define in workflow)
- CI_REQUIRED_JOBS_TO_MERGE : lint, unit-tests-backend, unit-tests-frontend, critical-integration, contract-validate

P. Coverage & kalite metrikleri
- COVERAGE_MINIMUM_OVERALL : 80  # yüzde
- COVERAGE_MINIMUM_CRITICAL_MODULES : 90  # kritik path'ler
- LINT_RULES : ruff/flake8 + black/isort enforced by pre-commit

Q. Flaky tests yönetimi
- FLAKY_MAX_RETRIES : 2
- FLAKY_MARKER : @pytest.mark.flaky
- FLAKY_QUARANTINE_LIST : tests/quarantine.txt (CI upload)
- FLUCTUATION_ALERT_THRESHOLD : 3 fails/week -> create incident

R. Security / Secrets in tests
- NEVER commit real API keys: use placeholders in repo and CI secrets
- USE ephemeral test keys for external providers (Stripe test keys)
- SCA_SCAN_ON_CI : true (Snyk / Dependabot / GitHub SCA)

S. Naming konvansiyonları
- Test sınıfları: Test<ClassName>
- Test fonksiyonları: test_<unit>_<behavior>_<expected>
- Fixtures: fixture_<scope>_<what>
- Factories: <ModelName>Factory

T. Test komutları (kısa)
- backend unit: poetry run pytest -m "unit" --maxfail=1 -q
- backend integration: ./infra/test-compose/up.sh && poetry run pytest -m "integration" tests/integration/ --maxfail=1
- frontend unit: pnpm test
- e2e (local): ./infra/test-compose/up-e2e.sh && npx playwright test --project=chromium
- full: ./infra/test-compose/up.sh && pytest

U. Testcontainers / lokal CI
- TESTCONTAINERS_POSTGRES_IMAGE : postgres:15-alpine
- TESTCONTAINERS_REDIS_IMAGE : redis:7-alpine

V. Log & artifact retention
- CI_ARTIFACT_RETENTION_DAYS : 30
- PLAYWRIGHT_MAX_TRACE_SIZE_MB : 50  # trace sıkıştırma/trim kuralları

W. Dosya yolları / dizinler
- BACKEND_TESTS_DIR : services/backend/tests/
- FRONTEND_TESTS_DIR : services/frontend/src/tests/
- E2E_TESTS_DIR : tests/e2e/playwright/
- CONTRACTS_DIR : tests/contracts/
- TEST_FIXTURES_DIR : tests/fixtures/

---

3) Mocking & Enjeksiyon tercihleri (kurallar)
- Unit testlerde harici HTTP çağrıları asla gerçek ortamda yapılmaz; kullan (sırasıyla):
  1. requests -> responses
  2. httpx -> respx
  3. async httpx -> respx + pytest-asyncio
- Stripe/Payment: stripe-mock veya responses/respx ile tüm webhook'lar simüle edilecek.
- S3: unit => moto; integration => MinIO container.
- Mail: unit => django MailBackend / in-memory; integration => mailhog container.
- Moderation/3rd-party: VCRpy sadece legacy kayıtlar için; tercih respx+fixtures.
- Channels/WebSocket: channels.testing.WebsocketCommunicator ve pytest-asyncio kullan.

4) CI Entegrasyon & Gate kuralları
- PR merge koşulu: lint + unit-tests (backend+frontend) + contract-validate must pass.
- integration tests or e2e smoke may be optional in PR but required in staging pipeline.
- Artifacts to upload on failure: junit xml, coverage report, playwright trace, failing test logs.
- Contract validation job should fail build on mismatch.

5) Dokümantasyon & Sürdürme
- Bu dosya değiştiğinde tests/README.md içinde "özeti" güncelleyin.
- Yeni test marker eklenirse tests/pytest.ini ve docs/TESTING.md güncellenecek.
- Her major değişiklik ADR ile kayıt altına alınmalı (docs/ADR/).

✅ Eksiksizlik (Completeness)
- Bu dosya:
  - test infra için kullanılan tüm önemli sabitleri kapsar (env, docker, db, redis, s3, timeouts, ci paths, markers).
  - mocking ve test davranışı için açık tercihleri belirtir.
  - CI entegrasyonu ve merge gate kurallarını içerir.
  - Dosya/dizin konvansiyonları ve komut örnekleri sağlanmıştır.
- Eksik bir gereksinim görürseniz (ör. özel ödeme sağlayıcı, farklı mail provider veya daha sıkı coverage hedefi) lütfen belirtin; ilgili sabitleri ekleyip ADR önerisi hazırlayayım.

Örnek .env.test (şablon)
```env
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_USER=test
POSTGRES_PASSWORD=test
POSTGRES_DB=test_db
REDIS_URL=redis://redis:6379/1
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
CELERY_ALWAYS_EAGER=true
DJANGO_SETTINGS_MODULE=project.settings.test
```

Checklist — hızlı doğrulama
- [ ] Bu dosya repoya ekli (tests/kurallar.md)
- [ ] pytest.ini / conftest.py sabitlerle uyumlu
- [ ] infra/test-compose/test-stack.yml sabitleri referans alıyor
- [ ] CI workflow sabitleri (artifact isimleri, job adları) eşleştirildi
- [ ] Dokümantasyon (tests/README.md) güncellendi

```