# UC03 — Etkinlik Başvurularının ve Katılımın Yönetilmesi

## 1. Amaç

Kulüp yöneticilerinin kendi kulüplerine ait etkinliklerin başvurularını yönetmesini ve etkinliğe katılım durumlarını görüntüleyip gerektiğinde düzeltmesini kapsar.

Bu işlemler yalnızca ilgili kulübün yöneticileri tarafından gerçekleştirilebilir.

Kulüp yöneticisi olmak ayrı bir kullanıcı tipi değildir. Kullanıcı, `ClubMember` üzerinden ilgili kulüpte `PRESIDENT`, `VICE_PRESIDENT` veya `MANAGER` rolüne sahipse o kulübün başvurularını ve katılım kayıtlarını yönetebilir.

---

## 2. Yetki

Başvuru ve katılım yönetimi için kullanıcının:

* giriş yapmış olması,
* ilgili kulübün aktif üyesi olması,
* kulüpte `PRESIDENT`, `VICE_PRESIDENT` veya `MANAGER` rolüne sahip olması

gerekir.

Kullanıcı başka kulüplerin etkinliklerine ait başvuru ve katılım kayıtlarını yönetemez.

Sistem yöneticisi (`ADMIN`) ise sistem genelindeki yönetim yetkileri kapsamında gerekli yönetim işlemlerini gerçekleştirebilir.

---

# 3. API Endpointleri

## 3.1 Etkinlik Başvurularını Listeleme

### `GET /events/{eventId}/applications`

Belirtilen etkinliğe yapılmış başvuruları getirir.

Yanıtta başvuru ile ilgili temel bilgiler bulunabilir:

* başvuru ID'si
* başvuran kullanıcı bilgileri
* başvuru durumu
* başvuru tarihi

Başvurular başvuru tarihine göre sıralanabilir. Bu sıralama özellikle bekleme listesi önceliğinin belirlenmesinde kullanılabilir.

### Yetki

Yalnızca:

* ilgili kulübün `PRESIDENT`
* `VICE_PRESIDENT`
* `MANAGER`

rollerinden birine sahip kullanıcıları ve yetkili `ADMIN` işlemi gerçekleştirebilir.

---

## 3.2 Başvuruyu Kabul Etme

### `POST /applications/{applicationId}/accept`

Bir başvuruyu kabul eder.

Geçerli durum geçişleri:

* `PENDING → ACCEPTED`
* `WAITLISTED → ACCEPTED`
* `REJECTED → ACCEPTED`

`REJECTED → ACCEPTED` geçişi, yöneticinin daha önce verdiği bir kararı düzeltmesi amacıyla kullanılabilir.

### Kapasite kontrolü

Etkinliğin kapasitesi belirlenmişse kabul işlemi sırasında kapasite kontrol edilir.

Kapasite doluysa başvuru doğrudan `ACCEPTED` durumuna geçirilemez.

Kapasite değişiklikleri ve bekleme listesinin işleyişi etkinliğin `applicationType` ve kapasite kurallarına göre uygulanır.

---

## 3.3 Başvuruyu Reddetme

### `POST /applications/{applicationId}/reject`

Bir başvuruyu reddeder.

Geçerli durum geçişleri:

* `PENDING → REJECTED`
* `ACCEPTED → REJECTED`

`ACCEPTED → REJECTED` geçişi, etkinlik başlamadan önce yöneticinin daha önce kabul ettiği bir başvuruyu düzeltmesi amacıyla kullanılabilir.

Başvuru reddedildiğinde başvuru kaydı silinmez ve durumu `REJECTED` olarak korunur.

### Reddetme sebebi

`rejectionReason` bilgisi şimdilik opsiyoneldir (`nullable`).

İleride reddetme sebebinin request body içerisinde zorunlu tutulmasına yönelik bir kural eklenebilir.

---

## 3.4 Başvuruyu Bekleme Listesine Alma

### `POST /applications/{applicationId}/waitlist`

Bir başvuruyu bekleme listesine alır.

Geçerli durum geçişi:

`PENDING → WAITLISTED`

Bekleme listesi kapasitesi sınırlı etkinliklerde kullanılır.

