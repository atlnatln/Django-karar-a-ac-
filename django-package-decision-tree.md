# Django 3rd Party Paketleri — Karar Ağacı (İhtiyaca Göre Seç)

Bu karar ağacı, “django-package-list.md” içeriğindeki paketleri, kullanım ihtiyacına göre adım adım seçebilmenizi sağlar.  
Notlar:
- ⭐ popüler veya pratikte sık tercih edilen seçenekleri belirtir.
- Bir dalda birden fazla paket alternatifi olabilir; ekosistem, tercih ve uyumluluk gereksinimlerinize göre tekini seçin.

İçindekiler:
- [Kimlik / Yetkilendirme](#kimlik--yetkilendirme)
- [Admin / Panel](#admin--panel)
- [REST API / GraphQL](#rest-api--graphql)
- [Cache / Session](#cache--session)
- [Statik / Medya](#statik--medya)
- [Background tasks](#background-tasks)
- [E-posta / Bildirim](#e-posta--bildirim)
- [Uluslararasılaştırma](#uluslararasılaştırma)
- [Form / UI helpers](#form--ui-helpers)
- [Yetki / RBAC](#yetki--rbac)
- [Denetim / Loglama](#denetim--loglama)
- [Deployment / Health](#deployment--health)
- [Test / Kalite / Güvenlik](#test--kalite--güvenlik)
- [CLI / Yardımcılar](#cli--yardımcılar)
- [E-Ticaret / Satış ve Ödeme / Faturalama](#e-ticaret--satış-ve-ödeme--faturalama)
- [Arama / Filtreleme](#arama--filtreleme)
- [SEO / Pazarlama ve Yardımcıları](#seo--pazarlama-ve-yardımcıları)
- [Analiz / Raporlama](#analiz--raporlama)
- [Coğrafi / Harita](#coğrafi--harita)
- [İçerik / CMS Gelişmiş](#içerik--cms-gelişmiş)
- [Mesajlaşma / Chatbot](#mesajlaşma--chatbot)
- [Veri Görselleştirme](#veri-görselleştirme)
- [Dosya / Paylaşım](#dosya--paylaşım)
- [Veri / ETL / API Tüketimi](#veri--etl--api-tüketimi)
- [İzin / Lisanslama / DRM](#izin--lisanslama--drm)
- [Performans / Ölçekleme ve Cache Ekleri](#performans--ölçekleme-ve-cache-ekleri)
- [Ortam / Konfigürasyon](#ortam--konfigürasyon)
- [İleri ORM Yardımcıları ve Enum / Sabit](#ileri-orm-yardımcıları-ve-enum--sabit)
- [İçerik Moderasyonu](#içerik-moderasyonu)
- [Yayın / Workflow](#yayın--workflow)
- [Zamanlama & Takvim ve Scheduler](#zamanlama--takvim-ve-scheduler)
- [Önbellek Tüketici Araçları](#önbellek-tüketici-araçları)
- [Bildirim / Queue Integrasyon](#bildirim--queue-integrasyon)
- [Graph / NoSQL entegrasyonları](#graph--nosql-entegrasyonları)
- [Veri Kalitesi / Validation ve CSV/Excel İçe Aktarma](#veri-kalitesi--validation-ve-csvexcel-içe-aktarma)
- [International Payment Adapters](#international-payment-adapters)
- [Access Logging / Forensics](#access-logging--forensics)
- [Sosyal / UGC Moderasyon](#sosyal--ugc-moderasyon)
- [Görsel Performans](#görsel-performans)
- [Accessibility yardımcıları](#accessibility-yardımcıları)
- [Migration & Schema Araçları](#migration--schema-araçları)
- [Dokümantasyon & API Docs](#dokümantasyon--api-docs)
- [ML/AI entegrasyon (yardımcı)](#mlai-entegrasyon-yardımcı)
- [Async / Concurrency helpers](#async--concurrency-helpers)
- [Güvenlik & Şifreleme ve Spam/Abuse](#güvenlik--şifreleme-ve-spamabuse)
- [Slug / URL helpers](#slug--url-helpers)
- [Taxonomy & Etiketleme ve Ağaç Yapıları](#taxonomy--etiketleme-ve-ağaç-yapıları)
- [Sürümleme & Versiyonlama ve Soft Delete](#sürümleme--versiyonlama-ve-soft-delete)
- [Multitenancy ve Tenant Yardımcıları](#multitenancy-ve-tenant-yardımcıları)

---

## Kimlik / Yetkilendirme
- Sadece temel kimlik sistemi yeterli mi?
  - Evet → django.contrib.auth
  - Hayır → aşağıdakilerden seç
- Sosyal giriş ve e-posta doğrulama gerekiyor mu?
  - Evet → django-allauth ⭐
- Kurumsal protokoller gerekli mi?
  - OAuth2 provider → django-oauth-toolkit ⭐
  - OpenID Connect → mozilla-django-oidc ⭐
  - SAML2 → django-saml2-auth ⭐
- MFA/2FA gerekiyor mu?
  - Evet → django-two-factor-auth ⭐
- Oturum ve güvenlik politikaları:
  - Bir kullanıcı için çoklu oturum yönetimi → django-user-sessions ⭐
  - Şifre güvenliği (policy/validators) → django-password-validators ⭐
- İzin modeliniz nasıl?
  - Satır (object) bazlı izin → django-guardian ⭐
  - Kural tabanlı izin → django-rules ⭐

## Admin / Panel
- Tema/arayüz özelleştirme:
  - Modern/gelişmiş tema → django-grappelli ⭐ / django-suit ⭐ / django-jet ⭐
  - Admin arayüzünü özelleştirme (renk, font vb.) → django-admin-interface ⭐
- Admin iş akışları:
  - CSV/Excel import-export → django-import-export ⭐
  - Liste sıralama sürükle-bırak → django-admin-sortable2 ⭐
  - Gelişmiş filtreler → django-adminfilters ⭐
  - ModelAdmin menüsünü yeniden sıralama → django-modeladmin-reorder ⭐
  - Admin’de özel aksiyon butonları → django-object-actions ⭐

## REST API / GraphQL
- API türü:
  - REST → djangorestframework ⭐
    - OpenAPI/Swagger dokümantasyonu → drf-yasg ⭐ / drf-spectacular ⭐
    - Auth:
      - JWT → djangorestframework-simplejwt ⭐
      - Token tabanlı → django-rest-knox ⭐
    - Router/Endpoint ihtiyaçları:
      - İç içe router → drf-nested-routers ⭐
      - Response alanlarını esnek seç → drf-flex-fields ⭐
    - Filtreleme → django-filter ⭐
  - GraphQL:
    - Klasik ekosistem → graphene-django ⭐
    - Alternatif/modern → strawberry-django ⭐

## Cache / Session
- Cache backend tercihi:
  - Redis → django-redis ⭐
  - Veritabanı → django-dbcache
  - Disk → django-diskcache ⭐
  - Memcached → django-memcached
- ORM sorgu cache’i → django-cachalot ⭐ / django-cacheops ⭐
- HTTP cache yönetimi → django-httpcache
- Fonksiyon sonuçlarını cache’leme → django-memoize ⭐
- Cache temizleme komutu → django-clearcache
- Cache izleme/panel → django-cache-panel / django-cache-monitor
- Hız odaklı cache layer → django-fastcache
- Oturum yönetimi (core) → django.contrib.sessions

## Statik / Medya
- Statik dosya sunumu:
  - Basit dağıtım/WSGI ile → whitenoise ⭐
  - Brotli ön-sıkıştırma → django-brotli-static ⭐
- Storage backend (S3/GCS/Azure):
  - Genel → django-storages ⭐
  - S3’e direkt upload → django-s3direct ⭐
  - CloudFront imzalı URL → django-cloudfront-toolbox ⭐
- Medya yönetimi ve editör:
  - Görsel işleme/thumbnail → django-imagekit ⭐ / sorl-thumbnail ⭐ / easy-thumbnails ⭐
  - Dinamik image field → django-versatileimagefield ⭐
  - WYSIWYG → django-ckeditor ⭐ / django-tinymce ⭐ / django-markdownx ⭐ / django-pagedown
  - Dosya yönetimi → django-filer ⭐
  - Test medyası fixture → django-media-fixtures

## Background tasks
- Görev kuyruğu:
  - Celery ekosistemi → celery ⭐ + django-celery-beat ⭐ + django-celery-results ⭐
  - Daha hafif alternatifler → huey ⭐ / dramatiq ⭐ / rq ⭐ + django-rq ⭐ / django-q ⭐
- Zamanlayıcı/Scheduler:
  - APScheduler → apscheduler ⭐
  - RQ tabanlı → django-rq-scheduler ⭐
- Basit arkaplan görevleri → django-background-tasks ⭐
- E-posta’yı Celery ile → django-celery-email ⭐

## E-posta / Bildirim
- E-posta servisleri (Mailgun/SendGrid vb.) → django-anymail ⭐
- E-posta kuyruğa alma/gönderme → django-post_office ⭐ / django-mailer ⭐
- Bildirim sistemi → django-notifications-hq ⭐
- Push bildirim → django-push-notifications ⭐ / django-fcm ⭐ / django-webpush ⭐
- Slack entegrasyonu → django-slack ⭐
- Mesaj framework genişletme → django-messages-extends ⭐
- SMS → django-sms ⭐

## Uluslararasılaştırma
- Core yardımcılar → django.contrib.humanize
- Çeviri yönetimi:
  - Web arayüzünden .po düzenleme → django-rosetta ⭐
- Çok dilli içerik:
  - Model tabanlı çok dillilik → django-parler ⭐ / django-modeltranslation ⭐ / django-translated-fields ⭐ / django-i18nfield ⭐
- URL/dil yönetimi → django-localeurl ⭐ / django-multilingual-urlresolver ⭐
- Babel entegrasyonu → django-babel ⭐
- Ülkeye özgü alanlar/validasyonlar → django-internationalflavor ⭐

## Form / UI helpers
- Form render ve Bootstrap entegrasyonu → django-crispy-forms ⭐
- Widget özelleştirme → django-widget-tweaks ⭐
- Otomatik tamamlama/Select2 → django-autocomplete-light ⭐
- Tablo oluşturma/sıralama → django-tables2 ⭐
- Form wizard (çok adımlı) → django-formtools
- JS destekli formset → django-formset-js ⭐
- Dinamik alanlar → django-dynamic-forms ⭐
- Form builder → django-formfactory
- Alternatif render → django-floppyforms / django-uni-form

## Yetki / RBAC
- Rol tabanlı izin → django-role-permissions ⭐ / django-roles / django-teamwork
- Organizasyon/üyelik → django-organization ⭐ / django-groups-manager
- Kullanıcı hesap/profil → django-user-accounts ⭐ / django-userena ⭐
- Davetiye tabanlı kayıt → django-invitations ⭐
- İzinleri geliştirme → django-improved-permissions
- JSON tabanlı policy → django-access-policy ⭐

## Denetim / Loglama
- Model değişiklik geçmişi → django-simple-history ⭐ / django-reversion ⭐
- Audit trail → django-auditlog ⭐
- Request/response loglama → django-request ⭐ / django-request-logger
- Aktivite stream → django-activity-stream ⭐
- Ziyaretçi tracking → django-tracking ⭐
- IP tespiti → django-ipware ⭐
- Request ID loglama → django-log-request-id ⭐
- Admin log genişletme → django-admin-logs
- Logları DB’de saklama → django-db-logger ⭐
- Log rotasyonu yardımcıları → django-log-rotation-manager

## Deployment / Health
- Sağlık kontrolleri → django-health-check ⭐
- Uygulama sunucusu:
  - WSGI → gunicorn
  - ASGI → uvicorn
- Ayar/konfig yönetimi → django-environ ⭐ / django-configurations ⭐ / django-split-settings ⭐
- Secrets yönetimi → django-secrets ⭐
- Ayar güvenliği → django-debreach
- Metrikler/izleme → django-prometheus ⭐ / django-opentelemetry ⭐

## Test / Kalite / Güvenlik
- Test çatıları → pytest-django ⭐ / tox
- Test verisi → factory_boy ⭐ / model_bakery ⭐ / faker
- Performans/profiling → django-debug-toolbar ⭐ / django-silk ⭐ / django-query-profiler
- Coverage → coverage
- Güvenlik taraması → bandit / safety
- Migration hataları → django-migration-linter ⭐

## CLI / Yardımcılar
- Genişletilmiş komutlar → django-extensions ⭐
- Fake data seed → django-seed ⭐
- Fixture yönetimi → django-fixture-magic ⭐
- Click tabanlı komutlar → django-click ⭐
- Gelişmiş runserver/shell → django-runserver-plus ⭐ / django-shell-plus ⭐
- Manage.py hızlandırma → django-manage-fast
- Admin honeypot → django-admin-honeypot ⭐
- Eski dosyaları temizleme → django-cleanup ⭐
- Dev deneyimi → django-git-hooks ⭐ / django-devserver ⭐ / django-live-reload ⭐

## E-Ticaret / Satış ve Ödeme / Faturalama
- E-ticaret çerçevesi:
  - Zengin özellikli → django-oscar ⭐ / saleor ⭐ (headless/GraphQL)
  - Hafif mağaza → django-shop ⭐
- Sepet/ödeme yardımcıları:
  - Sepet → django-carton ⭐
  - Ödeme abstraction → django-payments ⭐
  - Abonelik planları → django-plans ⭐ / django-subscriptions ⭐
  - Çok satıcılı pazar → django-vendor ⭐
  - Para/vergilendirme → django-prices ⭐
  - Fatura → django-invoices ⭐
- Ödeme ağ geçitleri:
  - Stripe → dj-stripe ⭐
  - PayPal → django-paypal ⭐
  - Braintree → django-braintree ⭐
  - MercadoPago → django-mercadopago ⭐
  - Bitcoin → django-bitcoin-payments ⭐
  - Adyen → django-adyen ⭐
  - Paymentwall → django-paymentwall ⭐
  - Bankalarka bağlantı → django-banklink ⭐
  - PagSeguro → django-pagseguro ⭐

## Arama / Filtreleme
- Full-text arama abstraction → django-haystack ⭐
- Elasticsearch:
  - Python DSL → elasticsearch-dsl ⭐
  - Django entegrasyon → django-elasticsearch-dsl ⭐
- Solr → django-solr
- Model tabanlı arama → django-watson ⭐
- Basit autocomplete → django-simple-autocomplete / django-searchable-select ⭐
- Gelişmiş arama formu → django-advanced-search ⭐
- Anahtar kelime filtre → django-keyword-filter
- Queryset search helper → django-qssearch ⭐
- Whoosh backend → whoosh
- Arama fonksiyonlarını zenginleştirme → django-search-extensions ⭐

## SEO / Pazarlama ve Yardımcıları
- Meta/SEO alanları → django-seo2 ⭐ / django-meta ⭐ / django-seo-helper ⭐ / django-seo-admin ⭐
- Sitemap/robots:
  - Core → django-sitemap / django-redirects / django-sitemaps (core)
  - XML genişletmeler → django-xml-sitemaps ⭐
  - Kanonik URL → django-canonical ⭐
  - robots.txt yönetimi → django-robots ⭐
- Sosyal paylaşım → django-social-share ⭐
- Ziyaretçi takibi (anonim) → django-tracking-anonymous ⭐
- A/B test → django-ab-tests ⭐
- Kampanya → django-marketing ⭐
- Open Graph/Twitter Cards → django-open-graph ⭐

## Analiz / Raporlama
- Analytics entegrasyonları → django-analytical ⭐ / django-matomo ⭐
- Metrik kaydı → django-metrics ⭐ / django-activity-metrics ⭐
- Chart’lar → django-chartjs ⭐ / django-charts ⭐
- DataTables → django-datatable-view ⭐
- Rapor/PDF/Excel → django-report-builder ⭐ / django-reports ⭐
- Query sayacı → django-querycount ⭐
- Dashboard’lar → bkz. Veri Görselleştirme

## Coğrafi / Harita
- Core GIS → django-gis (django.contrib.gis)
- Harita widget/alan:
  - Leaflet → django-leaflet ⭐
  - Admin konum alanı → django-geoposition ⭐ / django-mapwidgets ⭐
  - OSM field → django-osmfield ⭐
  - Adres → koordinat → django-geocoder ⭐ / django-location-field ⭐
- Geo veriler:
  - GeoJSON → django-geojson ⭐
  - Spatialite → django-spatialite ⭐
  - GeoPandas → django-geopandas ⭐

## İçerik / CMS Gelişmiş
- CMS seçimi:
  - Wagtail → wagtail ⭐
  - Klasik CMS → django-cms ⭐ / mezzanine ⭐ / feincms ⭐ / django-fiber ⭐
- İçerik blokları → django-flatblocks ⭐ / django-sections ⭐
- Editör/markup → django-markdownx ⭐ / django-pagedown / django-markupfield ⭐
- Workflow → django-publisher ⭐

## Mesajlaşma / Chatbot
- Gerçek zamanlı/ASGI → django-channels ⭐ + channels-redis ⭐
- Özel mesajlaşma → django-private-chat ⭐ / django-messages
- WebSocket messaging → django-websocket-redis ⭐
- Live chat → django-livechat ⭐
- Bot/entegrasyon:
  - Basit bot → django-bot ⭐
  - Telegram → django-telegrambot ⭐
  - Dialogflow → django-dialogflow ⭐
  - Slack bot → django-slackbot ⭐

## Veri Görselleştirme
- Dashboard/Grafik:
  - Plotly Dash → django-plotly-dash ⭐
  - Bokeh → django-bokeh ⭐
  - Altair → django-altair ⭐
  - Vega/VegaLite → django-vega ⭐
  - Pivot → django-pivot ⭐
  - Dashboard framework → django-dashboards ⭐ / django-gridstack ⭐ / django-metricsboard ⭐
  - Chartist → django-chartist ⭐
  - Vue.js charts → django-vue-charts ⭐

## Dosya / Paylaşım
- Dosya indirme/indirilebilirlik:
  - Güvenli indirme → django-sendfile ⭐ / django-downloadview ⭐ / django-filetransfers ⭐
  - Özel dosya erişimi → django-private-storage ⭐
- Büyük/parçalı upload → django-large-upload ⭐ / django-resumable ⭐
- Entegrasyon:
  - Dropbox → django-dropbox ⭐
  - Google Drive → django-gdrive ⭐
  - Servis transferleri → django-transfer ⭐

## Veri / ETL / API Tüketimi
- Import/Export:
  - CSV/Excel → django-import-export ⭐ / django-data-wizard ⭐
  - Büyük/chunked upload → django-chunked-upload ⭐
- ETL pipeline → django-etl ⭐
- Pandas entegrasyonu → django-pandas ⭐ / django-pandas-integration ⭐
- API client yardımcıları → django-api-client ⭐ / django-requests ⭐ / django-xmlrpc ⭐ / django-graphql-client ⭐
- OAuth consumer → django-oauth-consumer ⭐
- DRF + DataTables → django-rest-framework-datatables ⭐

## İzin / Lisanslama / DRM
- Lisans yönetimi → django-license ⭐ / django-subscription-licenses ⭐ / django-drm ⭐
- Feature flags → django-feature-flags ⭐ / django-constance ⭐ (+ admin: django-constance-admin ⭐)
- Multi-tenant:
  - Schema tabanlı → django-tenant-schemas ⭐ / django-tenants ⭐
- Kullanıcı taklit → django-impersonate ⭐
- Maintenance modu → django-maintenance-mode ⭐
- Terms/Conditions → django-termsandconditions ⭐
- Cookie izin yönetimi → django-cookie-consent ⭐

## Performans / Ölçekleme ve Cache Ekleri
- DB connection pool → django-db-geventpool ⭐
- Sharding/replica:
  - Sharding → django-sharding ⭐
  - Read-replica router → django-read-replica-router ⭐
- Template render hızlandırma → django-fast-render ⭐
- Gzip middleware → django-gzip-middleware ⭐
- SQL explorer → django-sql-explorer ⭐
- LRU cache helpers → django-lru-cache ⭐
- Dogpile cache paternleri → dogpile.cache

## Ortam / Konfigürasyon
- Runtime ayarları/switch → django-constance ⭐ (+ admin: django-constance-admin ⭐)
- Tekil ayar modelleri → django-solo ⭐
- Ortam değişkenleri:
  - .env dosyası → django-dotenv ⭐
  - Küçük yardımcı → django-env-var

## İleri ORM Yardımcıları ve Enum / Sabit
- Model yardımcıları → django-model-utils ⭐
- Toplu güncelleme optimize → django-bulk-update ⭐
- Migration yardımcıları → django-extensions-migration-helper
- Enum/Choices → django-enumfields ⭐ / django-choices ⭐

## İçerik Moderasyonu
- İçerik onay/workflow → django-moderation ⭐
- Spam kontrol (Akismet) → django-akismet ⭐
- İçeriği kullanıcıların flag’lemesi → django-flag ⭐

## Yayın / Workflow
- RQ için panel → django-rq-dashboard ⭐
- AWS SQS yardımcıları → django-sqs-utils ⭐
- Basit workflow/state machine → django-workflows ⭐

## Zamanlama & Takvim ve Scheduler
- iCal/ICS feed → django-ical ⭐
- Takvim yardımcıları → django-calendars ⭐
- Etkinlik/RSVP → django-events ⭐
- Cron job kayıtları → django-crontab ⭐

## Önbellek Tüketici Araçları
- Cache invalidation helpers → django-cache-toolbox ⭐
- Cache hit/miss izleme → django-cache-monitor ⭐
- Stale cache handler → django-stale-cache-handler

## Bildirim / Queue Integrasyon
- Pub/Sub → django-pubsub ⭐
- Kafka → django-kafka ⭐
- RabbitMQ yardımcıları → django-rabbitmq-helper ⭐

## Graph / NoSQL entegrasyonları
- Neo4j OGM → neomodel ⭐
- MongoEngine entegrasyonu → django-mongoengine ⭐
- RedisGraph wrapper → django-redisgraph ⭐

## Veri Kalitesi / Validation ve CSV/Excel İçe Aktarma
- Cleanup suite → django-cleanup-utils
- Duplicate detection → django-dedupe ⭐
- E-posta doğrulama → django-validate-email ⭐
- CSV/Excel/Pandas → bkz. Veri / ETL / API Tüketimi

## International Payment Adapters
- Vergi/fatura → django-sovos ⭐ / django-taxjar ⭐ / django-invoice-generator ⭐

## Access Logging / Forensics
- Audit trails ve adli analiz → django-audittrail ⭐
- Gelişmiş request logging → django-request-logger
- Log rotasyonu → django-log-rotation-manager

## Sosyal / UGC Moderasyon
- UGC moderation helpers → django-ugc-tools ⭐
- Görsel moderasyon (NSFW) → django-image-moderation ⭐
- Metin moderasyon/toxicity → django-text-moderation ⭐

## Görsel Performans
- Lazy-load → django-lazy-images ⭐
- Responsive srcset → django-srcset ⭐
- WebP dönüştürme → django-webp-converter ⭐

## Accessibility yardımcıları
- A11Y checkers → django-a11y-tools ⭐
- Template/form a11y helpers → django-accessibility-hooks

## Migration & Schema Araçları
- Migration squash → django-migration-squash
- Schema evolution → django-schema-evolution ⭐
- Migration kontrolleri → django-migrate-checker ⭐

## Dokümantasyon & API Docs
- OpenAPI generator → drf-spectacular ⭐ (REST için) / drf-yasg ⭐
- API doc helpers → django-apidocs ⭐
- Markdown tabanlı dokümantasyon → django-markdown-docs ⭐

## ML/AI entegrasyon (yardımcı)
- Model store & inference → django-ml-utils ⭐
- TensorFlow → django-tensorflow ⭐
- PyTorch servis wrapper → django-pytorch-deploy ⭐
- MLflow entegrasyonları → django-mlflow ⭐
- Workflow state machine → django-river ⭐

## Async / Concurrency helpers
- Core ASGI yardımcıları → asgiref (core)
- Channels için queue/worker → channels-queue ⭐
- Async DB helpers (deneysel) → django-async-orm-helpers ⭐

## Güvenlik & Şifreleme ve Spam/Abuse
- Alan şifreleme:
  - Genel → django-cryptography ⭐
  - Fernet → django-fernet-fields ⭐
- OTP → django-otp ⭐
- Rate limiting/throttle:
  - IP/User bazlı → django-ratelimit ⭐
  - Basit throttling → django-throttle
  - Diğer yardımcılar → django-limiter ⭐
- Brute force koruması → django-axes ⭐
- Antispam/honeypot → django-antispam ⭐ / django-honeypot ⭐

## Slug / URL helpers
- Otomatik slug → django-autoslug ⭐
- Unique slug → django-unique-slugify
- Kısa URL/redirect → django-shorturls ⭐

## Taxonomy & Etiketleme ve Ağaç Yapıları
- Etiketleme → django-taggit ⭐ / django-taxonomies ⭐ / django-tagging
- Ağaç yapıları:
  - Nested set → django-mptt ⭐
  - Alternatif → django-treebeard ⭐ / django-materialized-path ⭐

## Sürümleme & Versiyonlama ve Soft Delete
- Model versiyonlama → django-reversion ⭐ / django-modelversioning ⭐ / django-db-history ⭐
- Model change loggers (alternatif) → django-auditlog (zaten üstte)
- Soft delete/Arşivleme:
  - Soft-delete modeller → django-safedelete ⭐
  - Arşivleme/restore → django-archive ⭐ / django-trashcan ⭐

## Multitenancy ve Tenant Yardımcıları
- Tenant kullanıcılar → django-tenant-users ⭐
- Shared schema yardımcıları → django-shared-schema-tools
- Tenant client yönetimi → django-tenant-clients ⭐
- Tenant yönlendirme/cache/SSL:
  - SSL yönlendirme → django-tenant-ssl ⭐
  - Tenant-aware cache → django-tenant-caching ⭐