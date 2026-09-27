# UTEP — Technology Stack

## 1. Genel Teknoloji Yapısı

UTEP, mobil istemci ve REST API tabanlı backend mimarisinden oluşur.

```text
React Native / Expo
        │
        │ HTTP / JSON
        ▼
REST API
        │
        ▼
NestJS / Node.js
        │
        ▼
PostgreSQL
```

Mobil ve backend tarafında ortak ana programlama dili **TypeScript** olacaktır.

---

## 2. Mobil Uygulama

* **Framework:** React Native
* **Platform / Tooling:** Expo
* **Programlama dili:** TypeScript
* **Minimum Android sürümü:** Android 8.0 / API 26
* **Backend iletişimi:** REST API üzerinden HTTP/JSON

Mobil uygulamada daha önce kullanılan TypeScript odaklı geliştirme yaklaşımı mümkün olduğunca korunacaktır.

Mobil taraftaki framework ve kütüphaneler backend teknolojilerinden bağımsız olarak değerlendirilecektir.

---

## 3. Backend

* **Programlama dili:** TypeScript
* **Runtime:** Node.js
* **Framework:** NestJS
* **API yaklaşımı:** REST
* **Veri formatı:** JSON

Backend, UTEP'in etkinlik, başvuru, kapasite, bekleme listesi, katılım ve yetkilendirme gibi iş kurallarını uygulayan ana katman olacaktır.

---

## 4. Veritabanı

* **Database:** PostgreSQL
* **Database erişimi:** Prisma ORM
* **Migration:** Prisma Migrate

UTEP'in ilişkisel domain yapısı nedeniyle ilişkisel veritabanı kullanılacaktır.

Temel veri alanları arasında kullanıcılar, üniversiteler, kulüpler, kulüp üyelikleri, etkinlikler, başvurular ve katılım kayıtları bulunur.

---

## 5. Validation

Backend tarafında gelen verilerin runtime doğrulaması için **Zod** kullanılacaktır.

TypeScript'in compile-time type checking özelliği ile runtime validation birbirinden ayrı tutulacaktır.

---

## 6. Authentication

Kimlik doğrulama için **JWT tabanlı authentication** kullanılacaktır.

Token yapısı:

* Kısa ömürlü **Access Token**
* Uzun ömürlü **Refresh Token**

NestJS tarafında JWT authentication için Passport entegrasyonu kullanılacaktır.

Refresh token mekanizması sayesinde access token süresi dolduğunda kullanıcının tekrar giriş yapması gerekmeyecektir.

---

## 7. Authorization

Yetkilendirme, sistem rolü ve kulüp yöneticiliği ayrımını koruyacaktır.

### System Role

* `USER`
* `ADMIN`

### Club Member Role

* `PRESIDENT`
* `VICE_PRESIDENT`
* `MANAGER`

Kulüp yöneticiliği kullanıcının global rolü değildir.

Bir kullanıcının farklı kulüplerde farklı yöneticilik rolleri bulunabilir.

Etkinlik ve başvuru yönetimi gibi kulüp işlemlerinde kullanıcının ilgili kulübün aktif yöneticisi olup olmadığı kontrol edilecektir.

---

## 8. API Documentation

REST API için **OpenAPI / Swagger** kullanılacaktır.

API endpoint'leri, request/response modelleri ve ilgili API davranışları dokümante edilecektir.

---

## 9. Testing

Backend için NestJS ekosistemiyle uyumlu test altyapısı kullanılacaktır.

Test yaklaşımında özellikle iş kuralları ve kritik akışlar önemlidir:

* Başvuru oluşturma
* Kapasite kontrolü
* Waitlist
* Başvuru durum geçişleri
* Başvuru geri çekme
* Yetki kontrolleri
* Katılım
* Etkinlik lifecycle işlemleri

---

## 10. Local Development

İlk geliştirme ortamı tamamen local olacaktır.

Temel yapı:

```text
Expo / React Native
        │
        ▼
Local NestJS API
        │
        ▼
PostgreSQL
```

PostgreSQL local geliştirme ortamında **Docker** üzerinden çalıştırılacaktır.

Production ortamı ve deployment teknolojileri geliştirme aşamasında ayrıca belirlenecektir.

---

## 11. TypeScript Yaklaşımı

Backend tarafında TypeScript'in strict type checking özellikleri kullanılacaktır.

Özellikle:

* `strict`
* `noUncheckedIndexedAccess`
* `exactOptionalPropertyTypes`

ayarlarının kullanılması hedeflenmektedir.

Ancak mobil projenin TypeScript ayarları backend'e birebir kopyalanmayacak; backend'in ihtiyaçlarına göre düzenlenecektir.

---

## 12. Teknoloji Seçim İlkesi

UTEP'te kullanılacak teknolojiler:

1. Mevcut domain ve business rule'lara uygun olmalıdır.
2. Projenin tek geliştirici tarafından sürdürülebilmesine uygun olmalıdır.
3. Birbirleriyle uyumlu ve stabil sürümlerden seçilmelidir.
4. Gereksiz teknoloji ve abstraction eklenmemelidir.
5. Local geliştirme kolay olmalıdır.
6. Projenin ileride birden fazla üniversiteyi desteklemesine engel oluşturmamalıdır.

Exact package sürümleri proje kurulumu sırasında uyumlu stabil sürümler olarak belirlenecek ve package lock dosyası ile sabitlenecektir.