`PUBLIC` etkinliklerde kapasite dolduğunda yeni başvurular sistem tarafından otomatik olarak `WAITLISTED` durumuna alınabilir.

`APPROVAL_REQUIRED` etkinliklerde ise başvuruyu bekleme listesine alma kararı kulüp yöneticisi tarafından verilebilir.

---

# 4. Başvuru Durumları

`Application` için aşağıdaki durumlar kullanılır:

| Durum        | Anlamı                                      |
| ------------ | ------------------------------------------- |
| `PENDING`    | Başvuru yönetici değerlendirmesini bekliyor |
| `ACCEPTED`   | Başvuru kabul edildi                        |
| `REJECTED`   | Başvuru reddedildi                          |
| `WAITLISTED` | Başvuru bekleme listesinde                  |
| `WITHDRAWN`  | Öğrenci başvurusunu geri çekti              |

### Temel durum geçişleri

| Mevcut Durum | Yeni Durumlar                        |
| ------------ | ------------------------------------ |
| `PENDING`    | `ACCEPTED`, `REJECTED`, `WAITLISTED` |
| `WAITLISTED` | `ACCEPTED`                           |
| `ACCEPTED`   | `REJECTED`                           |
| `REJECTED`   | `ACCEPTED`                           |

Öğrencinin başvurusunu geri çekmesi UC01 kapsamında:

`PENDING → WITHDRAWN`

veya

`WAITLISTED → WITHDRAWN`

veya

`ACCEPTED → WITHDRAWN`

şeklinde gerçekleşebilir.

`WITHDRAWN` durumundaki eski kayıt yeniden `ACCEPTED`, `PENDING` veya başka bir aktif duruma geçirilmez.

Öğrenci tekrar başvurursa yeni bir `Application` kaydı oluşturulur.

---

# 5. Başvuru Yönetimi Kuralları

## Başvuru süresi devam ederken

Yeni başvurular alınabilir.

Başvurular ilgili etkinliğin `applicationType` değerine göre otomatik veya yönetici değerlendirmesiyle işlenir.

## Başvuru süresi bittikten sonra

Yeni başvuru alınamaz.

Ancak daha önce oluşturulmuş başvurular yönetici tarafından değerlendirilmeye devam edilebilir.

Örneğin başvuru süresi bittikten sonra:

`PENDING → ACCEPTED`

veya

`PENDING → REJECTED`

işlemleri gerçekleştirilebilir.

## Etkinlik başladıktan sonra

Başvuru kararları değiştirilemez.

Aşağıdaki durum değişiklikleri etkinlik başladıktan sonra yapılamaz:

* `PENDING → ACCEPTED`
* `PENDING → REJECTED`
* `PENDING → WAITLISTED`
* `WAITLISTED → ACCEPTED`
* `ACCEPTED → REJECTED`
* `REJECTED → ACCEPTED`

Öğrencinin başvuruyu geri çekmesiyle ilgili kurallar UC01 kapsamında ayrıca uygulanır.

---

# 6. PUBLIC Başvuru Davranışı

Etkinliğin `applicationType` değeri `PUBLIC` ise başvurular sistem tarafından otomatik olarak değerlendirilir.

### Kapasite belirtilmemişse

Başvuru otomatik olarak:

`ACCEPTED`

durumuna geçirilir.

### Kapasite belirtilmişse

Kapasitede yer varsa:

`ACCEPTED`

Kapasite doluysa:

`WAITLISTED`

durumuna geçirilir.

### Bekleme listesi

Bekleme listesi başvuru zamanına göre önceliklendirilir.

Kabul edilen bir kullanıcı başvurusunu geri çekerse boşalan kontenjan, uygun durumdaki ilk bekleme listesi başvurusuna açılır.

---

# 7. APPROVAL_REQUIRED Başvuru Davranışı

Etkinliğin `applicationType` değeri `APPROVAL_REQUIRED` ise yeni başvurular:

`PENDING`

durumunda oluşturulur.

Kulüp yöneticisi başvuruyu:

* kabul edebilir,
* reddedebilir,
* bekleme listesine alabilir.

