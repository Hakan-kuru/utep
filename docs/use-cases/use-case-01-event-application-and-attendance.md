# Use Case 01 — Öğrencinin Etkinliğe Başvurması ve Katılması

## 1. Amaç

Bu use case, öğrencinin bir etkinliği görüntülemesi, etkinliğe başvurması, başvuru sonucunu takip etmesi ve etkinliğe katılımının sisteme kaydedilmesini tanımlar.

Bu akış kapsamında:

* Etkinliğe başvurma
* Başvuru durumunun belirlenmesi
* Kontenjan ve bekleme listesi
* Ban kontrolü
* Etkinliğe katılım
* QR kod ile katılım
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
* Öğrencinin aynı etkinlikte daha önce başvurusu bulunmamalıdır.

Öğrencinin kulüp tarafından banlanmış olması başvuruyu görüntülemesini veya başlatmasını engellemez. Ban kontrolü başvurunun değerlendirilmesi sırasında yapılır.

---

# 4. Etkinliğe Başvurma

Öğrenci etkinlik detaylarını görüntüler ve başvuru koşullarını karşılıyorsa etkinliğe başvurabilir.

Başvuru sonucunda oluşacak durum:

* Etkinlik herkese açıksa ve kontenjan uygunsa → `ACCEPTED`
* Etkinlik onay gerektiriyorsa ve kontenjan uygunsa → `PENDING`
* Kontenjan doluysa → `WAITLISTED`
* Öğrencinin etkinlik tarihinde geçerli banı varsa → `REJECTED` + `BAN`

---

## 4.1. Herkese Açık Etkinlik

Etkinlik `PUBLIC` ise yönetici onayı gerekmez.

Kontenjan uygunsa:

```text
Başvuru
   ↓
ACCEPTED
```

Öğrenci doğrudan etkinliğe kabul edilmiş olur.

---

## 4.2. Onay Gerektiren Etkinlik

Etkinlik manuel onay gerektiriyorsa:

```text
Başvuru
   ↓
PENDING
```

Başvuru daha sonra kulüp yöneticisi tarafından kabul veya reddedilir.

Bu işlem **Use Case 03 — Kulübün Etkinlik Başvurularını Yönetmesi** kapsamında ele alınır.

---

# 5. Kontenjan

Etkinlik oluşturulurken kapasite belirtilmesi isteğe bağlıdır.

## 5.1. Kapasite Belirtilmemişse

Etkinlikte kapasite sınırı bulunmaz.

Başvurular kapasite nedeniyle bekleme listesine alınmaz.

---

## 5.2. Kapasite Dolmuşsa

Etkinliğin kapasitesi dolmuşsa yeni başvuru:

```text
WAITLISTED
```

durumuna alınır.

Bekleme listesindeki kullanıcıların sırası başvuru zamanına göre belirlenir.

Daha önce başvuran kullanıcı daha önceliklidir.

---

# 6. Bekleme Listesi

Bekleme listesi otomatik olarak yönetilir.

Kontenjan açıldığında sistem, `WAITLISTED` kullanıcıları başvuru sırasına göre işler.

## 6.1. Herkese Açık Etkinlik

```text
WAITLISTED
     ↓
ACCEPTED
```

Kullanıcı doğrudan kabul edilir.

## 6.2. Onay Gerektiren Etkinlik

```text
WAITLISTED
     ↓
PENDING
```

Kullanıcı tekrar yönetici onayına alınır.

---

## 6.3. Kontenjan Açılması

Kontenjan aşağıdaki durumlarda açılabilir:

* Yönetici kapasiteyi artırabilir.
* Daha önce kabul edilmiş bir başvuru yönetici tarafından reddedilebilir.

Açılan kontenjan, bekleme listesindeki ilk kişilerden başlanarak doldurulur.

Örneğin:

```text
Kapasite: 50
ACCEPTED: 50
WAITLISTED: 10
```

5 kabul edilmiş başvuru reddedilirse:

```text
5 kişilik kontenjan açılır
        ↓
İlk 5 WAITLISTED kullanıcı
        ↓
PUBLIC       → ACCEPTED
ONAY GEREKLİ → PENDING
```

Yönetici ayrıca bekleme listesinden manuel olarak kişi seçmek zorunda değildir.

---

# 7. Aynı Etkinliğe Tekrar Başvurma

Bir öğrenci aynı etkinliğe yalnızca **bir kez** başvurabilir.

Öğrenci:

* `PENDING`
* `ACCEPTED`
* `WAITLISTED`
* `REJECTED`

durumlarından hangisinde olursa olsun yeni bir başvuru oluşturamaz.

Özellikle:

```text
REJECTED → yeni başvuru
```

mümkün değildir.

Ancak yönetici daha önce reddedilmiş başvuruyu tekrar kabul edebilir:

```text
REJECTED → ACCEPTED
```

Bu durumda öğrencinin yeniden başvuru yapmasına gerek yoktur.

---

# 8. Başvuru Süresi

Yeni başvurular yalnızca belirlenen başvuru tarihleri arasında oluşturulabilir.

Başvuru başlangıç tarihi gelmeden başvuru yapılamaz.

Başvuru bitiş tarihi geçtikten sonra yeni başvuru oluşturulamaz.

