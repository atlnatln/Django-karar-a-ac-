# Ev Hayvanları Satış & Sahiplendirme Pazaryeri — Proje Altyapısı (MVP → Ölçeklenebilir)

📜 Açıklık (Clarity)
- Problem: Orta büyüklükte ekip için tek DB + modüler monolitik mimari ile; kullanıcıların kayıt olup ilan verebildiği, ilan paketleriyle ücret ödeyebildiği, sahipsiz/ücretsiz sahiplendirme ilanlarının olduğu, ilan verenlerle alıcıların Facebook-benzeri mesajlaşma ile haberleşebildiği; ana sayfa SEO uyumlu, uygulama mobil deneyimli bir web pazaryeri istiyorsunuz.
- Hedef: Sektör standardına uygun, güvenli, testlenebilir, MVP kısa sürede çıkarılabilecek; sonrasında yatay parçalanmaya izin veren monolitik-modüler bir altyapı tanımlamak.

🔎 Çözümleme (Analysis)
- Ana parçalar (modüller): Auth, Listings, Payments, Messaging (realtime), Media, Moderation, Search, Notifications, Admin/CMS, Analytics.
- Veri akışı (yüksek seviye):
  1. Kullanıcı kayıt → kimlik doğrulama (Django Auth + DRF SimpleJWT rotated refresh tokens via httpOnly cookie).
  2. İlan oluşturma → medya S3'e yükleme (signed upload) → DB kayıt + search index güncelleme.
  3. Paket satın alma → Stripe ödeme → webhook ile ödeme onayı → ilan hakkı aktif.
  4. Mesajlaşma → WebSocket (Channels + Redis) veya fallback polling → mesajlar DB'ye ve cache'e yazılır.
  5. Moderasyon: Görsel/metin kontrol → otomatik/manuel iş akışı → ilan görünürlüğü belirlenir.
- Bağımlılıklar: PostgreSQL (tek DB), Redis (cache + channels), S3 (media), Elasticsearch/OpenSearch (arama), Celery (async), Stripe (ödeme), SendGrid/Postmark (mail), Sentry (monitoring), reCAPTCHA & content moderation API.

📐 Düzenlilik (Order) — Adım adım uygulanacak plan (basitten karmaşığa)
1. Proje başlatma: Cookiecutter / repo template, monorepo (apps/frontend + apps/backend) veya iki-repo yaklaşımı (tercih: mono-repo) — .env.example, ADR başlangıç.
2. Temel backend: Django + DRF, PostgreSQL modelleri, auth (AbstractUser), admin.
3. Frontend MVP: Next.js — Ana sayfa SSR (SEO), diğer sayfalar SPA davranışı (App Router ile server&client components).
4. Ödeme altyapısı: Stripe test entegrasyonu + webhooks.
5. Mesajlaşma MVP: Django Channels + Redis (WebSocket); client tarafı Next.js websocket client.
6. Media & Storage: S3 signed-upload + django-storages.
7. Arama: Postgres FTS ilk etap, ileride Elasticsearch for ranking & typo-tolerance.
8. İşler/Asenkron: Celery + Redis + Flower. Cron görevleri için beat.
9. Güvenlik & Moderasyon: reCAPTCHA, content moderation API, rate-limits, CSP, OWASP checklist.
10. CI/CD, testler, observability: GitHub Actions, unit/integration tests, Playwright e2e, Sentry, Prometheus/Grafana (opsiyonel).

✅ Eksiksizlik (Completeness)
- Idea: Tanımlandı (monolitik-modüler Django + Next.js).
- Test: Unit (pytest/pytest-django), API contract (OpenAPI + contract tests), E2E (Playwright), a11y (axe).
- Dokümantasyon: /docs/ADR, API spec (OpenAPI YAML), README, SECURITY, CONTRIBUTING, infra runbooks.

---

