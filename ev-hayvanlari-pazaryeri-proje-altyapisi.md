Proje

İki özellikli (İlanlı satış + sahiplendirme) bir çevrimiçi ev hayvanları pazaryeri.

Kullanıcılar kayıt olur. İki tip ilan:

- **Ücretli satış ilanları:** Kullanıcılar ilan paketleri satın alıp (dış HTML ödeme linki ile) evcil hayvanlarını satış amacıyla ilân eder. Ödeme altyapısı entegre edilmeyecek; site ödeme için dış bağlantı sağlar.  
  **Güçlendirme:** Ödeme süreci tek bir uygulama altında soyutlanmalı (ör. `odeme` modülü / PaymentProvider soyutlaması). Bu soyutlama:
  - external_link oluşturma ve tıklama takibini kaydeder (idempotent istek anahtarı ile),
  - admin onayı ve paket atamasını merkezi bir iş akışı içine alır,
  - ileride webhook destekli gerçek ödeme sağlayıcısına geçiş için adapter/arayüz sunar,
  - tıklama/geri dönüş logları ile manuel onay sürecini hızlandıracak raporlar üretir (admin UI / moderation aracıyla kullanılacak).

- **Ücretsiz sahiplendirme ilanları:** Satış amaçlı olmayan, ücretsiz sahiplendirme ilanlarıdır; bunlar ilan paketi satın almadan verilebilir.

**Mail sunucu:** Google Workspace (iletişim için) kullanılacak. SMTP ayarları environment üzerinden yönetilecek; e-posta queue ve retry mekanizması bulunmalı.

**Sosyal giriş ve kayıt:** Facebook ve Google OAuth ile giriş ve kayıt yapılacak. Sosyal oturumlar için email doğrulama politikası, account linking ve duplicate account prevention tanımlanmalı.

**Admin onaylı kayıt:** Telegram botu kullanılacak, admin Telegram üzerinden kullanıcıyı aktif yapabilecek.  
**Güçlendirme:** Admin kimliği doğrulaması ve bot–backend arası veri bütünlüğü ve kimlik doğrulama sağlanmalı. Teknik gereksinimler:
- Bot → backend iletişiminde imzalama (payload + timestamp + nonce ile HMAC veya eşdeğeri) ve replay koruması,
- Callback endpoint’leri için isteğin geldiği admin Telegram ID doğrulaması ve zaman penceresi kontrolü,
- Onay işlemleri atomik yapılmalı (veritabanı transaction) ve audit log tutulmalı.

**İlan verenler ile alıcı/sahiplendirmek isteyenler arasında** Facebook benzeri anlık mesajlaşma (site içi) olacak.  
**Güçlendirme:** Mesajlaşma altyapısı kimlik doğrulaması ve kullanım sınırı (rate limit) sağlayacak şekilde tasarlanmalı. Teknik detaylar:
- WebSocket/ASGI tabanlı gerçek zamanlı kanal ile kimlik doğrulama (session veya token bazlı) zorunlu,
- Mesaj üretimi için per-user throttling ve attachment size/type kontrolleri,
- Mesajlar DB'ye yazılırken idempotency ve ordering garantileri göz önünde bulundurulmalı,
- Presence ve unread counters için hafızada (cache) tutulan hızlı katman kullanılmalı,
- Spam/misuse tespiti için mesaj rate/log analizi ve otomatik kısıtlama (geçici engelleme) eklenmeli.

**Frontend SSR entegrasyonu:** Ana sayfa ve ilan detayları için Next.js tabanlı SSR kullanılacak; bu nedenle Django tarafında Next.js'in sunucu tarafı render'ından veri çekmek için açık, sürülebilir ve test edilebilir BFF (Backend-for-Frontend) endpoint sözleşmeleri (API contract / OpenAPI) tanımlanmalıdır. SSR entegrasyonu aşağıdaki ek gereklilikleri karşılamalı:
- BFF endpoint’leri güvenilir ve prod-ready olacak şekilde low-latency JSON payload sağlayacak,
- Contract-first yaklaşımı ile API sözleşmeleri (OpenAPI) kesinleştirilecek,
- Endpoint testleri (unit, integration) ile sözleşme doğrulanacak,
- Cache invalidation, idempotency ve preview token destekleri BFF üzerinden güvenli şekilde sağlanacak,
- SSR auth/session paylaşımı için domain-scoped, HttpOnly, SameSite ayarlı session cookie veya güvenli token akışı kullanılacak,
- Preview token veya SSR özel token üretimi ve doğrulaması HMAC ile güvence altına alınacak,
- CSRF ve CORS kuralları net şekilde tanımlanacak.