Kapasite dolu olduğunda başvurunun otomatik olarak bekleme listesine alınması zorunlu değildir. Yönetici başvurunun durumuna karar verir.

---

# 8. Kapasite ve Bekleme Listesi Kuralları

Kapasitesi belirlenmiş etkinliklerde kabul işlemleri mevcut kabul edilmiş başvuru sayısı dikkate alınarak gerçekleştirilir.

Bir kullanıcı `ACCEPTED` durumundan `WITHDRAWN` durumuna geçtiğinde ilgili kontenjan boşalır.

Boşalan kontenjan için bekleme listesi varsa, öncelik sırasındaki uygun başvuru `ACCEPTED` durumuna geçirilebilir.

Bekleme listesinde öncelik:

`appliedAt`

zamanına göre belirlenir.

Etkinlik kapasitesi UC02'de tanımlanan kurallara göre yönetilir:

* başvuru başlamadan önce artırılabilir veya azaltılabilir,
* başvurular başladıktan sonra azaltılamaz,
* başvurular başladıktan sonra artırılabilir.

---

# 9. Katılım Kayıtlarını Görüntüleme

### `GET /events/{eventId}/attendance`

Belirtilen etkinliğin katılım kayıtlarını getirir.

Katılım kayıtları `Application` üzerinden etkinliğe bağlanır.

Yanıtta örneğin:

* kullanıcı bilgisi
* başvuru durumu
* katılım durumu
* katılım kaydı

gibi bilgiler bulunabilir.

Katılım listesi etkinliğin kabul edilmiş katılımcıları üzerinden oluşturulur.

### Yetki

Yalnızca:

* ilgili kulübün `PRESIDENT`
* `VICE_PRESIDENT`
* `MANAGER`

rollerinden birine sahip kullanıcıları ve yetkili `ADMIN` işlemi gerçekleştirebilir.

---

# 10. Katılım Durumları

`Attendance` için üç durum bulunur:

| Durum          | Anlamı                        |
| -------------- | ----------------------------- |
| `NULL`         | Katılım henüz kesinleşmedi    |
| `ATTENDED`     | Kullanıcı etkinliğe katıldı   |
| `NOT_ATTENDED` | Kullanıcı etkinliğe katılmadı |

### Katılım kaydının oluşturulması

Etkinlik başladığında `ACCEPTED` durumundaki başvurular için `Attendance` kaydı oluşturulur.

Başlangıç durumu:

`NULL`

olur.

Öğrenci etkinlik QR kodunu başarıyla tarattığında:

`NULL → ATTENDED`

geçişi gerçekleşir.

Etkinlik sona erdiğinde `NULL` olarak kalan kayıtlar sistem tarafından:

`NULL → NOT_ATTENDED`

durumuna geçirilir.

---

# 11. Katılım Durumunu Düzeltme

### `PATCH /attendance/{attendanceId}`

Yetkili kulüp yöneticisinin mevcut katılım durumunu düzeltmesini sağlar.

Örneğin:

```json
{
  "status": "NOT_ATTENDED"
}
```

### İzin verilen geçişler

* `ATTENDED → NOT_ATTENDED`
* `NOT_ATTENDED → ATTENDED`

Yönetici tarafından:

* `NULL → ATTENDED`
* `NULL → NOT_ATTENDED`

geçişleri yapılamaz.

`NULL` durumu sistem tarafından katılım sürecinin ilgili aşamalarında yönetilir.

### Etkinlik tamamlandıktan sonra

Katılım düzeltmesi etkinlik `COMPLETED` durumuna geldikten sonra da yapılabilir.

Örneğin etkinlik tamamlandıktan sonra yanlışlık fark edilirse:

`ATTENDED → NOT_ATTENDED`

veya

`NOT_ATTENDED → ATTENDED`

düzeltmesi gerçekleştirilebilir.

### Alternatif endpoint

İleride daha aksiyon odaklı bir API yapısına ihtiyaç duyulursa:

`POST /attendance/{attendanceId}/correct`

yaklaşımı değerlendirilebilir.

MVP kapsamında kullanılan endpoint:

`PATCH /attendance/{attendanceId}`

