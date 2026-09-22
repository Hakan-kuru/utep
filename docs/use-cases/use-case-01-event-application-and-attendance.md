# Use Case 01 — Öğrencinin Etkinliğe Başvurması ve Katılması

## 1. Amaç

Bu use case, öğrencinin bir etkinliği görüntülemesi, etkinliğe başvurması, başvuru sonucunu takip etmesi, başvurusunu geri çekebilmesi ve etkinliğe katılımının sisteme kaydedilmesini tanımlar.

Bu akış kapsamında:

* Etkinliğe başvurma
* Başvuru durumunun belirlenmesi
* Kontenjan ve bekleme listesi
* Başvurunun geri çekilmesi (`WITHDRAWN`)
* Geri çekilen başvuru sonrası tekrar başvurma
* Etkinliğe katılım
* QR kod ile katılım (fiziksel etkinliklerde)
* Katılım kaydının yönetici tarafından düzeltilmesi

ele alınır.

---

## 2. Aktörler

### Ana Aktör

* Öğrenci

### İlgili Aktörler

* Kulüp yöneticisi
* Sistem

---

## 3. Ön Koşullar

Öğrencinin etkinliğe başvurabilmesi için:

* Sisteme giriş yapmış olması gerekir.
* Etkinliğin yayınlanmış olması gerekir.
* Etkinliğin iptal edilmemiş olması gerekir.
* Başvuru başlangıç tarihinin gelmiş olması gerekir.
* Başvuru bitiş tarihi geçmemiş olmalıdır.
* Öğrencinin aynı etkinlikte aktif bir başvurusu bulunmamalıdır.

> **Not:** Öğrencinin daha önceki başvurusu `WITHDRAWN` durumundaysa ve başvuru süreci hâlâ açıksa yeni bir başvuru oluşturabilir. Detaylar için bkz. Bölüm 7.

---

# 4. Etkinliğe Başvurma

Öğrenci etkinlik detaylarını görüntüler ve başvuru koşullarını karşılıyorsa etkinliğe başvurabilir.

Başvurunun ilk durumu, etkinliğin başvuru türüne ve kapasite durumuna göre belirlenir.

Başvuru türleri:

* `PUBLIC`
* `PRIVATE`

### PUBLIC etkinliklerde

* Kapasite belirtilmemişse başvuru `PENDING` durumundan `ACCEPTED` durumuna otomatik olarak geçer.
* Kapasite belirtilmiş ve henüz dolmamışsa başvuru `PENDING` durumundan `ACCEPTED` durumuna otomatik olarak geçer.
* Kapasite dolmuşsa başvuru `PENDING` durumundan `WAITLISTED` durumuna otomatik olarak geçer.

### PRIVATE etkinliklerde

* Kapasite belirtilmemişse başvuru `PENDING` durumuna geçer ve yönetici tarafından değerlendirilir.
* Kapasite belirtilmiş olsa bile başvuru süresi devam ettiği sürece başvuru `PENDING` durumuna geçer. Kapasitenin dolmuş olması yeni başvuruyu engellemez.
* Başvuru süresi sona erdikten sonra yönetici başvuruları `ACCEPTED`, `WAITLISTED` veya `REJECTED` olarak sonuçlandırabilir.
* Kapasite bulunmayan PRIVATE etkinliklerde `WAITLISTED` kullanılmaz.

---

## 4.1. Herkese Açık Etkinlik

Etkinlik `PUBLIC` ise yönetici onayı gerekmez.

### Kapasite belirtilmemişse

Başvuru:

```text
Başvuru
   ↓
PENDING
   ↓
ACCEPTED
```

durumuna otomatik olarak geçer.

### Kapasite belirtilmiş ve kontenjan uygunsa

Başvuru:

```text
Başvuru
   ↓
PENDING
   ↓
ACCEPTED
```

durumuna otomatik olarak geçer.

### Kapasite dolmuşsa

Yeni başvuru:

```text
Başvuru
   ↓
PENDING
   ↓
WAITLISTED
```

durumuna otomatik olarak geçer.

Bekleme listesine alınan kullanıcıların sırası başvuru zamanına göre belirlenir.

