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
- Tüm uygulamalar mümkün olduğunca hazır bileşenler kullanılarak sağlanacak; özel kod minimumda tutulacak.  
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

**Dizin yapısı** Bu dosya, `apps/` dizini altındaki uygulama ve dokümantasyon klasörlerinin şemasını test ve referans amacıyla listeler
Directory: apps/
├── api/              
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the api app
├── arama/            
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the arama app
├── audit/            
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the audit app
├── authn/            
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the authn app
├── basvurular/       
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the basvurular app
├── bff/              
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the bff app
├── bildirimler/      
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the bildirimler app
├── blog/             
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the blog app
├── etiketler/        
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the etiketler app
├── favoriler/        
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the favoriler app
├── hayvanlar/        
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the hayvanlar app
├── ilanlar/          
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the ilanlar app
├── kategoriler/      
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the kategoriler app
├── kullanicilar/     
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the kullanicilar app
├── medya/            
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the medya app
├── mesajlasma/       
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the mesajlasma app
├── moderation/       
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the moderation app
├── ortak/            
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the ortak app
├── packages/         
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the packages app
├── payments/         
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the payments app
├── raporlar/         
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the raporlar app
├── telegram_bot/     
│   └── docs/
│       └── kurallar.md    # Rules and documentation for the telegram_bot app
└── __init__.py       

