# UC02 — Etkinliğin Oluşturulması ve Yönetilmesi

## 1. Amaç

Kulüp yöneticilerinin kendi kulüplerine ait etkinlikleri oluşturmasını, yayınlamasını, etkinlik bilgilerini düzenlemesini ve etkinliği tamamlanmış veya iptal edilmiş duruma getirmesini kapsar.

Bu işlemler yalnızca ilgili kulübün yöneticileri tarafından gerçekleştirilebilir.

Kulüp yöneticisi olmak ayrı bir kullanıcı tipi değildir. Kullanıcı, `ClubMember` üzerinden ilgili kulüpte `PRESIDENT`, `VICE_PRESIDENT` veya `MANAGER` rolüne sahipse o kulübün etkinliklerini yönetebilir.

---

## 2. Yetki

Etkinlik yönetimi için kullanıcının:

* giriş yapmış olması,
* ilgili kulübün aktif üyesi olması,
* kulüpte `PRESIDENT`, `VICE_PRESIDENT` veya `MANAGER` rolüne sahip olması

gerekir.

Kullanıcı başka kulüplerin etkinliklerini yönetemez.

Sistem yöneticisi (`ADMIN`) ise sistem genelindeki yönetim yetkileri kapsamında gerekli yönetim işlemlerini gerçekleştirebilir.

---

# 3. API Endpointleri

## 3.1 Etkinlik Oluşturma ve Yayınlama

### `POST /clubs/{clubId}/events`

Belirtilen kulüp adına yeni bir etkinlik oluşturur ve yayınlar.

Etkinlik oluşturma ve yayınlama ayrı işlemler değildir. Başarılı oluşturma sonucunda etkinliğin durumu doğrudan:

`PUBLISHED`

olur.

### Yetki

* İlgili kulübün `PRESIDENT`, `VICE_PRESIDENT` veya `MANAGER` üyesi
* Yetkili `ADMIN`

### Temel bilgiler

Etkinlik için gerekli bilgiler arasında:

* etkinlik adı
* açıklama
* görsel
* etkinlik türü
* başlangıç tarihi/saatı
* bitiş tarihi/saatı
* fiziksel konum veya online bağlantı
* başvuru başlangıç tarihi/saatı
* başvuru bitiş tarihi/saatı
* başvuru tipi
* kapasite

bulunur.

### Sonuç

Başarılı oluşturma sonucunda yeni `Event` kaydı:

`PUBLISHED`

durumunda olur.

Etkinliği oluşturan kullanıcı `createdBy` alanında tutulur.

---

## 3.2 Etkinlik Detayını Görüntüleme

### `GET /events/{eventId}`

Belirli bir etkinliğin detaylarını getirir.

Bu endpoint UC01'deki öğrenci etkinlik görüntüleme işlemiyle ortak kullanılabilir.

Yanıtta etkinlik bilgilerinin yanında, giriş yapmış kullanıcının bu etkinliğe ilişkin kendi başvuru ve katılım durumu da bulunabilir.

Örneğin:

* başvuru yok
* `PENDING`
* `ACCEPTED`
* `WAITLISTED`
* `REJECTED`
* `WITHDRAWN`
* `ATTENDED`
* `NOT_ATTENDED`

Bu bilgiler için mobil uygulamanın ayrı ayrı `Event`, `Application` ve `Attendance` endpointlerine istek göndermesi gerekmez.

---

## 3.3 Etkinlik Bilgilerini Düzenleme

### `PATCH /events/{eventId}`

Yayınlanmış bir etkinliğin değiştirilebilen bilgilerini günceller.

`PATCH` kullanıldığı için yalnızca değiştirilmek istenen alanların gönderilmesi yeterlidir.

### Yetki

Yalnızca:

* ilgili kulübün `PRESIDENT`
* `VICE_PRESIDENT`
* `MANAGER`

rollerinden birine sahip kullanıcıları ve yetkili `ADMIN` işlemleri gerçekleştirebilir.

### Düzenleme kuralları

Etkinlik başlamadan önce bazı etkinlik bilgileri değiştirilebilir.

Etkinlik başladıktan sonra etkinlik üzerinde yönetimsel değişiklik yapılamaz.

### Alan bazlı kurallar

| Alan                      | Kural                                                                                         |
| ------------------------- | --------------------------------------------------------------------------------------------- |
| `name`                    | Yayınlandıktan sonra değiştirilemez                                                           |
| `description`             | Etkinlik başlamadan önce değiştirilebilir                                                     |
| `image`                   | Etkinlik başlamadan önce değiştirilebilir                                                     |
| `eventType`               | Yayınlandıktan sonra değiştirilemez                                                           |
| `location` / `onlineLink` | Etkinlik başlamadan önce değiştirilebilir                                                     |
| `applicationType`         | `applicationStartAt` başlamadan önce değiştirilebilir                                         |
| `capacity`                | Başvuru başlamadan önce artırılabilir/azaltılabilir; başladıktan sonra yalnızca artırılabilir |
| `applicationStartAt`      | Başlamadan önce değiştirilebilir; başladıktan sonra değiştirilemez                            |
| `applicationEndAt`        | Etkinlik başlamadan önce değiştirilebilir                                                     |
| `startAt`                 | Etkinlik başlamadan önce değiştirilebilir                                                     |
| `endAt`                   | Etkinlik başlamadan önce değiştirilebilir                                                     |