Yönetici onayı gerekmez.

---

## 4.2. Özel Etkinlik

Etkinlik `PRIVATE` ise başvurular yönetici tarafından değerlendirilir.

Başvuru süresi devam ederken:

```text
Başvuru
   ↓
PENDING
```

durumuna geçer.

Kapasitenin dolmuş olması, başvuru süresi devam ederken öğrencinin `PENDING` durumuna geçmesini engellemez.

Başvuru süresi sona erdikten sonra yönetici başvuruları değerlendirir:

```text
PENDING
   ├──→ ACCEPTED
   ├──→ WAITLISTED
   └──→ REJECTED
```

`WAITLISTED`, yalnızca kapasitesi bulunan PRIVATE etkinliklerde yöneticinin uygun gördüğü yedek adaylar için kullanılır.

Kapasitesi bulunmayan PRIVATE etkinliklerde yönetici başvuruyu:

```text
PENDING
   ├──→ ACCEPTED
   └──→ REJECTED
```

şeklinde sonuçlandırır.

Bu işlemler **Use Case 03 — Kulübün Etkinlik Başvurularını Yönetmesi** kapsamında ele alınır.

---

# 5. Kontenjan

Etkinlik oluşturulurken kapasite belirtilmesi isteğe bağlıdır.

## 5.1. Kapasite Belirtilmemişse

Etkinlikte kapasite sınırı bulunmaz.

Başvurular kapasite nedeniyle bekleme listesine alınmaz.

`PUBLIC` etkinliklerde başvurular uygunluk kontrolünden sonra otomatik olarak `ACCEPTED` olur.

`PRIVATE` etkinliklerde başvurular `PENDING` durumunda tutulur ve yönetici tarafından `ACCEPTED` veya `REJECTED` olarak sonuçlandırılır.

---

## 5.2. Kapasite Dolmuşsa

Kapasitenin dolmuş olması, her etkinlik türünde yeni başvurunun aynı şekilde sonuçlanacağı anlamına gelmez.

### PUBLIC etkinliklerde

Kapasite dolduktan sonra yeni başvurular:

```text
PENDING
   ↓
WAITLISTED
```

durumuna otomatik olarak geçer.

### PRIVATE etkinliklerde

Başvuru süresi devam ettiği sürece kapasitenin dolmuş olması yeni başvuruyu engellemez.

Yeni başvuru:

```text
PENDING
```

durumunda kalır.

Başvuru süresi sona erdikten sonra yönetici başvuruları değerlendirerek uygun kişileri `ACCEPTED`, yedek olarak belirlediği kişileri `WAITLISTED`, diğerlerini `REJECTED` durumuna getirir.

---

# 6. Bekleme Listesi

Bekleme listesi, mevcut kapasite nedeniyle doğrudan kabul edilemeyen veya yönetici tarafından yedek aday olarak belirlenen öğrencilerin tutulduğu **yedek listedir**.

Bekleme listesinde öncelik başvuru zamanına göre belirlenir.

Daha önce başvuran kullanıcı daha önceliklidir.

Bekleme listesi:

* `PUBLIC` etkinliklerde sistem tarafından otomatik olarak oluşturulur.
* `PRIVATE` ve kapasitesi bulunan etkinliklerde yönetici tarafından değerlendirme sırasında oluşturulur.

Kapasite açıldığında bekleme listesindeki ilk uygun adaydan başlanır.

---

## 6.1. Herkese Açık Etkinlik

`PUBLIC` etkinliklerde kapasite açıldığında:

```text
WAITLISTED
    ↓
ACCEPTED
```

Bekleme listesindeki ilk kişi otomatik olarak kabul edilir.

Örneğin:

```text
Kapasite: 50

ACCEPTED: 50
WAITLISTED:
#1
#2
#3
```

Bir `ACCEPTED` öğrenci ayrılırsa:

```text
1 kişilik kontenjan açılır
        ↓
WAITLIST #1
        ↓
ACCEPTED
```

olur.

Yönetici bekleme listesinden manuel olarak kişi seçmek zorunda değildir.

---

## 6.2. Özel Etkinlik

