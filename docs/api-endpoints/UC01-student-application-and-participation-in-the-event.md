# UC01 — Öğrencinin Etkinliğe Başvurması ve Katılması

Bu doküman, UC01 kapsamında öğrencinin gerçekleştirebildiği işlemler için gerekli API endpointlerini tanımlar.

> **Not:** Bu aşamada endpointlerin amacı ve temel kullanım şekli belirlenmektedir. Ayrıntılı request/response modelleri, query parameter isimleri, pagination, QR veri formatı ve backend teknolojisine özgü uygulama detayları daha sonraki aşamalarda belirlenecektir.

---

## 1. Etkinlikleri Listeleme

### `GET /events`

Öğrencinin erişebildiği yayınlanmış etkinlikleri listeler.

Kullanım alanları:

* Etkinliklerin genel listelenmesi
* Geçmiş etkinliklerin filtrelenmesi
* İleride eklenecek etkinlik filtreleri

Etkinlik listesi, giriş yapmış kullanıcı için gerekli olduğunda kullanıcının ilgili etkinlikteki başvuru ve katılım durumlarını da gösterebilecek bilgileri içerebilir.

> Geçmiş etkinlikler ekranı için ayrı bir endpoint oluşturulması şu aşamada planlanmamaktadır. Bu ekranın `GET /events` üzerinden filtreleme ile oluşturulması planlanmaktadır. Kullanılacak filtrelerin kesin isimleri daha sonra belirlenecektir.

---

## 2. Etkinlik Detayını Görüntüleme

### `GET /events/{eventId}`

Belirli bir etkinliğin detaylarını getirir.

Giriş yapmış öğrenci açısından, etkinlik bilgilerinin yanında öğrencinin bu etkinlikle ilgili durumunun gösterilebilmesi gerekir.

Örneğin:

* Kendi başvuru durumu
* Kendi katılım durumu

Bu bilgiler backend tarafından ilgili `Event`, `Application` ve gerektiğinde `Attendance` verilerinden oluşturulabilir.

Örneğin öğrenci bir etkinliğin detayını açtığında:

```text
Etkinlik bilgileri
Başvuru durumu: ACCEPTED
Katılım durumu: ATTENDED
```

şeklinde bir bilgi gösterilebilir.

Bu durumlar için mobil uygulamanın ayrı ayrı HTTP istekleri yapması zorunlu değildir. Backend gerekli verileri kendi içerisinde birleştirerek tek response döndürebilir.

---

## 3. Etkinliğe Başvurma

### `POST /events/{eventId}/applications`

Giriş yapmış öğrencinin belirli bir etkinliğe başvurmasını sağlar.

Öğrencinin kimliği request body üzerinden gönderilmez; backend giriş yapan kullanıcı üzerinden öğrenciyi belirler.

Başvurunun başlangıç durumu etkinliğin `applicationType` ve kapasite durumuna göre belirlenir:

* `PUBLIC` + kapasite mevcut → `ACCEPTED`
* `PUBLIC` + kapasite dolu → `WAITLISTED`
* `APPROVAL_REQUIRED` → `PENDING`
* `APPROVAL_REQUIRED` + kapasite dolu → ilgili iş kurallarına göre değerlendirilir

Başvuru oluşturulduğunda `Application` kaydı oluşturulur.

---

## 4. Kendi Başvurusunu Görüntüleme

### `GET /events/{eventId}/application`

Giriş yapmış öğrencinin belirli bir etkinlikteki kendi başvurusunu ve başvuru durumunu getirir.

Başvuru durumları:

* `PENDING`
* `ACCEPTED`
* `REJECTED`
* `WAITLISTED`
* `WITHDRAWN`

Bu endpoint özellikle öğrencinin kendi başvurusunun ayrıntılı durumunu görüntülemesi için kullanılır.

> Etkinlik listesi veya etkinlik detayında özet başvuru durumu gösterilebildiği için bu endpoint her listeleme işleminde kullanılmak zorunda değildir.

---

## 5. Başvuruyu Geri Çekme

### `POST /events/{eventId}/application/withdraw`

Giriş yapmış öğrencinin kendi başvurusunu geri çekmesini sağlar.

Başvuru kaydı silinmez. Başvurunun durumu `WITHDRAWN` olarak değiştirilir.

Geçerli örnekler:

```text
PENDING → WITHDRAWN
WAITLISTED → WITHDRAWN
ACCEPTED → WITHDRAWN
```