Başarılı bir `PATCH` işlemi sonucunda güncellenmiş `Event` bilgisi döndürülür.

Bu nedenle istemcinin güncel etkinliği almak için ayrıca `GET /events/{eventId}` çağırması gerekmez.

---

## 3.4 Etkinliği İptal Etme

### `POST /events/{eventId}/cancel`

Etkinliği iptal eder.

Geçerli durum geçişi:

`PUBLISHED → CANCELLED`

### İptal koşulları

Etkinlik:

* `PUBLISHED` durumunda olmalı,
* henüz başlamamış olmalıdır (`now < startAt`).

Etkinlik başladıktan sonra iptal edilemez.

### İptal edilen etkinlik

İptal edilen etkinlik:

* yeni başvuru alamaz,
* QR ile katılım işlemi gerçekleştirilemez,
* geçmişte oluşturulmuş `Application` kayıtları silinmez,
* `Application` için ayrıca `CANCELLED` durumu oluşturulmaz.

Etkinliğin kendisi `CANCELLED` durumunda tutulur ve geçmişteki etkinlik bilgisi korunur.

İptal edilmeden önce etkinlik başlamadığı için bu senaryoda yeni `Attendance` kaydı oluşturulmaz.

---

## 3.5 Etkinliği Tamamlandı Olarak İşaretleme

### `POST /events/{eventId}/complete`

Etkinliğin tamamlandığını belirtir.

Geçerli durum geçişi:

`PUBLISHED → COMPLETED`

### Tamamlama koşulu

Etkinliğin bitiş zamanı gelmiş olmalıdır:

`now >= endAt`

Etkinlik henüz bitmemişse `COMPLETED` durumuna geçirilemez.

Etkinlik tamamlandıktan sonra:

* yeni başvuru alınamaz,
* başvuru kabul/reddetme/bekleme listesi kararları değiştirilemez.

Katılım kayıtlarının sonuçlandırılması bu yaşam döngüsüyle ilişkilidir.

---

## 3.6 Etkinliği Silme

### `DELETE /events/{eventId}`

Etkinliğin fiziksel olarak veritabanından silinmesini sağlar.

Bu endpoint yalnızca henüz kullanıcı etkileşimi başlamamış etkinliklerde kullanılabilir.

### Silme koşulları

Etkinliğin:

* `PUBLISHED` durumunda olması,
* `applicationStartAt` zamanının henüz gelmemiş olması,
* hiçbir `Application` kaydının bulunmaması

gerekir.

Bu üç koşul birlikte sağlanmıyorsa fiziksel silme yapılamaz.

Özellikle başvuru süreci başlamışsa, hiç başvuru bulunmasa bile etkinlik fiziksel olarak silinmez. Bu durumda etkinlik `CANCELLED` olarak işaretlenebilir.

Herhangi bir `Application` kaydı bulunan etkinlik ise fiziksel olarak silinmez.

Bu yaklaşım, kullanıcı etkileşimi başlamış etkinliklerin geçmişinin korunmasını sağlar.

---

# 4. Etkinlik Durumları

`Event` için MVP kapsamında üç durum bulunur:

| Durum       | Anlamı                                          |
| ----------- | ----------------------------------------------- |
| `PUBLISHED` | Etkinlik yayınlanmış ve normal yaşam döngüsünde |
| `COMPLETED` | Etkinlik tamamlanmış                            |
| `CANCELLED` | Etkinlik iptal edilmiş                          |

Temel yaşam döngüsü:

`PUBLISHED → COMPLETED`

veya

`PUBLISHED → CANCELLED`

Etkinlik oluşturulduğunda doğrudan `PUBLISHED` olur.

MVP kapsamında `DRAFT` durumu bulunmaz.

---

# 5. Etkinlik Düzenleme Kuralları

## İsim

Etkinlik yayınlandıktan sonra değiştirilemez.

## Etkinlik türü

Etkinlik `PHYSICAL` veya `ONLINE` olarak oluşturulur.

Yayınlandıktan sonra etkinlik türü değiştirilemez.

## Diğer bilgiler

Etkinlik başlamadan önce izin verilen alanlar değiştirilebilir.

## Etkinlik başladıktan sonra

Etkinlik üzerinde yönetimsel değişiklik yapılamaz.

Başvuru yönetimindeki:

* `PENDING`
* `ACCEPTED`
* `REJECTED`
* `WAITLISTED`

