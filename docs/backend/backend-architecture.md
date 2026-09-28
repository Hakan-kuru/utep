# UTEP Backend Architecture

## 1. Amaç

Bu doküman, UTEP backend uygulamasının genel mimari yapısını, modül organizasyonunu ve temel teknik sorumluluklarını tanımlar.

Backend:

* Node.js
* NestJS
* TypeScript
* PostgreSQL
* Prisma
* REST API

teknolojileri kullanılarak geliştirilecektir.

Bu dokümanın amacı, backend geliştirmesi sırasında modüllerin ve sorumlulukların tutarlı şekilde ilerlemesini sağlamaktır.

---

# 2. Mimari Yaklaşım

UTEP backend'de **feature-based modular architecture** kullanılacaktır.

Uygulama teknik katmanlara göre tamamen ayrılmak yerine, temel iş alanlarına göre modüllere ayrılacaktır.

Temel yaklaşım:

```text
HTTP Request
     ↓
Controller
     ↓
Service
     ↓
Prisma
     ↓
PostgreSQL
```

NestJS `Module` yapısı, her iş alanının sınırlarını belirlemek için kullanılacaktır.

---

# 3. Neden Feature-Based Architecture?

UTEP'in domain'i belirgin iş alanlarına ayrılmaktadır:

* Authentication
* Users
* Universities
* Clubs
* Events
* Applications
* Attendance

Bu nedenle backend'in:

```text
controllers/
services/
repositories/
entities/
```

gibi tamamen teknik katmanlara ayrılması yerine iş alanlarına göre organize edilmesi tercih edilmiştir.

Örneğin:

```text
events/
applications/
attendance/
```

modülleri kendi iş kurallarına yakın tutulacaktır.

Bu yaklaşım:

* Kodun bulunabilirliğini artırır.
* İş kurallarının ilgili domain'e yakın olmasını sağlar.
* Modüllerin bağımsız geliştirilmesini kolaylaştırır.
* Projenin büyümesi durumunda modül sınırlarının korunmasını kolaylaştırır.

---

# 4. Genel Backend Yapısı

Backend'in başlangıç yapısı aşağıdaki şekilde planlanmaktadır:

```text
src/
├── app.module.ts
│
├── auth/
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── auth.module.ts
│   ├── dto/
│   ├── guards/
│   └── strategies/
│
├── users/
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── users.module.ts
│   └── dto/
│
├── universities/
│   ├── universities.controller.ts
│   ├── universities.service.ts
│   ├── universities.module.ts
│   └── dto/
│
├── clubs/
│   ├── clubs.controller.ts
│   ├── clubs.service.ts
│   ├── clubs.module.ts
│   └── dto/
│
├── events/
│   ├── events.controller.ts
│   ├── events.service.ts
│   ├── events.module.ts
│   └── dto/
│
├── applications/
│   ├── applications.controller.ts
│   ├── applications.service.ts
│   ├── applications.module.ts
│   └── dto/
│
├── attendance/
│   ├── attendance.controller.ts
│   ├── attendance.service.ts
│   ├── attendance.module.ts
│   └── dto/
│
├── common/
│   ├── decorators/
│   ├── guards/
│   ├── filters/
│   └── interceptors/
│
└── prisma/
    ├── prisma.module.ts
    └── prisma.service.ts
```

Bu yapı başlangıç mimarisidir.

İhtiyaç ortaya çıkmadıkça yeni abstraction veya klasör oluşturulmayacaktır.

---

# 5. Modül Sorumlulukları

## 5.1. Auth

`auth` modülü kimlik doğrulama işlemlerinden sorumludur.

Temel sorumluluklar:

* Login
* Register
* Access token
* Refresh token
* Token doğrulama
* Authentication strategy'leri

Auth, kullanıcı domain'inden ayrı tutulur.

```text
Auth ≠ User
```

Authentication kullanıcının kim olduğunu doğrularken, `users` modülü kullanıcı verilerinin yönetiminden sorumludur.

---

