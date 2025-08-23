# Django Karar Ağacı — Mermaid Mindmap (v1.5)




```mermaid
mindmap
  root(("Django Ekosistemi - Birleştirilmiş Genişletilmiş Karar Ağacı (v1.5)"))
    1. Başlangıç: Proje Kurulumu
      Cookiecutter vs custom repo template
      Virtualenv / pyenv / poetry / pipenv
      Monorepo vs Polyrepo
      Repo structure conventions (apps/, config/, infra/)
      Environment strategy: dev/staging/prod
      Project README, CONTRIBUTING, LICENSE
    2. Proje Tipi / Mimari
      Monolith
        Single Django project, multiple apps
        Simple deploy (single image or VPS)
        Small teams, low ops overhead
      API + Frontend Ayrı
        Django REST Framework (DRF) or GraphQL
        Frontend: Next.js / React / Vue / Svelte
        Contract-first (OpenAPI) + mock server (Prism/MSW)
        BFF (Backend for Frontend) pattern
      Microservices
        Domain-separated services
        Service discovery, messaging, distributed tracing
        Event sourcing / CQRS
      CMS Tabanlı
        Wagtail / Django CMS / Mezzanine
        Custom integrations vs headless CMS
    3. Veri Katmanı
      Relational DB
        PostgreSQL
          Django ORM migrations
          Backup: pg_dump, WAL archiving, cloud snapshots
          Connection pooling: pgbouncer
          High availability: streaming replication
          Read replicas + load balancing
        MySQL / MariaDB
          Django ORM migrations
          Replication seçenekleri
        Oracle / MSSQL (kurumsal ihtiyaç)
      SQLite
        Local dev / tests
        Not for prod
      NoSQL & Search
        MongoDB (document store)
        Redis as primary (rare)
        Elasticsearch / OpenSearch (search & analytics)
        Full-text search: Postgres tsvector vs ES
      Time-series & Analytical
        ClickHouse / Timescale / BigQuery
        Data warehouse integration (Snowflake, Redshift)
      Data streaming
        Kafka / Redpanda
        CDC (Change Data Capture)
      Access patterns
        Read-heavy vs write-heavy
        Outbox pattern + ES reindex
    4. API & Entegrasyon Stilleri
      REST (DRF)
        ViewSets, Routers, Pagination, Throttling
        OpenAPI / Swagger auto-doc
        Versioning (url/header)
      GraphQL
        Graphene / Ariadne
        Query complexity limits, persisted queries
      gRPC / Protobuf
        Internal services, typed contracts
      Webhook & Event-driven
        Delivery retry, signing, idempotency
    5. Kimlik Doğrulama & Yetkilendirme
      Built-in Django Auth
        AbstractUser / AbstractBaseUser customization
        Admin, password reset, email confirmation
      Token / JWT
        DRF SimpleJWT or custom implementation
        Refresh tokens, blacklisting, rotation
      OAuth2 / Social Login
        django-allauth / Authlib / django-oauth-toolkit
        SSO (Okta, Keycloak)
      Permission Models
        Object level permissions (django-guardian)
        Role-based vs claim-based
    6. Önbellek & Asenkron İşler
      Cache
        Redis
        Memcached
        Cache strategies: per-view, per-object, template fragment
      Task Queue
        Celery (broker: Redis/RabbitMQ)
        Dramatiq
        Huey
        Task design: idempotency, retry, circuit-breaker
      Background jobs
        Scheduled tasks (Celery Beat, cron, Airflow)
        Event-driven pipelines
    7. Depolama / Media / Asset Yönetimi
      Local FS (dev)
      S3 / MinIO / GCP Storage / Azure Blob
      django-storages integration
      Signed URLs, upload direct to S3
      CDN (CloudFront, Cloudflare, Fastly)
    8. Gerçek Zamanlı İletişim
      Django Channels
        WebSockets, ASGI setup, Redis layer
      Server-Sent Events / Webhooks
        Use cases: notifications, streams
      Realtime alternatives
        Pusher / Ably / Firebase / MQTT
    9. Frontend Entegrasyonu
      Server-side rendered
        Django templates (classic)
        Next.js SSR integrated with API
      Single Page App
        React / Vue / Svelte with API
        State management, SWR/react-query
      Static site / Jamstack
        SSG on build, CDN hosting
    10. Test & Kalite Güvencesi
      Unit tests
        pytest-django / unittest
      Integration tests
        DB fixtures, testcontainers
      Contract tests
        Pact / contract verification
      E2E
        Cypress / Playwright
      Static analysis
        flake8 / pylint / mypy / black
      Security tests
        Dependency scan (Snyk), SAST
    11. Geliştirme Deneyimi (Local DX)
      docker-compose: db, redis, backend, frontend
      makefile / scripts for common tasks
      .env.example, env var strategy
      Local mocks (MSW, Prism) for contract-first
      Hot reload, debug toolbar
      Pre-commit hooks
    12. CI / CD
      CI pipelines
        Lint → Test → Build → Security checks
        Contract verification job (Pact/OpenAPI)
        Auto-generated clients from OpenAPI
      CD strategies
        Image registry (ECR/GCR/Docker Hub)
        K8s rollout (blue/green, canary)
        Feature flags
    13. Deploy & Infra
      Simple VPS
        Gunicorn + Nginx, systemd
        Let's Encrypt
      Container-first
        Dockerfile multi-stage, image scanning
        docker-compose for dev
      Container orchestration
        Kubernetes (Helm charts, ingress, autoscaling)
        Service mesh optional
      Managed / PaaS
        Heroku, Railway, Render, Elastic Beanstalk
      Edge deployments
        Cloudflare Workers, Vercel, Netlify (hybrid with Django)
    14. Monitoring / Logging / Tracing
      Sentry (errors)
      Prometheus + Grafana (metrics)
      OpenTelemetry / Jaeger (tracing)
      ELK / EFK stack for logs
      Health checks & readiness / liveness
      Business metrics dashboards
    15. Security & Compliance
      OWASP top 10 mitigations
      CSP, HSTS, X-Frame-Options, secure cookies
      Rate limiting (throttling)
      Secret management (Vault / cloud KMS)
      GDPR / data retention & export
      Penetration testing
      Threat modeling (STRIDE), security review gates
      Secret rotation & key lifecycle (KMS, age, rotation policy)
    16. Performans & Ölçeklendirme
      DB indexing, query optimization
      Caching layer (Redis), CDN
      Horizontal scaling: multiple app instances
      Load balancing (nginx/ingress/ELB)
      Profiling (django-silk, py-spy)
      Async views for IO heavy endpoints
    17. Packaging, Release & Versioning
      Semantic versioning
      Changelog generation
      Migration management and backward-compatible DB changes
      Release branches, hotfix process
      Dependency pinning and Renovate bot
    18. Popüler Django Eklentileri / Ready-made Apps
      django-allauth
      Wagtail / Django CMS
      Django Oscar (e-commerce)
      django-rest-framework (DRF)
      django-channels
      django-storages
      django-guardian
      django-haystack (search)
      django-cors-headers
    19. Team & Process
      ADR (Architecture Decision Records)
      Code ownership, CODEOWNERS
      Branching model (gitflow/trunk-based)
      RFCs, design reviews
      Pair programming, mob sessions
      Agile vs Kanban vs Shape Up
    20. Data Science & AI Entegrasyonu
      ML models serving (Django + TensorFlow/PyTorch)
      Feature stores
      Data pipelines (Airflow, Prefect)
      Realtime inference with Celery or gRPC
      Vector DB integration (Pinecone, Weaviate)
    21. Diğer / Özel Gereksinimler
      Multi-tenancy
      Data migrations (complex transformation)
      Analytics & events pipeline
      Feature flags / experimentation
      Domain-driven design
    22. E-posta & Bildirim Altyapısı
      Email providers: SMTP, Postmark/Sendgrid
      Templating: django-templated-email, MJML
      Push: Web Push, FCM/APNs
      In-app notifications & digest jobs
    23. Ödemeler & Faturalama
      Stripe / Iyzico / PayPal entegrasyonu
      Abonelikler, metered billing, webhooks
      Vergi, iade, sahtekârlık önleme
    24. Uluslararasılaştırma & Erişilebilirlik
      i18n/l10n: django-rosetta, gettext
      Timezone & para birimi
      WCAG, ARIA, klavye navigasyonu
    25. Veri Yönetişimi & Gizlilik
      PII sınıflandırma, minimizasyon
      Audit logging (kim, ne zaman, ne yaptı)
      Data retention/erasure, DPO süreçleri
      Data catalog & lineage (OpenMetadata)
    26. Tedarik Zinciri Güvenliği
      SBOM (CycloneDX), Dependabot/Renovate
      İmzalı container imajları (cosign)
      Pinned hashes, pip-tools/poetry lock
    27. IaC, Dayanıklılık & Felaket Kurtarma
      Terraform/Ansible, değişiklik onay akışları
      Backup/restore runbook, RPO/RTO hedefleri
      Multi-AZ/region, chaos testing
    28. Maliyet & FinOps
      Kaynak etiketleme, showback/chargeback
      Autoscaling eşikleri, kapasite planlama
      CDN/egress optimizasyonu
    29. Arama & Alaka Düzeyi
      Synonym/typo toleransı
      Sıralama, boosting, A/B
      ES/OpenSearch tune, Postgres FTS
    30. Mobil/Masaüstü İstemciler
      React Native/Flutter entegrasyonu
      Offline-first, delta sync
      Deep link, app auth akışları
    31. Sıfır Kesinti Değişiklikler
      Backfill → dual write → flip → cleanup
      Blue/green + veritabanı şeması evrimi
      Feature flag ile kademeli açılış
    32. Karar Kriterleri
      Ekip yetkinliği, işe alım piyasası
      Regülasyon/uyum gereksinimleri (KVKK/GDPR/PCI)
      SLO/SLA hedefleri (availability, latency)
      Bütçe ve operasyon olgunluğu
    33. Antipatterns & Kokular
      Erken mikroservisleşme
      Aşırı genel cache (stale data, invalidation yok)
      Şemasız eventler (versiyonlama yok)
      Gizli bilgileri .env yerine repo’ya koymak
```