durumları da etkinlik başladıktan sonra değiştirilemez.

Bu kural UC03'teki başvuru yönetimiyle birlikte uygulanır.

---

# 6. Tarih ve Zaman Kuralları

Etkinlik tarihleri aşağıdaki sırayı sağlamalıdır:

`applicationStartAt < applicationEndAt < startAt < endAt`

Ayrıca başvuru başlangıcı ile etkinlik başlangıcı arasında en az **1 gün** bulunmalıdır:

`startAt - applicationStartAt >= 1 gün`

### Başvuru başlangıcı

`applicationStartAt` başlamadan önce değiştirilebilir.

Başvuru başladıktan sonra değiştirilemez.

### Başvuru bitişi

`applicationEndAt`, etkinlik başlamadan önce değiştirilebilir.

### Etkinlik başlangıç ve bitişi

`startAt` ve `endAt`, etkinlik başlamadan önce değiştirilebilir.

Etkinlik başladıktan sonra değiştirilemez.

---

# 7. Kapasite Kuralları

Kapasite isteğe bağlıdır.

### Kapasite belirtilmemişse

Etkinlik sınırsız kontenjanlı kabul edilir.

### Kapasite belirtilmişse

Başvuruların kabul ve bekleme listesi davranışları kapasiteye göre yürütülür.

Kapasite:

* etkinlik oluşturulurken belirlenebilir,
* başvuru süreci başlamadan önce artırılabilir veya azaltılabilir,
* başvurular başladıktan sonra azaltılamaz,
* başvurular başladıktan sonra artırılabilir.

Kapasite ve başvuruların kabul/bekleme listesi davranışları UC01 ve UC03 kapsamında uygulanır.

---

# 8. Etkinlik Türleri

Etkinlik iki türden biri olabilir:

* `PHYSICAL`
* `ONLINE`

### `PHYSICAL`

Etkinlik fiziksel bir konumda gerçekleştirilir ve `location` bilgisi tutulur.

### `ONLINE`

Etkinlik çevrim içi gerçekleştirilir ve `onlineLink` bilgisi tutulur.

Etkinlik türü yayınlandıktan sonra değiştirilemez.

QR ile katılım gibi etkinlik türüne bağlı katılım ayrıntıları ilgili kullanım durumunda ayrıca ele alınır.

---

# 9. Başvuru Türleri

Etkinlik iki başvuru tipinden biriyle yayınlanabilir:

### `PUBLIC`

Başvurular kapasite ve sistem kurallarına göre otomatik olarak değerlendirilir.

### `APPROVAL_REQUIRED`

Başvurular kulüp yöneticilerinin değerlendirmesine sunulur.

Bu iki başvuru tipinin başvuru davranışları UC01 ve UC03 içerisinde tanımlanır.

Başvuru tipi, başvuru süreci başlamadan önce değiştirilebilir. Başvuru başladıktan sonra değiştirilemez.

---

# 10. Endpoint Özeti

| İşlem                       | Endpoint                          | Yetki                                   |
| --------------------------- | --------------------------------- | --------------------------------------- |
| Etkinlik oluştur ve yayınla | `POST /clubs/{clubId}/events`     | İlgili kulüp yöneticisi / yetkili ADMIN |
| Etkinlik detayını görüntüle | `GET /events/{eventId}`           | Giriş yapmış kullanıcı                  |
| Etkinliği düzenle           | `PATCH /events/{eventId}`         | İlgili kulüp yöneticisi / yetkili ADMIN |
| Etkinliği iptal et          | `POST /events/{eventId}/cancel`   | İlgili kulüp yöneticisi / yetkili ADMIN |
| Etkinliği tamamla           | `POST /events/{eventId}/complete` | İlgili kulüp yöneticisi / yetkili ADMIN |
| Etkinliği sil               | `DELETE /events/{eventId}`        | İlgili kulüp yöneticisi / yetkili ADMIN |

---

# 11. UC02 Kapsamı

UC02 aşağıdaki işlemleri kapsar:

1. Kulüp yöneticisinin etkinlik oluşturması
2. Etkinliğin oluşturulurken yayınlanması
3. Yayınlanmış etkinliğin görüntülenmesi
4. Etkinlik bilgilerinin etkinlik başlamadan önce düzenlenmesi
5. Yayınlandıktan sonra etkinlik adının ve türünün değiştirilememesi
6. Etkinlik kapasitesinin zaman kurallarına göre yönetilmesi
7. Etkinliğin tamamlanması
8. Etkinliğin iptal edilmesi
9. Kullanıcı etkileşimi başlamamış etkinliğin belirli koşullarda silinmesi
10. Etkinlik yönetimindeki yetki kontrolleri
11. Etkinlik tarihleri ve durum geçişlerinin kontrol edilmesi

Başvuru yönetimi UC03'te, öğrencinin başvuru ve katılım süreci UC01'de, bildirim işlemleri ise UC04'te ele alınır.