`PRIVATE` ve kapasitesi bulunan etkinliklerde bekleme listesi, başvuru süresi sona erdikten sonra yönetici tarafından belirlenir.

Yönetici, başvuruları değerlendirirken bazı öğrencileri yedek aday olarak `WAITLISTED` durumuna alabilir.

Örneğin:

```text
Kapasite: 50
Toplam başvuru: 100

ACCEPTED: 50
WAITLISTED: 20
REJECTED: 30
```

Buradaki 20 öğrenci, kabul edilen öğrencilerden birinin etkinliğe katılamaması veya kabul durumunun sonradan değişmesi halinde kullanılabilecek yedek adaylardır.

Kapasite açıldığında bekleme listesindeki ilk aday:

```text
WAITLISTED
    ↓
ACCEPTED
```

durumuna geçirilir.

Bekleme listesindeki sıra başvuru zamanına göre korunur.

Yönetici bekleme listesindeki adayları yeniden sıralamak zorunda değildir.

---

## 6.3. Kontenjan Açılması

Kontenjan aşağıdaki durumlarda açılabilir:

* Yönetici kapasiteyi artırabilir.
* Daha önce kabul edilmiş bir başvuru yönetici tarafından reddedilebilir.
* Daha önce kabul edilmiş bir başvuru öğrenci tarafından geri çekilebilir (`ACCEPTED → WITHDRAWN`).

Açılan kontenjan, bekleme listesindeki sıraya göre değerlendirilir.

Örneğin:

```text
Kapasite: 50

ACCEPTED: 50
WAITLISTED: 10
```

5 `ACCEPTED` başvurudan biri geri çekilirse:

```text
1 kişilik kontenjan açılır
        ↓
Bekleme listesindeki ilk aday
        ↓
ACCEPTED
```

Sonuç:

```text
ACCEPTED: 50
WAITLISTED: 9
```

olur.

Bu işlem:

* `PUBLIC` etkinliklerde sistem tarafından otomatik gerçekleştirilir.
* `PRIVATE` ve kapasitesi bulunan etkinliklerde, yöneticinin daha önce belirlediği bekleme listesindeki ilk aday kabul edilir.

---

# 7. Başvuru Geri Çekme

Öğrenci başvurusunu etkinlik başlamadan önce geri çekebilir.

Geri çekilen başvuru `WITHDRAWN` durumuna geçer.

Geçerli geçişler:

```text
PENDING    → WITHDRAWN

WAITLISTED → WITHDRAWN

ACCEPTED   → WITHDRAWN
```

`REJECTED` durumundaki başvurunun geri çekilmesine gerek yoktur.

Etkinlik başladıktan sonra başvuru geri çekilemez.

## 7.1. ACCEPTED → WITHDRAWN ve Kapasite

Kabul edilmiş bir başvuru geri çekildiğinde kapasite boşalır.

```text
ACCEPTED → WITHDRAWN
        ↓
Kapasite açıldı
        ↓
WAITLISTED sıradaki aday işlenir
```

Bekleme listesindeki ilk aday, etkinliğin başvuru türüne göre `ACCEPTED` durumuna geçirilir.

`PUBLIC` ve `PRIVATE` etkinliklerde bu işlem için bekleme listesinin sırası başvuru zamanına göre korunur.

## 7.2. Geri Çekilen Öğrencinin Tekrar Başvurması

`WITHDRAWN` durumuna geçen öğrenci, başvuru süreci hâlâ açıksa aynı etkinliğe tekrar başvurabilir.

Bu yeni başvuru, eski başvurunun devamı değildir; yeni bir application kaydı olarak değerlendirilir.

```text
Eski başvuru:

ACCEPTED → WITHDRAWN

Yeni başvuru:

PENDING / ACCEPTED / WAITLISTED
```

Öğrencinin aynı anda aynı etkinlik için birden fazla aktif başvurusu bulunamaz.

Eski `WITHDRAWN` başvuru kaydı silinmez; başvuru geçmişi korunur.

Yeni başvurunun sonucu etkinliğin başvuru türüne ve kapasite durumuna göre belirlenir.

---

# 8. Aynı Etkinliğe Başvurma Kuralları