olacaktır.

---

# 12. İptal Edilen Etkinliklerde Başvuru ve Katılım

Etkinlik `CANCELLED` durumuna geçtiğinde:

* yeni başvuru alınamaz,
* mevcut başvuruların etkinliğe katılım sağlaması mümkün değildir,
* mevcut `Application` kayıtları silinmez,
* `Application` durumlarına `CANCELLED` eklenmez,
* etkinlik başlamadan iptal edildiği için yeni `Attendance` kayıtları oluşturulmaz,
* QR ile katılım işlemi gerçekleştirilemez.

Etkinlik geçmişte `CANCELLED` olarak görüntülenmeye devam eder.

---

# 13. Etkinlik Başladıktan Sonraki Kurallar

Etkinlik başladıktan sonra başvuru kararları değiştirilemez.

Buna karşılık katılım süreci devam eder.

Bu nedenle:

### Başvuru

`PENDING / ACCEPTED / REJECTED / WAITLISTED`

durumları arasında yönetici tarafından yeni karar verilemez.

### Katılım

QR taraması ile:

`NULL → ATTENDED`

gerçekleşebilir.

Etkinlik tamamlandığında `NULL` kalan katılımlar:

`NULL → NOT_ATTENDED`

olarak sonuçlandırılır.

Etkinlik `COMPLETED` olduktan sonra yönetici yalnızca:

`ATTENDED ↔ NOT_ATTENDED`

düzeltmesi yapabilir.

---

# 14. Endpoint Özeti

| İşlem                          | Endpoint                                      | Yetki                                   |
| ------------------------------ | --------------------------------------------- | --------------------------------------- |
| Etkinlik başvurularını listele | `GET /events/{eventId}/applications`          | İlgili kulüp yöneticisi / yetkili ADMIN |
| Başvuruyu kabul et             | `POST /applications/{applicationId}/accept`   | İlgili kulüp yöneticisi / yetkili ADMIN |
| Başvuruyu reddet               | `POST /applications/{applicationId}/reject`   | İlgili kulüp yöneticisi / yetkili ADMIN |
| Başvuruyu bekleme listesine al | `POST /applications/{applicationId}/waitlist` | İlgili kulüp yöneticisi / yetkili ADMIN |
| Katılım listesini görüntüle    | `GET /events/{eventId}/attendance`            | İlgili kulüp yöneticisi / yetkili ADMIN |
| Katılım durumunu düzelt        | `PATCH /attendance/{attendanceId}`            | İlgili kulüp yöneticisi / yetkili ADMIN |

---

# 15. UC03 Kapsamı

UC03 aşağıdaki işlemleri kapsar:

1. Kulüp yöneticisinin etkinlik başvurularını görüntülemesi
2. Başvuruların `PENDING`, `ACCEPTED`, `REJECTED`, `WAITLISTED` ve `WITHDRAWN` durumlarının yönetilmesi
3. `PUBLIC` etkinliklerde otomatik kabul ve bekleme listesi davranışının uygulanması
4. `APPROVAL_REQUIRED` etkinliklerde yönetici değerlendirmesinin yapılması
5. Kapasite ve bekleme listesi kurallarının uygulanması
6. Kabul edilen başvuruların geri çekilmesi sonucunda kontenjanın yeniden değerlendirilmesi
7. Etkinlik başladıktan sonra başvuru kararlarının değiştirilememesi
8. Etkinlik katılım kayıtlarının görüntülenmesi
9. QR ile gerçekleşen katılım sonuçlarının görüntülenmesi
10. `ATTENDED` ve `NOT_ATTENDED` durumlarının yönetici tarafından düzeltilmesi
11. Etkinlik tamamlandıktan sonra katılım düzeltmelerinin yapılabilmesi
12. İptal edilen etkinliklerde başvuru ve katılım kurallarının uygulanması

Öğrencinin etkinliğe başvurması, başvurusunu geri çekmesi ve QR ile katılım sağlaması UC01'de; etkinliğin oluşturulması, düzenlenmesi, tamamlanması ve iptal edilmesi UC02'de; bildirim işlemleri ise UC04'te ele alınır.
