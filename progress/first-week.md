# Haftalık Çalışma Özeti

## Hafta 1 — Gereksinimlerden Teknik Altyapıya

**Dönem:** 21–28 Eylül 2026

### Bu Hafta Yapılanlar

Bu hafta UTEP projesinde gereksinim ve domain çalışmalarından gerçek teknik altyapının kurulmasına geçildi.

#### 1. Gereksinim ve Use Case Çalışmaları

* UTEP'in temel amacı ve kapsamı netleştirildi.
* Öğrenci, kulüp yöneticisi ve sistem yöneticisi gibi roller belirlendi.
* Öğrencinin etkinlik keşfetmesi, başvuru yapması ve katılımının takip edilmesi senaryoları oluşturuldu.
* Kulüp yöneticilerinin etkinlik ve başvuruları yönetmesine yönelik use case'ler hazırlandı.
* UC-01, UC-02, UC-03 ve ilgili use case çalışmaları dokümante edildi.
* Use case'ler ile API ihtiyaçları arasında bağlantı kurulmaya başlandı.

[UC- 01, 02, 03](./docs/use-cases)

#### 2. Domain Model

Temel domain varlıkları netleştirildi:

* User
* University
* Faculty
* Department
* Club
* ClubMember
* Event
* Application
* Attendance

Ayrıca kullanıcı rolleri, kulüp üyelik rolleri, etkinlik ve başvuru arasındaki ilişkiler üzerinde çalışıldı.

[domain model](./docs/domain-model.md)

#### 3. İş Kuralları

Etkinlik ve başvuru sisteminin temel davranışları ayrıntılı şekilde tanımlandı.

Özellikle:

* PUBLIC ve APPROVAL_REQUIRED başvuru türleri
* Kapasiteli / kapasitesiz etkinlikler
* PENDING, ACCEPTED, REJECTED, WAITLISTED ve WITHDRAWN durumları
* Başvuru durum geçişleri
* Bekleme listesi
* Kapasite artırımı
* Başvuru son tarihi
* Başvuru geri çekme
* Aynı etkinliğe tekrar başvuru
* Etkinlik başladıktan sonra yapılabilecek işlemler
* Attendance'ın Application'dan ayrı tutulması

netleştirildi.

[business rules](./docs/business-rules/event-and-application-business-rules.md)

#### 4. API Tasarımının Başlangıcı

Use case'lerden hareketle API endpoint tasarımına geçildi.

* UC-01 için endpoint çalışmaları
* UC-02 için endpoint çalışmaları
* UC-03 için endpoint çalışmaları

hazırlandı ve iş kurallarıyla tutarlılıkları kontrol edilmeye başlandı.

[api endpoint çalışmaları](./docs/api-endpoints)

#### 5. Teknoloji Kararları

Backend ve mobile için temel teknoloji stack'i belirlendi.

**Mobile:**

* React Native
* Expo
* TypeScript
* Android API 26+

[tech stack](./docs/technology-stack.md)

**Backend:**

* Node.js 22 LTS
* NestJS 11
* REST API
* PostgreSQL
* Prisma 7
* Zod 4
* JWT
* Access / Refresh Token
* Passport
* Swagger / OpenAPI

Geliştirme ortamında PostgreSQL'in Docker üzerinden çalıştırılmasına karar verildi.

[tech stack](./docs/technology-stack.md)

#### 6. Backend Projesi

`utep-backend` GitHub reposu oluşturuldu ve NestJS projesi başlatıldı.

* NestJS projesi oluşturuldu.
* Projenin Git yapısı düzenlendi.
* `.gitignore` oluşturuldu.
* `node_modules` gibi dosyaların Git'e dahil edilmesi engellendi.
* NestJS uygulamasının lokal olarak çalıştığı doğrulandı.
* `http://localhost:3000` üzerinden temel API yanıtı test edildi.

[utep backend](https://github.com/Hakan-kuru/utep-backend)

#### 7. PostgreSQL ve Docker

Local geliştirme ortamı hazırlandı.

* PostgreSQL Docker container'ı oluşturuldu.
* `utep-postgres` container'ı çalıştırıldı.
* `utep` veritabanı oluşturuldu.
* PostgreSQL için kalıcı Docker volume oluşturuldu.
* Backend'in local PostgreSQL veritabanına erişimi doğrulandı.

#### 8. Prisma

Prisma backend'e dahil edildi.

* Prisma 7 kuruldu.
* Prisma Client kuruldu.
* `prisma/schema.prisma` oluşturuldu.
* Prisma config oluşturuldu.
* `DATABASE_URL` üzerinden PostgreSQL bağlantısı yapılandırıldı.
* `prisma validate` başarıyla çalıştırıldı.
* Veritabanı bağlantısı test edildi.
* Veritabanının henüz boş olduğu doğrulandı.

Database tablolarının henüz oluşturulmaması bilinçli olarak sonraki aşamaya bırakıldı.

[tech stack](./docs/technology-stack.md)

#### 9. Mobile Projesi

`utep-mobile` GitHub reposu oluşturuldu.

* Expo projesi oluşturuldu.
* Expo SDK 57 kullanıldı.
* Expo Router yapısı oluşturuldu.
* `src/app` tabanlı yapı hazırlandı.
* TypeScript altyapısı hazırlandı.
* Expo uygulamasının Metro üzerinde başarıyla çalıştığı doğrulandı.

[utep mobile](https://github.com/Hakan-kuru/utep-mobile)

### Haftanın Son Durumu

UTEP artık yalnızca bir fikir ve dokümantasyon projesi olmaktan çıkarak gerçek geliştirme ortamına geçti.

**Tamamlanan ana aşamalar:**

> Gereksinimler → Use Case'ler → Domain Model → İş Kuralları → API tasarımının başlangıcı → Backend kurulumu → PostgreSQL → Prisma → Mobile scaffold

### Sonraki Hafta

Bir sonraki aşamada:

1. Yetki ve state kurallarının son tutarlılık kontrolü
2. Backend architecture
3. Prisma database schema
4. Database migration
5. API contract'ın tamamlanması
6. Mobile architecture

üzerinde çalışılması planlanmaktadır.