# 6. Users

`users` modülü kullanıcı domain'inden sorumludur.

Örneğin:

* Kullanıcı bilgilerinin yönetilmesi
* Kullanıcı profil bilgilerinin alınması
* Kullanıcıya ait domain işlemleri

burada bulunabilir.

Sistem yöneticisi rolü gibi global kullanıcı yetkileri User domain'i ile ilişkilidir.

Kulüp yöneticiliği ise User'ın doğrudan bir özelliği değildir.

Kulüp yöneticiliği `ClubMember` üzerinden belirlenir.

---

# 7. Universities

`universities` modülü üniversite ve akademik yapı ile ilgili domain işlemlerini kapsar.

Domain içerisindeki:

```text
University
    ↓
Faculty
    ↓
Department
```

ilişkileri bu domain altında yönetilebilir.

Bu yapı ileride UTEP'in birden fazla üniversitede kullanılabilmesine olanak sağlayacak şekilde tasarlanacaktır.

---

# 8. Clubs

`clubs` modülü kulüp domain'inden sorumludur.

Temel sorumluluklar:

* Kulüp bilgileri
* Kulüp yönetimi
* Kulüp üyelik ilişkileri
* Kulüp yöneticilik rolleri
* Kulübe ait etkinliklerin yönetim bağlamı

Kulüp yöneticisi ayrı bir User tipi olarak modellenmez.

Bir kullanıcının kulüp üzerindeki yöneticilik yetkisi:

```text
User
  ↓
ClubMember
  ↓
role
```

ilişkisi üzerinden belirlenir.

Geçerli kulüp yöneticisi rolleri:

```text
PRESIDENT
VICE_PRESIDENT
MANAGER
```

---

# 9. Events

`events` modülü etkinlik domain'inden sorumludur.

Temel sorumluluklar:

* Etkinlik oluşturma
* Etkinliği yayınlama
* Etkinlik bilgilerini düzenleme
* Kapasite yönetimi
* Etkinlik iptali
* Etkinlik tamamlanması
* Etkinlik yaşam döngüsü

Event status'leri:

```text
PUBLISHED
COMPLETED
CANCELLED
```

Etkinlik oluşturma ve yayınlama tek işlem olduğundan kalıcı:

```text
DRAFT
CREATED
```

status'leri bulunmaz.

---

# 10. Applications

`applications` modülü etkinlik başvurularından sorumludur.

Temel sorumluluklar:

* Başvuru oluşturma
* Başvuru durumlarının yönetimi
* Başvuru geri çekme
* Kapasite kontrolü
* Bekleme listesi
* Başvuru state transition'ları
* PUBLIC başvurularının otomatik sonuçlandırılması
* APPROVAL_REQUIRED başvurularının yönetici tarafından değerlendirilmesi

Application status'leri:

```text
PENDING
ACCEPTED
REJECTED
WAITLISTED
WITHDRAWN
```

Application state transition'ları domain kurallarına uygun şekilde uygulanacaktır.

Örneğin:

```text
PENDING → ACCEPTED
PENDING → REJECTED
PENDING → WAITLISTED
PENDING → WITHDRAWN

WAITLISTED → ACCEPTED
WAITLISTED → WITHDRAWN

ACCEPTED → REJECTED
ACCEPTED → WITHDRAWN

REJECTED → ACCEPTED
```

`WAITLISTED → ACCEPTED` kapasite açıldığında sistem tarafından otomatik olarak gerçekleştirilir.

---

# 11. Attendance

`attendance` modülü etkinlik katılımından sorumludur.

Attendance, Application'ın alt durumu olarak modellenmez.

Bunlar iki ayrı domain kavramıdır:

```text
Application ≠ Attendance
```

Application:

```text
Öğrenci etkinliğe kabul edildi mi?
```

sorusunu temsil eder.

Attendance:

```text
Öğrenci etkinliğe gerçekten katıldı mı?
```

sorusunu temsil eder.

Attendance durumları:

```text
NULL
ATTENDED
NOT_ATTENDED
```

olarak modellenir.

Etkinlik başladığında `ACCEPTED` öğrenciler için Attendance kayıtları oluşturulabilir.

QR katılımı sonucunda:

```text
NULL → ATTENDED
```

gerçekleşir.

Etkinlik tamamlandığında katılımı kesinleşmemiş kayıtlar:

```text
NULL → NOT_ATTENDED
```

olarak sonuçlandırılabilir.

Yönetici gerekli durumlarda:

```text
ATTENDED ↔ NOT_ATTENDED
```

düzeltmesi yapabilir.

---

# 12. Prisma

Prisma, PostgreSQL ile backend arasındaki veri erişim katmanı olarak kullanılacaktır.

Prisma uygulama içerisinde merkezi bir servis üzerinden yönetilecektir:

```text
prisma/
├── prisma.module.ts
└── prisma.service.ts
```

Uygulamadaki servisler doğrudan yeni `PrismaClient` instance'ları oluşturmayacaktır.

Bunun yerine NestJS dependency injection üzerinden `PrismaService` kullanılacaktır.

Örneğin:

```text
EventsService
      ↓
PrismaService
      ↓
PostgreSQL
```

şeklinde bir akış kullanılacaktır.

---

# 13. Controller Sorumluluğu

Controller'ların temel sorumluluğu HTTP katmanını yönetmektir.

Controller:

* HTTP request'i alır.
* Authentication / authorization sonuçlarını kullanır.
* Request DTO'larını alır.
* Service'i çağırır.
* HTTP response döndürür.

Business logic controller içerisinde tutulmaz.

Örneğin aşağıdaki işlemler controller içerisinde uygulanmamalıdır:

```text
Kapasite hesaplama
Waitlist sıralama
Application state transition
Etkinlik yetki kuralları
```

Bunlar ilgili service/domain mantığında ele alınacaktır.

Temel akış:

```text
HTTP Request
     ↓
Controller
     ↓
Service
     ↓
Database
```

---

# 14. Service Sorumluluğu

Service'ler ilgili feature'ın uygulama ve domain işlemlerinden sorumludur.

Örneğin `ApplicationsService`:

```text
PUBLIC kontrolü
APPROVAL_REQUIRED kontrolü
Kapasite kontrolü
Aktif başvuru kontrolü
State transition
Waitlist işlemleri
```

gibi Application domain kurallarını uygular.

Service'ler yalnızca CRUD işlemleri yapan ince wrapper'lar olarak tasarlanmayacaktır.

İş kurallarının önemli bölümü service/domain seviyesinde bulunacaktır.

---

# 15. Repository Abstraction

Başlangıç aşamasında her entity için ayrı repository abstraction oluşturulmayacaktır.

Örneğin:

```text
IApplicationRepository
ApplicationRepository
PrismaApplicationRepository
```

gibi bir abstraction yalnızca gerçek bir ihtiyaç ortaya çıkarsa değerlendirilecektir.

İlk aşamada:

```text
Service
   ↓
PrismaService
```

yaklaşımı kullanılacaktır.

Bunun amacı:

* Gereksiz abstraction oluşturmamak
* Kod miktarını azaltmak
* NestJS + Prisma kullanımını sade tutmak
* Küçük bir ekip/proje için gereksiz katmanlardan kaçınmak

olacaktır.

---

# 16. Common

`common` klasörü bütün modüller tarafından gerçekten ortak kullanılan altyapıları içerecektir.

Örneğin:

```text
common/
├── decorators/
├── guards/
├── filters/
└── interceptors/
```

Buraya:

* Current user decorator
* Ortak authorization guard'ları
* Global exception filter
* Ortak interceptor'lar

gibi yapılar alınabilir.

Domain'e özel kodlar `common` altında tutulmayacaktır.

Örneğin:

```text
ApplicationService
EventService
WaitlistLogic
```

common içerisine taşınmayacaktır.

---

# 17. Authorization

Authentication ve authorization birbirinden ayrılacaktır.