Ancak başvuru süresinin bitmiş olması mevcut başvuruların yönetilmesine engel değildir.

Mevcut:

* `PENDING`
* `WAITLISTED`

başvurular ilgili kurallara göre işlem görmeye devam edebilir.

---

# 9. Ban Kontrolü

Kulüp, kullanıcıya geçici veya kalıcı ban uygulayabilir.

Ban kulüp bazındadır.

Banlı kullanıcı:

* Kulüp sayfasını görüntüleyebilir.
* Kulübün etkinliklerini görüntüleyebilir.
* Etkinliğe başvurabilir.

Ancak sistem başvuruyu değerlendirirken banın etkinlik tarihinde geçerli olup olmadığını kontrol eder.

---

## 9.1. Ban Tarihi

Ban değerlendirmesinde başvuru tarihi değil, **etkinliğin gerçekleşeceği tarih** esas alınır.

Örneğin:

```text
Ban:
01 Ekim – 30 Ekim

Etkinlik:
01 Kasım
```

ise kullanıcı etkinlik tarihinde banlı değildir.

Ancak:

```text
Ban:
01 Ekim – 30 Ekim

Etkinlik:
25 Ekim
```

ise kullanıcı etkinlik tarihinde banlıdır.

---

## 9.2. Ban Nedeniyle Reddetme

Kullanıcı etkinlik tarihinde banlıysa başvurusu otomatik olarak reddedilir:

```text
REJECTED
rejectionReason = BAN
```

Ban nedeniyle oluşan ret, normal yönetici reddinden ayrı bir neden olarak tutulur.

---

# 10. Kabul Edilmiş Başvurunun Banlanması

Bir kullanıcı etkinliğe daha önce kabul edilmiş olsa bile daha sonra banlanabilir.

Eğer ban etkinlik tarihinde geçerliyse:

```text
ACCEPTED
    ↓
REJECTED
reason = BAN
```

Başvuru otomatik olarak reddedilir.

---

# 11. Başvuru Durumları

Temel başvuru durumları:

| Durum        | Açıklama                               |
| ------------ | -------------------------------------- |
| `PENDING`    | Yönetici onayı bekleniyor              |
| `ACCEPTED`   | Başvuru kabul edildi                   |
| `REJECTED`   | Başvuru reddedildi                     |
| `WAITLISTED` | Kontenjan nedeniyle bekleme listesinde |

Ban ayrı bir durum olarak tutulmaz.

Ban nedeniyle oluşan ret:

```text
REJECTED + BAN
```

şeklinde ifade edilir.

---

# 12. Başvuru Durum Geçişleri

Geçerli temel geçişler:

```text
PENDING
 ├──→ ACCEPTED
 └──→ REJECTED

WAITLISTED
 ├──→ ACCEPTED     (PUBLIC)
 └──→ PENDING      (ONAY GEREKLİ)

REJECTED
 └──→ ACCEPTED
```

Ban nedeniyle:

```text
ACCEPTED
    ↓
REJECTED + BAN
```

mümkündür.

### Geçersiz geçişler

```text
ACCEPTED  → PENDING       ❌
REJECTED  → PENDING       ❌
```

Öğrencinin reddedildikten sonra yeni başvuru oluşturması da mümkün değildir.

---

# 13. Etkinliğe Katılım

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

# 14. QR Kod ile Katılım

Etkinliklerde QR kod kullanılarak katılım kaydı oluşturulabilir.

QR kod kullanımı yalnızca fiziksel etkinliklerle sınırlı değildir.

Online etkinliklerde QR kod etkinlik sırasında paylaşılabilir.

Öğrenci QR kodu tarattığında sistem gerekli kontrolleri yaparak katılım kaydı oluşturur.

Başarılı katılım:

```text
ACCEPTED
   ↓
Katılım kaydı oluşturuldu
```

---

# 15. Katılmama

Öğrenci `ACCEPTED` durumunda olmasına rağmen etkinliğe katılmadıysa etkinlik sonrasında katılım kaydı bulunmaz.

Bu durumda ayrıca:

```text
NOT_ATTENDED
```

şeklinde bir başvuru durumu oluşturulması gerekmez.

Sistem kabul durumunu ve katılım kaydını ayrı değerlendirir.

---

# 16. Manuel Katılım Düzeltmesi

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

# 17. Karar Verilmemiş Konular

Bu use case kapsamında aşağıdaki konular henüz kesinleştirilmemiştir:

### 17.1. Normal Ret Gerekçesi

`PENDING → REJECTED` işleminde yöneticinin ret gerekçesi girmesinin zorunlu olup olmadığı henüz kararlaştırılmamıştır.

Ancak:

```text
ACCEPTED → REJECTED
```

işleminde gerekçe zorunludur.

### 17.2. Bildirimler

Başvuru durumlarının değişmesi sonucunda öğrenciye veya yöneticilere gönderilecek bildirimler henüz bu use case kapsamında tanımlanmamıştır.

Bildirim sistemi ayrı bir use case olarak ele alınacaktır.

---

# 18. İlgili Use Case'ler

Bu use case aşağıdaki use case'lerle doğrudan ilişkilidir:

* **Use Case 02 — Kulübün Etkinlik Oluşturması ve Yayınlaması**
* **Use Case 03 — Kulübün Etkinlik Başvurularını Yönetmesi**