Öğrencinin aynı etkinlikte aynı anda yalnızca bir aktif başvurusu olabilir.

Öğrenci aşağıdaki durumlardayken yeni bir başvuru oluşturamaz:

* `PENDING`
* `ACCEPTED`
* `WAITLISTED`

`REJECTED` durumundaki bir başvurudan sonra öğrenci yeni başvuru oluşturamaz.

Ancak yönetici daha önce reddedilmiş başvuruyu tekrar kabul edebilir:

```text
REJECTED → ACCEPTED
```

Bu işlem kapasite kurallarına tabidir. Kapasite doluysa `REJECTED → ACCEPTED` işlemi yapılamaz.

Bu durumda öğrencinin yeniden başvuru yapmasına gerek yoktur.

`WITHDRAWN` sonrası tekrar başvuru kuralları için bkz. Bölüm 7.2.

---

# 9. Başvuru Süresi

Yeni başvurular yalnızca belirlenen başvuru tarihleri arasında oluşturulabilir.

Başvuru başlangıç tarihi gelmeden başvuru yapılamaz.

Başvuru bitiş tarihi geçtikten sonra yeni başvuru oluşturulamaz.

Ancak başvuru süresinin bitmiş olması mevcut başvuruların yönetilmesine engel değildir.

Özellikle `PRIVATE` etkinliklerde başvuru süresi sona erdikten sonra mevcut `PENDING` başvurular yönetici tarafından değerlendirilir.

Başvuru süresinin sonunda sonuçlandırma tamamlandığında ilgili başvuruların `PENDING` durumunda kalmaması gerekir.

Mevcut:

* `PENDING`
* `WAITLISTED`

başvurular ilgili kurallara göre işlem görmeye devam edebilir.

---

# 10. Başvuru Durumları

Temel başvuru durumları:

| Durum        | Açıklama                                                    |
| ------------ | ----------------------------------------------------------- |
| `PENDING`    | Başvuru yapılmış ancak henüz nihai sonucu belirlenmemiştir. |
| `ACCEPTED`   | Başvuru kabul edildi.                                       |
| `REJECTED`   | Başvuru reddedildi.                                         |
| `WAITLISTED` | Öğrenci yedek listesinde bekliyor.                          |
| `WITHDRAWN`  | Öğrenci tarafından geri çekildi.                            |

`PENDING`, yalnızca yönetici onayı bekleyen başvurular anlamına gelmez.

Örneğin `PUBLIC` ve kapasitesi bulunan bir etkinlikte, kapasite dolana kadar başvuru `PENDING` üzerinden `ACCEPTED` durumuna otomatik olarak geçirilebilir.

`BAN` bir application status değildir.

`NOT_ATTENDED` bir application status değildir; attendance ayrı bir kavramdır.

---

# 11. Başvuru Durum Geçişleri

Geçerli temel geçişler:

```text
PENDING

 ├──→ ACCEPTED
 ├──→ REJECTED
 └──→ WITHDRAWN (öğrenci tarafından)


WAITLISTED

 ├──→ ACCEPTED
 └──→ WITHDRAWN (öğrenci tarafından)


ACCEPTED

 ├──→ REJECTED     (yönetici tarafından, etkinlik başlamadan önce)
 └──→ WITHDRAWN    (öğrenci tarafından, etkinlik başlamadan önce)


REJECTED

 └──→ ACCEPTED     (yönetici tarafından, kapasite uygunsa)
```

`WAITLISTED → ACCEPTED` geçişinde bekleme listesindeki sıra başvuru zamanına göre belirlenir.

### Geçersiz geçişler

```text
ACCEPTED  → PENDING       ❌

REJECTED  → PENDING       ❌

REJECTED  → yeni başvuru  ❌
```

Öğrenci `REJECTED` durumundan sonra yeni başvuru oluşturamaz.

`WITHDRAWN` sonrası yeni başvuru ayrı bir kural olarak tanımlanmıştır. Bkz. Bölüm 7.2.

---

# 12. Etkinliğe Katılım

Kabul edilmiş öğrenciler etkinliğe katılabilir.

Katılım, başvuru durumundan ayrı bir bilgi olarak değerlendirilir.