1. Tür Analizi
- Seçilen tür: 3. Ticaret & Pazar Yeri (E-Ticaret / İlan Sitesi) + 4. Topluluk & Sosyal (Mesajlaşma). => Hibrit: "İlan Pazaryeri + Sosyal Mesajlaşma + Sahiplendirme (ücretsiz ilanlar)".
- Neden: Ödeme, ilan paketleri, çok kullanıcı iletişimi, ilan yayın akışı ve moderasyon gereksinimleri tipik e-ticaret/ilan sitesi özellikleri; gerçek zamanlı mesajlaşma ise sosyal/komünite fonksiyonu.

2. Mimari Öneri
- Model: Monolitik, modüler (Django "monolith" with well-separated apps). Tek DB (Postgres). Frontend ayrı uygulama (Next.js) — her iki taraf da kendi ölçeklenebilirliğine sahip.
- Rasyonel: Orta büyüklükte ekip için geliştirme hızını korurken, modüller (listings, messaging, payments) sonradan bağımsız servisler haline getirilebilir.

3. Ekosistem Seçimi
- Backend: Django 4.x + Django REST Framework + Django Channels
- Frontend: Next.js (App Router) + React; Next.js kullanalım çünkü ana sayfa SSR/SEO gereksinimi var.
- Diğer: Celery (Redis broker), PostgreSQL, Redis, S3 (MinIO opsiyonel), OpenSearch/Elasticsearch (opsiyonel), Stripe (ödeme).

4. Temel Bileşenlerin Listesi
- Core backend apps:
  - users (AbstractUser, profile, KYC metadata)
  - listings (ilan CRUD, kategori, tags, location)
  - packages_billing (ilan paketleri, kuponlar, paket yönetimi)
  - payments (Stripe integration, webhooks, invoices)
  - messaging (conversations, messages, unread counters) — realtime via Channels
  - media (image/video handling, signed upload, thumbnails)
  - moderation (text/image moderation tasks & admin triage)
  - search (index sync, ranking)
  - notifications (in-app, email, push)
  - analytics (events pipeline)
  - admin (Django admin + Wagtail minimal CMS for content pages)
- Infra & ops:
  - redis (cache, channels), celery workers, worker queues, docker images, k8s manifests (ileride)
- Frontend:
  - Next.js app (routes: /, /listings/[id], /create, /account, /messages, /admin-cms-preview)
  - Component system + Storybook
  - SWR / react-query for client caching
- Güvenlik:
  - reCAPTCHA v3/v2 for forms
  - CSP, HSTS, secure cookies, rate limiting (DRF throttling)
  - Secret management (Vault / cloud KMS)
- 3rd party entegrasyonlar (opsiyonel / öneri):
  - Stripe, Iyzico optional, PayPal
  - SendGrid / Postmark (mail)
  - Cloudflare (WAF, CDN), Vercel (frontend hosting)
  - Sentry, Prometheus/Grafana, Log aggregation (ELK/EFK)
  - Google Vision / AWS Rekognition + Perspective API (text toxicity) — içerik moderasyonu
  - Fraud detection: Stripe Radar + 3rd party (Sift) for higher-risk flows
  - reCAPTCHA, Turnstile (Cloudflare) for bot protection
  - Payment tax services (TaxJar) if needed

5. Karar Ağacı Yolculuğu (adım adım / nedenleriyle)
- A. Monolith vs Microservice?
  - Ekip orta büyüklükte → Monolith (modüler) seç: hızlı iterasyon, tek DB kolaylığı.
  - Eğer ileride >100k DAU veya mesaj trafik çok artarsa → messaging servisini ayır.
- B. Frontend: Next.js mi?
  - Ana sayfa SEO = SSR gerekir → Next.js App Router (server components) seç.
  - Mobil deneyimi için client-side interaktif sayfalar + PWA opsiyonel.
- C. Auth seçimi:
  - Next.js + Django ayrık istemci → Cookie-based httpOnly refresh token rotation (SimpleJWT) + CSRF koruması. Neden: güvenlik + SSR endpointlerle uyum.