**Media & image delivery:** Next.js SSR sırasında kullanılacak görseller için backend signed URL üretme ve / veya CDN/edge kullanım akışı tanımlanmalı; SSR sayfaları image URLs/variants ile dönecek. Media pipeline ile üretilen varyantlar (webp, avif, srcset) Next.js tarafında tanımlı image loader/performans ayarlarına uygun olmalıdır.

**SSR cache & invalidation:** Ana sayfa kısa TTL ile CDN/edge cache’e çıkacak; ilan detaylarında içerik değiştiğinde (moderasyon onayı, medya güncellemesi, paket ataması) cache invalidation akışı (BFF notify / cache purge veya ISR benzeri mekanizma) tanımlanmalıdır.

**Preview & admin workflow:** Moderasyon onayı öncesi veya onaylandıktan hemen sonra yöneticinin Next.js üzerinde önizleme yapabilmesi için preview endpoint veya kısa ömürlü preview token mekanizması tanımlanacak. Bu, HMAC ile doğrulanan preview token üretimi ve Next.js'de preview mod trigger'ı şeklinde çalışacaktır.

**Ana sayfa SEO uyumlu olacak; diğer sayfalar mobil kullanıcı deneyimi odaklı (responsive, hızlı).**
- Ana sayfa server-side render ile sunulmalı, dinamik içerikler progressive enhancement ile eklenmeli.
- Önemli meta, Open Graph, JSON-LD structured data her ilan detay sayfasına eklenecek.

**Mimari:** Django ile tek veritabanı (monolitik + modüler yapı).  
Tüm uygulamalar mümkün olduğunca hazır bileşenler kullanılarak sağlanacak; özel kod minimumda tutulacak.  
**Güçlendirme:** Başlangıçta monolitik modüler yapı korunacak ancak her modülün dışa açık bir arayüzü (API / service interface) olacak; böylece:
- ileride messaging, search, media gibi kritik parçalar ayrı servislerle değiştirilebilir,
- bağımlılıklar açıkça tanımlı olduğundan refactor maliyeti düşer.

**Medya:** kullanıcılar tarafından yüklenen medya VPS içinde in-house olarak tutulacak. Yüklenen medyalar VPS’de alan optimizasyonu için boyut ve format olarak optimize edilecek.  
**Güçlendirme:** Medya pipeline tasarımı:
- Upload → geçici alan (quarantine / tmp) → arka plan göreviyle doğrulama (dosya türü/virüs taraması opsiyonel) → optimize (resize, webp, transcode video) → final storage.
- Storage için soyutlama katmanı olacak; yerel FS ile başlayıp ileride obje depolama + CDN’e taşımayı kolaylaştıracak.
- Erişim kontrolü: her medya nesnesi için per-object permission, imzalı/tanımlı süreli erişim URL’leri desteklenecek (signed URLs). Sunumda Nginx internal redirect veya streaming proxy kullanılabilir.
- Depolama kotası, yaşlı dosya temizleme (retention policy) ve otomatik thumbnail/variant üretimi tanımlanmalı.

**Arama:** sadece OpenSearch.  
**Güçlendirme:** Arama mimarisi:
- İndex mapping tasarımında typo-tolerant analyzer, n-gram/edge-ngram, synonym list ve geo-point destekleri planlanmalı.
- Create/update/delete olaylarında sinyal tabanlı incremental indexleme (enqueue → background job) kullanılacak.
- Tam reindex için yönetim komutu ve periyodik (scheduled) full reindex iş akışı tanımlanmalı; reindex sırasında versiyonlama/alias kullanımıyla downtime minimize edilecek.

**Moderasyon:** manuel admin onaylı tek taraflı Telegram botu kullanılacak; yüklenen ilanlar admin Telegram uygulamasına bot mesajı olarak gelecek, admin Telegram üzerinden ilanları onaylayacak.  
**Güçlendirme:** İlan ve kullanıcı onay bilgileri tek model üzerinden tutulmalı, tekrar eden kayıtlar olmamalı. Ayrıca:
- Moderasyon kayıtları için Generic ilişki (tek tablo hem ilan hem kullanıcı için) ve unique constraint/kalıcılık politikası kullanılmalı,
- Moderasyon eylemleri için atomic update + audit log zorunlu,
- Moderasyon queue throttling ve admin iş listesinin batch-manage edilebilmesi sağlanmalı.