Örneğin:

```text
Başvuru:

ACCEPTED

Katılım:

ATTENDED
```

Bu nedenle `ATTENDED`, doğrudan bir başvuru durumu olarak değerlendirilmez.

---

# 13. QR Kod ile Katılım

Fiziksel etkinliklerde QR kod kullanılarak katılım kaydı oluşturulabilir.

Öğrenci QR kodu tarattığında sistem gerekli kontrolleri yaparak katılım kaydı oluşturur.

Başarılı katılım:

```text
ACCEPTED
   ↓
Katılım kaydı oluşturuldu
```

> Online etkinliklerde QR kodun nasıl kullanılacağı henüz kesinleştirilmemiştir. Bkz. Bölüm 17 — Karar Verilmemiş Konular.

---

# 14. Katılmama

Öğrenci `ACCEPTED` durumunda olmasına rağmen etkinliğe katılmadıysa etkinlik sonrasında katılım kaydı bulunmaz.

Bu durumda ayrıca:

```text
NOT_ATTENDED
```

şeklinde bir başvuru durumu oluşturulması gerekmez.

Sistem kabul durumunu ve katılım kaydını ayrı değerlendirir.

---

# 15. Manuel Katılım Düzeltmesi

İlk sürümde yetkili kulüp yöneticileri katılım kayıtlarını manuel olarak düzeltebilir.

Örneğin:

* QR okutulmamış ancak öğrenci etkinliğe katılmışsa katılım eklenebilir.
* Hatalı oluşturulmuş katılım kaydı kaldırılabilir.
* Mevcut katılım bilgisi düzeltilebilir.

Bu yetki:

* Başkan
* Başkan Yardımcısı
* Yönetici

rollerine verilir.

---

# 16. Ön Koşullar — Özet

| Koşul                           | Açıklama                                   |
| ------------------------------- | ------------------------------------------ |
| Sisteme giriş yapılmış olmalı   | Kimliği doğrulanmış kullanıcı gerektirir   |
| Etkinlik yayınlanmış olmalı     | `PUBLISHED` durumunda olmalı               |
| Etkinlik iptal edilmemiş olmalı | `CANCELLED` etkinliğe başvurulamaz         |
| Başvuru süresi açık olmalı      | Başvuru başlangıç ≤ şimdi ≤ başvuru bitiş  |
| Aktif başvuru bulunmamalı       | Aynı anda tek aktif başvuruya izin verilir |

---

# 17. Karar Verilmemiş Konular

Bu use case kapsamında aşağıdaki konular henüz kesinleştirilmemiştir:

### 17.1. Normal Ret Gerekçesi

`PENDING → REJECTED` işleminde yöneticinin ret gerekçesi girmesinin zorunlu olup olmadığı henüz kararlaştırılmamıştır.

Ancak:

```text
ACCEPTED → REJECTED
```

işleminde gerekçe zorunludur.

### 17.2. Online Etkinliklerde QR Kullanımı

Online etkinliklerde QR kod ile katılım doğrulamasının nasıl gerçekleştirileceği henüz kesinleştirilmemiştir.

Fiziksel etkinliklerde QR ile katılım doğrulaması planlanmaktadır. Online etkinliklerdeki katılım doğrulama yöntemi ayrıca değerlendirilecektir.

### 17.3. Ret Nedeni Yapısı

Ret bilgisinin nasıl tutulacağı (serbest metin mi, önceden tanımlı kategoriler mi) henüz kesinleştirilmemiştir.

Veri modeli detayları domain model aşamasında belirlenecektir.

### 17.4. Bildirimler

Başvuru durumlarının değişmesi sonucunda öğrenciye veya yöneticilere gönderilecek bildirimler henüz bu use case kapsamında tanımlanmamıştır.

Bildirim sistemi ayrı bir use case olarak ele alınacaktır.

---

# 18. İlgili Use Case'ler

Bu use case aşağıdaki use case'lerle doğrudan ilişkilidir:

* **Use Case 02 — Kulübün Etkinlik Oluşturması ve Yayınlaması**
* **Use Case 03 — Kulübün Etkinlik Başvurularını Yönetmesi**