- D. Mesajlaşma:
  - Gerçek zamanlı (Channels + Redis) ile başla. Eğer ihtiyaç çok yüksekse → dedicated messaging service (Elixir/Phoenix veya Go + NATS) düşün.
- E. Ödeme:
  - Stripe (well-documented) + webhook iş akışı (ödeme tamamlandı → paket hakkı ver). PCI scope minimal tutmak için Checkout/Elements kullan.
- F. Storage & Media:
  - S3 signed uploads + CDN. Görsel moderasyon pipeline için upload öncesi/sonrası async job.
- G. Search:
  - MVP için Postgres FTS + trigram, ileri aşamada OpenSearch/Elasticsearch index (typo, synonyms, ranking).
- H. Moderasyon:
  - Text: Perspective API; Images: AWS Rekognition (adult/violence) veya Google Vision.
  - Riskli içerik = otomatik takla düşür + manuel moderation queue.
- I. Observability:
  - Sentry for errors, Prometheus for metrics, traces via OpenTelemetry.
- J. Deployment:
  - MVP: Containerized Docker, tek image, managed Postgres (RDS/Cloud SQL), Redis managed, S3, deploy on Render / DigitalOcean App Platform / Heroku alternative.
  - Scale: K8s (GKE/EKS) + autoscaling, split workers, separate DB replicas, read-replicas, search cluster.

6. Nihai Özet (hazır proje altyapısı şablonu)
- Backend stack:
  - Django + DRF + Channels + Celery
  - Postgres (primary), Redis (cache & broker), S3 (media), OpenSearch (search)
  - Stripe, SendGrid, reCAPTCHA, Content moderation APIs
- Frontend stack:
  - Next.js App Router + React + SWR/react-query + TailwindCSS + Storybook
  - Hosting: Vercel (frontend), Django REST on Docker (backend)
- CI/CD:
  - GitHub Actions: lint → tests → build docker image → push registry → deploy
  - CD: staging + production with migrations & canary/blue-green
- Security & compliance:
  - OWASP controls, CSP, secure cookies, rate-limit, data retention & GDPR flows
- Tests:
  - Unit: pytest/pytest-django; Frontend: vitest/jest + RTL
  - Integration/API: pytest + requests + contract-tests (Pact/OpenAPI)
  - E2E: Playwright (user flows: signup, create listing, buy package, message)

7. 📂 Proje Dizin Yapısı (frontend + backend)

Aşağıdaki dizin monorepo örneği (tercih). İki-repo yerine monorepo ile CI kolaylığı sağlanır.