**Bildirim:** kayıtlı üyelere bildirim sistemin kendi mesajlaşma uygulaması üzerinden yapılacak.  
**Güçlendirme:** Bildirimler asenkron şekilde dağıtılacak bir yapıya sahip olmalı:
- Notification yazımı DB’de single source-of-truth olacak,
- WebSocket/real-time push, e-posta queue ve opsiyonel push notification kanalları için publisher/subscriber pattern ile dağıtım,
- Her bildirim için teslimat durumu (queued/sent/failed) ve retry mantığı olacak.

**Django sürümü:** en son 5.x uyumlu.

---

### İSTENİLENLER:

- **Proje kök dizini ağacı (Türkçe dizin/adlar)** — her app için ana dosyalar ve önemli alt dizinler.
- Her app için:
  - Amaç (tek cümle).
  - Kullanılacak ana bileşenlerin türü (kısa ve genel).
  - Gerekli küçük özelleştirmeler (varsa, kısa maddeler).
  - İlişkili DB tabloları / modeller (alan isimleri ve türleri, örnek ilişkiler).
- Tek DB monolitik+modüler tablo listesi (app bazlı, tablolar ve ana alanlar, anahtar ilişkiler).
- **Ek konfigürasyonlar/entegrasyonlar kısa listesi:**
  - Google Workspace mail ayarları (kısa parametre adları).
  - Facebook ve Google OAuth (kısa parametre adları ve account linking notları).
  - Ödeme davranışı: dış link ile paket ödeme akışı + PaymentProvider/adapter soyutlaması (idempotency, click-tracking, admin manual confirmation).
  - Telegram bot admin onayı için veri bütünlüğü ve kimlik doğrulama planı (imza/timestamp/nonce, admin id doğrulama, replay koruması).
  - Medya için optimize ve erişim izin planı (upload pipeline, temp/quarantine, signed URLs, quotas, retention).
  - Arama için incremental ve full reindex (signal -> enqueue -> background job; yönetim komutları; index aliasing stratejisi).
  - SSR & Next.js BFF entegrasyonu: endpoint sözleşmeleri, auth/session paylaşımı, preview token, cache invalidation, idempotency, prod-ready test planı, HMAC doğrulama.
  - Güvenlik & uyumluluk kısa notları: rate-limiting, CAPTCHA opsiyonu, CSRF, secure cookies, CSP, secrets yönetimi (env / secret manager), GDPR/KVKK uyumluluğu (data export & delete).
  - Test ve CI kısa notu: telegram ve ödeme adapter'ları için mock/stub test harness; e2e test ortamı (DB, cache, search, mail stub) tanımlı olacak.
- **SEO ve mobil hız için önerilen Django/ön yüz ayarları** (kısa maddeler):
  - Ana sayfa ve ilan detayları server-side render (dinamik meta ve JSON-LD).
  - Sitemap + robots, canonical, Open Graph, Twitter Card.
  - Görüntüler responsive srcset, WebP/modern formatlar, lazy-loading.
  - Static asset optimizasyon: minifikasyon, brotli/gzip, critical CSS inline.
  - Cache: template fragment cache, HTTP cache headers, CDN/edge cache planı.
  - Lighthouse hedefleri ve ölçüm adımları (LCP, CLS, TTFB).


## Proje kök dizini ağacı (Türkçe dizin/adlar)
```plaintext
apps/
├── api/              
│   └── docs/
│       └── kurallar.md
├── arama/            
│   └── docs/
│       └── kurallar.md
├── audit/            
│   └── docs/
│       └── kurallar.md
├── authn/            
│   └── docs/
│       └── kurallar.md
├── basvurular/       
│   └── docs/
│       └── kurallar.md
├── bff/              
│   └── docs/
│       └── kurallar.md
├── bildirimler/      
│   └── docs/
│       └── kurallar.md
├── blog/             
│   └── docs/
│       └── kurallar.md
├── etiketler/        
│   └── docs/
│       └── kurallar.md
├── favoriler/        
│   └── docs/
│       └── kurallar.md
├── hayvanlar/        
│   └── docs/
│       └── kurallar.md
├── ilanlar/          
│   └── docs/
│       └── kurallar.md
├── kategoriler/      
│   └── docs/
│       └── kurallar.md
├── kullanicilar/     
│   └── docs/
│       └── kurallar.md
├── medya/            
│   └── docs/
│       └── kurallar.md
├── mesajlasma/       
│   └── docs/
│       └── kurallar.md
├── moderation/       
│   └── docs/
│       └── kurallar.md
├── ortak/            
│   └── docs/
│       └── kurallar.md
├── packages/         
│   └── docs/
│       └── kurallar.md
├── payments/         
│   └── docs/
│       └── kurallar.md
├── raporlar/         
│   └── docs/
│       └── kurallar.md
├── telegram_bot/     
│   └── docs/
│       └── kurallar.md
└── __init__.py       