### Authentication

```text
"Bu kullanıcı kim?"
```

sorusunu çözer.

Auth/JWT altyapısı tarafından yönetilir.

### Authorization

```text
"Bu kullanıcı bu işlemi yapabilir mi?"
```

sorusunu çözer.

Örneğin:

```text
PATCH /events/42
```

isteğinde sadece kullanıcının login olması yeterli değildir.

Sistem:

```text
currentUser
    ↓
ClubMember
    ↓
Event.club
    ↓
PRESIDENT / VICE_PRESIDENT / MANAGER
```

ilişkisini kontrol etmelidir.

Kullanıcı başka bir kulübün yöneticisiyse ilgili etkinliği yönetemez.

---

# 18. Modüller Arası Bağımlılık

Modüller birbirlerinin iç implementasyonlarına mümkün olduğunca doğrudan bağımlı olmayacaktır.

Örneğin:

```text
Applications
    ↓
Events
```

ilişkisi gerektiğinde Events modülünün public service/interface davranışı üzerinden kurulmalıdır.

Bir modülün başka bir modülün:

```text
controller
private helper
internal implementation
```

detaylarına erişmesi tercih edilmez.

Amaç modül sınırlarını korumaktır.

---

# 19. Domain Kurallarının Merkezi Olması

UTEP'teki kritik state transition ve kapasite kuralları endpointlere dağılmamalıdır.

Örneğin:

```text
ACCEPTED → WITHDRAWN
```

sonrasında kapasite açılması ve waitlist'in işlenmesi:

* Öğrenci endpointi
* Yönetici endpointi
* Kapasite endpointi

gibi farklı noktalarda ayrı ayrı uygulanmamalıdır.

Tek bir domain kuralı olarak ele alınmalıdır.

Böylece aynı davranış farklı endpointlerden tetiklense bile sonuç tutarlı olur.

---

# 20. Transaction Kullanımı

Birden fazla verinin birlikte değiştirilmesi gereken işlemlerde database transaction kullanılacaktır.

Örneğin:

```text
ACCEPTED → WITHDRAWN
        ↓
Kapasite açıldı
        ↓
WAITLISTED → ACCEPTED
```

gibi işlemler tek bir tutarlı işlem olarak ele alınmalıdır.

Benzer şekilde kapasite artırıldığında:

```text
Event.capacity
       +
WAITLISTED → ACCEPTED
```

işlemleri arasında veri tutarlılığı korunmalıdır.

Transaction sınırları implementation aşamasında ilgili use case'e göre belirlenecektir.

---

# 21. API Katmanı

Backend REST API olarak geliştirilecektir.

Endpointler use case'lerde tanımlanan domain davranışlarına göre oluşturulacaktır.

Örnek:

```text
POST /clubs/{clubId}/events
GET /events/{eventId}
PATCH /events/{eventId}

POST /events/{eventId}/applications
POST /events/{eventId}/application/withdraw

GET /events/{eventId}/applications
POST /applications/{applicationId}/accept
POST /applications/{applicationId}/reject
POST /applications/{applicationId}/waitlist

GET /events/{eventId}/attendance
PATCH /attendance/{attendanceId}
```

Kesin request/response contractları ayrı API contract dokümanında belirlenecektir.

---

# 22. Validation

API'ye gelen kullanıcı girdileri controller/API katmanında doğrulanacaktır.

Validation yaklaşımı ayrı API contract aşamasında kesinleştirilecek olsa da temel hedef:

```text
Invalid Request
       ↓
Validation
       ↓
Business Logic'e ulaşmadan hata
```

şeklindedir.

Business rule validation ise yalnızca DTO validation'a bırakılmayacaktır.

Örneğin:

```text
capacity >= acceptedCount
```

gibi kurallar domain/service seviyesinde kontrol edilmelidir.

---

# 23. Error Handling

Backend ortak ve tutarlı bir error response yapısı kullanacaktır.

Global exception handling için NestJS'in exception mekanizması kullanılabilir.