```text
repo-root/
├─ README.md
├─ .env.example
├─ .github/
│  └─ workflows/
│     ├─ ci.yml
│     └─ cd.yml
├─ docs/
│  ├─ ADR/
│  │  ├─ 0001-project-scope.md
│  │  └─ 0002-auth-strategy.md
│  ├─ API/
│  │  └─ openapi.yaml
│  └─ runbooks/
│     └─ restore-db.md
├─ infra/
│  ├─ k8s/
│  │  ├─ backend-deployment.yaml
│  │  ├─ celery-deployment.yaml
│  │  └─ ingress.yaml
│  ├─ terraform/
│  └─ ci-cd/
│     └─ pipeline-templates/
├─ packages/                # shared packages (ui-kit, utils)
│  ├─ ui/
│  └─ shared-types/
├─ services/
│  ├─ backend/              # Django monolith app
│  │  ├─ Dockerfile
│  │  ├─ docker-compose.yml
│  │  ├─ manage.py
│  │  ├─ pyproject.toml / poetry.lock
│  │  ├─ requirements.txt
│  │  ├─ config/
│  │  │  ├─ settings/
│  │  │  │  ├─ base.py
│  │  │  │  ├─ dev.py
│  │  │  │  └─ prod.py
│  │  │  └─ wsgi.py
│  │  ├─ apps/
│  │  │  ├─ users/
│  │  │  │  ├─ models.py
│  │  │  │  ├─ serializers.py
│  │  │  │  ├─ views.py
│  │  │  │  └─ tests/
│  │  │  ├─ listings/
│  │  │  │  ├─ models.py
│  │  │  │  ├─ serializers.py
│  │  │  │  ├─ views.py
│  │  │  │  └─ tests/
│  │  │  ├─ payments/
│  │  │  │  ├─ stripe.py
│  │  │  │  └─ webhooks.py
│  │  │  ├─ messaging/
│  │  │  │  ├─ consumers.py
│  │  │  │  ├─ models.py
│  │  │  │  └─ tests/
│  │  │  ├─ media/
│  │  │  ├─ moderation/
│  │  │  └─ notifications/
│  │  ├─ templates/         # if any server-side pages
│  │  ├─ static/
│  │  ├─ scripts/
│  │  │  └─ migrate_and_seed.sh
│  │  ├─ infra/             # infra-related manifests for backend
│  │  ├─ tests/
│  │  │  ├─ unit/
│  │  │  └─ integration/
│  │  └─ docs/
│  │     └─ api_usage.md
│  └─ frontend/             # Next.js app
│     ├─ Dockerfile
│     ├─ package.json
│     ├─ next.config.js
│     ├─ src/
│     │  ├─ app/            # App-router routes
│     │  │  ├─ page.tsx     # Home (SSR)
│     │  │  ├─ listings/
│     │  │  │  └─ [id]/page.tsx
│     │  │  ├─ create/page.tsx
│     │  │  ├─ messages/page.tsx
│     │  │  └─ account/page.tsx
│     │  ├─ components/
│     │  ├─ lib/            # api clients, auth utilities
│     │  ├─ hooks/
│     │  ├─ styles/
│     │  └─ tests/
│     ├─ public/
│     └─ .storybook/
├─ tests/                   # monorepo-level test runners / e2e
│  ├─ e2e/
│  │  └─ playwright/
│  └─ contracts/
├─ .pre-commit-config.yaml
├─ .eslintrc / .stylelintrc
└─ LICENSE
```

Detaylı dosya örnekleri (önemli dosyalar):
- services/backend/apps/listings/models.py
- services/backend/apps/messaging/consumers.py (Channels consumer)
- services/backend/apps/payments/webhooks.py (Stripe webhook signing verification)
- services/frontend/src/app/page.tsx (SSR ana sayfa, SEO meta tags)
- docs/API/openapi.yaml (tüm REST endpointler)
- docs/ADR/0002-auth-strategy.md (cookie + refresh rotation rationale)

Test & Dokümantasyon:
- Backend tests: services/backend/tests/unit/, services/backend/tests/integration/
- Frontend tests: services/frontend/src/tests/ (unit + integration)
- E2E: tests/e2e/playwright/ (senaryolar: signup → create listing with images → buy package → message seller)
- CI: .github/workflows/ci.yml => lint → test → build → docker push (staging)

Ek notlar (kısa, rasyonel)
- Token storage: httpOnly refresh cookie + access token in memory is ideal for SPA/SSR balance.
- Mesaj geçmişi: DB store + optional message archive in object storage for very large attachments.
- Search: index update via async celery task (eventual consistency acceptable).
- Moderasyon: otomatik false-positive riskine karşı insan-onaylı queue oluşturun.
- GDPR: kullanıcı verisini "erasure" akışı ve audit logs planlayın.

İlerleme için önerilen MVP scope (2–4 sprint):
1. Sprint 1: Kullanıcı kayıt/giriş, ilan CRUD (metin + resim signed upload), ana sayfa SSR.
2. Sprint 2: Paket satın alma (Stripe Checkout), admin panel + ilan yayın akışı.
3. Sprint 3: Messaging (WebSocket minimal), notifications (email), basic moderation hook.
4. Sprint 4: Search tuning, caching (Redis), E2E tests + deploy pipeline.

İsterseniz:
- Bu çıktıyı TXT/ZIP olarak derleyip sunabilirim,
- Veya hemen repo için starter template (Docker + minimal Django+Next scaffold) oluşturup PR açabilirim.