`ACCEPTED` durumundaki bir başvurunun geri çekilmesi kapasiteyi boşaltabilir ve ilgili bekleme listesindeki öğrencinin kabul edilmesini sağlayabilir.

`WITHDRAWN` durumundaki eski kayıt korunur. Öğrenci başvuru dönemi hâlâ açıksa yeniden başvurabilir; bu durumda yeni bir `Application` kaydı oluşturulur.

---

## 6. Geçmiş Etkinlikleri Görüntüleme

Öğrenci geçmişte ilişkili olduğu etkinlikleri ve bu etkinliklerdeki katılım durumunu görüntüleyebilir.

Örneğin:

```text
Android Workshop       → ATTENDED
Yapay Zeka Semineri    → NOT_ATTENDED
Kariyer Söyleşisi      → ATTENDED
```

Bu işlem için şu aşamada ayrı bir `/history` veya `/my-events` endpointi oluşturulmamaktadır.

Mevcut:

```text
GET /events
```

endpointinin uygun filtrelerle kullanılması planlanmaktadır.

Kesin filtre isimleri API contract aşamasında belirlenecektir.

Geçmiş etkinliklerde öğrencinin katılım durumu `Attendance` verisinden elde edilir.

---

## 7. QR ile Katılım İşaretleme

### `POST /attendance/scan`

Öğrencinin etkinlik sırasında etkinliğe ait QR kodunu kendi telefonundaki QR okuyucu ile okutmasını sağlar.

Attendance kayıtlarının oluşturulması etkinlik başladığında sistem tarafından otomatik olarak gerçekleştirilir.

Akış:

```text
Etkinlik başlar
       ↓
ACCEPTED öğrenciler belirlenir
       ↓
Bu öğrenciler için Attendance kayıtları oluşturulur
       ↓
Attendance.status = NULL
```

Öğrenci etkinlik sırasında QR kodunu okuttuğunda:

```text
NULL → ATTENDED
```

olur.

QR işlemi sırasında öğrencinin kimliği request içerisinden manuel olarak belirtilmez. Backend giriş yapan kullanıcıyı belirler ve ilgili öğrencinin Attendance kaydı üzerinde işlem yapar.

QR'ın içeriğinin nasıl oluşturulacağı, ne kadar süre geçerli olacağı ve doğrulama mekanizmasının teknik ayrıntıları bu aşamada belirlenmemiştir.

---

## 8. Attendance'ın Otomatik İşlemleri

Aşağıdaki işlemler kullanıcı tarafından çağrılan API endpointleri değildir.

### Etkinlik başladığında

Kabul edilmiş başvurular için Attendance kayıtları otomatik oluşturulur:

```text
ACCEPTED Application
        ↓
Attendance
status = NULL
```

`NULL` değeri, öğrencinin henüz katılımının belirlenmediğini ifade eder.

Bunun temel amacı, etkinlik sırasında QR okutulduğunda başvuru kaydına ayrıca bakılmasına gerek kalmadan mevcut Attendance kaydının:

```text
NULL → ATTENDED
```

olarak güncellenebilmesidir.

### Etkinlik bittikten sonra

Katılımı `ATTENDED` olarak işaretlenmemiş ve hâlâ `NULL` olan Attendance kayıtları sistem tarafından:

```text
NULL → NOT_ATTENDED
```

olarak güncellenir.

Bu işlem de kullanıcı tarafından çağrılan bir endpoint değildir.

---

## 9. UC01 Endpoint Özeti

| İşlem                           | HTTP Method | Endpoint                                 |
| ------------------------------- | ----------- | ---------------------------------------- |
| Etkinlikleri listeleme          | `GET`       | `/events`                                |
| Etkinlik detayını görüntüleme   | `GET`       | `/events/{eventId}`                      |
| Etkinliğe başvurma              | `POST`      | `/events/{eventId}/applications`         |
| Kendi başvurusunu görüntüleme   | `GET`       | `/events/{eventId}/application`          |
| Başvuruyu geri çekme            | `POST`      | `/events/{eventId}/application/withdraw` |
| Geçmiş etkinlikleri görüntüleme | `GET`       | `/events` + filtre                       |
| QR ile katılım işaretleme       | `POST`      | `/attendance/scan`                       |

### UC01 kapsamında endpoint olmayan otomatik işlemler

* Etkinlik başlangıcında `ACCEPTED → Attendance(NULL)`
* Etkinlik bitiminde `NULL → NOT_ATTENDED`