Domain/business rule hataları ile:

```text
Validation
Authentication
Authorization
Not Found
Conflict
```

gibi HTTP seviyesindeki hatalar birbirinden ayrılacaktır.

Kesin error response formatı API contract aşamasında belirlenecektir.

---

# 24. Authentication ve Authorization Akışı

Genel istek akışı:

```text
Client
  ↓
JWT Authentication
  ↓
Current User
  ↓
Authorization
  ↓
Controller
  ↓
Service
  ↓
Prisma
  ↓
PostgreSQL
```

Örneğin bir kulüp yöneticisinin etkinlik güncellemesi:

```text
PATCH /events/42
        ↓
JWT doğrula
        ↓
currentUser belirle
        ↓
Event #42'yi bul
        ↓
Event.club'u bul
        ↓
ClubMember rolünü kontrol et
        ↓
Yetkili mi?
   ├── Hayır → hata
   └── Evet
          ↓
     EventsService
          ↓
       Prisma
```

---

# 25. Backend ile Domain Dokümantasyonu İlişkisi

Backend implementation sırasında temel referanslar:

1. Gereksinimler
2. Use Case'ler
3. Domain Model
4. İş Kuralları
5. Bu Architecture dokümanı
6. API Contract

olacaktır.

Kod ile dokümantasyon arasında çelişki oluştuğunda önce domain kararının gerçekten değişip değişmediği kontrol edilmelidir.

Kodun mevcut davranışı tek başına yeni bir business rule olarak kabul edilmemelidir.

---

# 26. Kapsam Dışı

Bu doküman şu konuları henüz detaylandırmaz:

* Prisma modellerinin kesin yapısı
* Database migration'ları
* Exact request/response DTO'ları
* HTTP status code'larının tamamı
* Error response JSON formatı
* JWT token payload'ının kesin yapısı
* Refresh token storage stratejisi
* Pagination
* Rate limiting
* Caching
* Background job altyapısı
* Notification sistemi
* Ban/engelleme sistemi
* Production deployment
* CI/CD

Bu konular ilgili teknik tasarım aşamalarında ayrıca belirlenecektir.

---

# 27. Mimari Karar Özeti

| Konu                   | Karar                                 |
| ---------------------- | ------------------------------------- |
| Backend                | NestJS + TypeScript                   |
| Mimari yaklaşım        | Feature-based modular architecture    |
| API                    | REST                                  |
| Database               | PostgreSQL                            |
| ORM                    | Prisma                                |
| Dependency Injection   | NestJS DI                             |
| Controller             | HTTP/API katmanı                      |
| Service                | Business/application logic            |
| Prisma erişimi         | Merkezi `PrismaService`               |
| Repository abstraction | Başlangıçta kullanılmayacak           |
| Authentication         | JWT tabanlı                           |
| Authorization          | ClubMember + role üzerinden           |
| Validation             | API/DTO + domain kuralları            |
| Error handling         | Ortak NestJS exception mekanizması    |
| Transaction            | Çok adımlı kritik domain işlemlerinde |
| Application            | Ayrı modül                            |
| Attendance             | Ayrı modül                            |
| Event                  | Ayrı modül                            |
| Gereksiz abstraction   | Kullanılmayacak                       |

---

# 28. Sonraki Aşama

Backend architecture belirlendikten sonra sıradaki teknik aşama:

```text
Backend Architecture
        ↓
Database Model
        ↓
Prisma Schema
        ↓
Migration
        ↓
PostgreSQL tabloları
```

olacaktır.

Database model oluşturulurken özellikle aşağıdaki domain ilişkileri detaylandırılacaktır:

```text
User
University
Faculty
Department
Club
ClubMember
Event
Application
Attendance
```

Bu aşamada:

* Primary key
* Foreign key
* Unique constraint
* Enum
* Index
* Nullable alanlar
* Cascade/restrict davranışları
* Application state yapısı
* Attendance ilişkisi

kesinleştirilecektir.
