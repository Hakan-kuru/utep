# Use Case 02 — Kulübün Etkinlik Oluşturması ve Yayınlaması

## 1. Amaç

Bu use case, yetkili bir kulüp yöneticisinin yeni bir etkinlik oluşturmasını, etkinliği doğrudan yayınlamasını ve etkinlik başlamadan önce belirli etkinlik bilgilerini düzenleyebilmesini tanımlar.

Bu akış kapsamında:

* Etkinlik oluşturma
* Etkinliğin doğrudan yayınlanması
* Etkinlik türü
* Başvuru tarihleri
* Başvuru tipi
* Kapasite
* Etkinlik düzenleme
* Etkinlik iptali
* Etkinliğin silinmemesi

kuralları ele alınır.

---

# 2. Aktörler

## Ana Aktör

Kulüp yöneticisi.

Etkinlik oluşturma ve yönetme yetkisi bulunan kulüp rolleri:

* Başkan
* Başkan Yardımcısı
* Yönetici

## Yetkisiz Aktörler

Aşağıdaki kullanıcılar etkinlik oluşturamaz:

* Normal öğrenci
* Yalnızca kulübü takip eden öğrenci
* Kulüpte eski yönetici olan kullanıcı

Global admin normal kulüp etkinliklerini oluşturmaz.

---

# 3. Ön Koşullar

Etkinlik oluşturabilmek için:

* Kulüp sistem tarafından onaylanmış olmalıdır.
* Kulüp aktif durumda olmalıdır.
* İşlemi yapan kullanıcı ilgili kulübün yetkili yöneticisi olmalıdır.
* Kullanıcının etkinlik oluşturma yetkisi bulunmalıdır.

---

# 4. Etkinlik Oluşturma ve Yayınlama

Etkinlik oluşturma ve yayınlama iki ayrı işlem değildir.

Yönetici gerekli bilgileri doldurup etkinliği oluşturduğunda etkinlik doğrudan yayınlanır.

```text
Etkinlik bilgileri girilir

        ↓

Etkinlik oluşturulur

        ↓

Etkinlik yayınlanır
```

Etkinlik oluşturulduktan sonra ayrıca global admin onayı beklenmez.

---

# 5. Etkinlik Bilgileri

Etkinlik oluşturulurken aşağıdaki bilgiler belirlenir:

* Etkinlik adı
* Açıklama
* Görsel
* Etkinlik türü
* Başlangıç tarihi ve saati
* Bitiş tarihi ve saati
* Fiziksel konum veya online bağlantı
* Başvuru başlangıç tarihi
* Başvuru bitiş tarihi
* Başvuru tipi
* İsteğe bağlı kapasite

Sistem ayrıca etkinliği oluşturan yöneticiyi kaydeder.

Bu bilgi öğrencilere gösterilmek zorunda değildir ancak sistemde geçmiş ve denetim amacıyla tutulur.

---

# 6. Etkinlik Türü

Etkinlik iki türden biri olabilir:

```text
PHYSICAL

ONLINE
```

## 6.1. Fiziksel Etkinlik

Fiziksel etkinliklerde etkinliğin gerçekleştirileceği konum belirtilir.

Örneğin:

* Fakülte
* Salon
* Derslik
* Kampüs alanı

## 6.2. Online Etkinlik

Online etkinliklerde etkinliğe katılım için bağlantı bilgisi bulunur.

Online etkinlik türü ayrıca platformlara bölünmez.

Örneğin:

* Google Meet
* Zoom
* YouTube
* Diğer online platformlar

aynı `ONLINE` etkinlik türü içerisinde değerlendirilebilir.

---

# 7. Etkinlik Türünün Değiştirilmesi

Etkinlik yayınlandıktan sonra etkinlik türü değiştirilemez.

```text
PHYSICAL → ONLINE    ❌

ONLINE → PHYSICAL    ❌
```

Ancak etkinliğin türüne ait bilgiler değiştirilebilir.

### Fiziksel etkinlik

Konum değiştirilebilir.

### Online etkinlik

Online bağlantı değiştirilebilir.

---

# 8. Başvuru Tipi

Etkinlik oluşturulurken başvuru tipi belirlenir.

İki temel başvuru tipi vardır:

```text
PUBLIC

PRIVATE
```

## 8.1. Herkese Açık

`PUBLIC` etkinliklerde öğrencinin başvurusu yönetici onayı gerektirmez.

Kapasite belirtilmemişse veya kapasite henüz dolmamışsa başvuru otomatik olarak kabul edilir.

```text
Başvuru
   ↓
PENDING
   ↓
ACCEPTED
```

Kapasite dolduktan sonra yeni başvurular bekleme listesine alınır:

```text
Başvuru
   ↓
PENDING
   ↓
WAITLISTED
```

Bekleme listesindeki öğrenciler başvuru zamanına göre sıralanır.

Kapasite açıldığında bekleme listesindeki ilk öğrenci otomatik olarak `ACCEPTED` durumuna geçirilir.

## 8.2. Özel Etkinlik

`PRIVATE` etkinliklerde öğrencinin başvurusu yönetici tarafından değerlendirilir.

Kapasite bulunup bulunmamasından bağımsız olarak, başvuru süresi devam ederken başvuru:

```text
Başvuru
   ↓
PENDING
```

durumuna geçer.

Kapasite bulunmayan PRIVATE etkinliklerde yönetici başvuruları:

```text
PENDING
   ├──→ ACCEPTED
   └──→ REJECTED
```

şeklinde sonuçlandırır.

Kapasitesi bulunan PRIVATE etkinliklerde ise yönetici değerlendirme sonucunda başvuruları:

```text
PENDING
   ├──→ ACCEPTED
   ├──→ WAITLISTED
   └──→ REJECTED
```

şeklinde sonuçlandırabilir.

`WAITLISTED`, yöneticinin uygun gördüğü yedek adayları ifade eder.

---

# 9. Başvuru Tipinin Değiştirilmesi

Etkinlik yayınlandıktan sonra başvuru tipi değiştirilemez.

```text
PUBLIC → PRIVATE    ❌

PRIVATE → PUBLIC    ❌
```

Bu nedenle etkinlik oluşturulurken başvuru tipi dikkatli belirlenmelidir.

---

# 10. Kapasite

Etkinlik oluşturulurken kapasite belirtilmesi isteğe bağlıdır.

Kapasite belirtilmezse etkinliğin kapasite sınırı bulunmaz.

Kapasite belirtilmişse başvurular etkinliğin başvuru tipine göre kapasiteyle birlikte değerlendirilir.

### PUBLIC etkinliklerde

Kapasite dolana kadar uygun başvurular otomatik olarak kabul edilir.

Kapasite dolduktan sonra yeni başvurular `WAITLISTED` durumuna geçer.

### PRIVATE etkinliklerde

Kapasitenin dolmuş olması başvuru süresi devam ederken yeni başvuruları engellemez.

Başvurular `PENDING` durumunda kalır.

Başvuru süresi sona erdikten sonra yönetici kapasiteyi dikkate alarak:

* `ACCEPTED`
* `WAITLISTED`
* `REJECTED`

durumlarını belirler.

Kapasitesi bulunmayan PRIVATE etkinliklerde `WAITLISTED` kullanılmaz.

Bu kurallar Use Case 01 ve Use Case 03 kapsamında ayrıntılandırılmıştır.

---

# 11. Kapasitenin Değiştirilmesi

Kapasite değişikliği başvuru başlangıç tarihine göre farklı kurallara tabidir.

## 11.1. Başvuru Başlangıç Tarihinden Önce

Başvuru süreci henüz başlamamışsa kapasite hem artırılabilir hem azaltılabilir.

```text
50 → 40    ✅

50 → 70    ✅
```

## 11.2. Başvuru Süreci Başladıktan Sonra

Başvuru başlangıç tarihi geçtikten sonra kapasite yalnızca artırılabilir.

```text
50 → 70    ✅

50 → 40    ❌
```

Bu kural doğrudan **başvuru başlangıç tarihine** bağlıdır. Etkinliğe herhangi bir başvuru yapılmış olup olmaması bu kuralı değiştirmez.

Kapasite artırıldığında mevcut bekleme listesi de ilgili kurallara göre işlenir.

Örneğin:

```text
Kapasite: 50
ACCEPTED: 50
WAITLISTED: 10

Kapasite → 55
```

olduğunda açılan 5 kontenjan, bekleme listesindeki ilk 5 adayın kabul edilmesi için kullanılır.

---

# 12. Başvuru Başlangıç Tarihi

Etkinlik oluşturulurken başvuruların başlayacağı tarih ve saat belirlenir.

Başvuru başlangıç tarihi henüz gelmemişse yönetici bu tarihi değiştirebilir.

Örneğin:

```text
Başvuru başlangıcı: 01 Ekim

Bugün: 25 Eylül
```

ise başlangıç tarihi değiştirilebilir.

Başvuru başladıktan sonra başlangıç tarihi değiştirilemez.

```text
Başvuru başlamadı → değiştirilebilir

Başvuru başladı    → değiştirilemez
```

---

# 13. Başvuru Bitiş Tarihi

Başvuru bitiş tarihi etkinlik başlamadan önce değiştirilebilir.

Bu özellikle başvuru süresinin uzatılabilmesini sağlar.

Örneğin:

```text
10 Ekim → 12 Ekim
```

şeklinde başvuru süresi uzatılabilir.

Başvuru süresi sona erdikten sonra mevcut başvurular yönetilmeye devam edebilir.

Özellikle PRIVATE etkinliklerde başvuru süresinin sona ermesinden sonra `PENDING` başvurular yönetici tarafından sonuçlandırılabilir.

---

# 14. Etkinlik Tarihleri

Etkinliğin başlangıç ve bitiş tarihi/saatleri etkinlik başlamadan önce değiştirilebilir.

```text
Başlangıç tarihi/saati → değiştirilebilir

Bitiş tarihi/saati     → değiştirilebilir
```

Etkinlik başladıktan sonra bu alanların değiştirilmesi mümkün değildir.

---

# 15. Etkinlik Adı

Etkinlik adı yayınlandıktan sonra değiştirilemez.

```text
Etkinlik adı → ❌
```

Bunun amacı etkinlik yayınlandıktan sonra öğrencilerin gördüğü etkinliğin kimliğinin değiştirilmemesidir.

---

# 16. Açıklama

Etkinlik başlamamışsa açıklama düzenlenebilir.

```text
Açıklama → ✅
```

Yönetici etkinlik hakkında ek bilgi verebilir veya mevcut açıklamayı güncelleyebilir.

---

# 17. Görsel

Etkinlik başlamamışsa etkinliğin görseli değiştirilebilir.

```text
Görsel → ✅
```

---

# 18. Konum / Online Bağlantı

Etkinlik başlamamışsa etkinliğin erişim bilgileri değiştirilebilir.

### Fiziksel etkinlik

```text
Konum → değiştirilebilir
```

### Online etkinlik

```text
Online bağlantı → değiştirilebilir
```

Ancak etkinliğin türü değiştirilemez.

---

# 19. Etkinlik Düzenleme Özeti

Etkinlik henüz başlamamışsa:

| Alan                     | Düzenlenebilir                                                                                            |
| ------------------------ | --------------------------------------------------------------------------------------------------------- |
| Etkinlik adı             | ❌                                                                                                         |
| Açıklama                 | ✅                                                                                                         |
| Görsel                   | ✅                                                                                                         |
| Etkinlik türü            | ❌                                                                                                         |
| Başlangıç tarihi/saati   | ✅                                                                                                         |
| Bitiş tarihi/saati       | ✅                                                                                                         |
| Fiziksel konum           | ✅                                                                                                         |
| Online bağlantı          | ✅                                                                                                         |
| Başvuru tipi             | ❌                                                                                                         |
| Kapasite                 | ✅ Başvuru başlamadıysa artırılabilir veya azaltılabilir; başvuru başladıktan sonra yalnızca artırılabilir |
| Başvuru başlangıç tarihi | ✅ Başvuru başlamadıysa                                                                                    |
| Başvuru bitiş tarihi     | ✅                                                                                                         |

Etkinlik başladıktan sonra etkinliğin düzenlenmesine ilişkin bu değişiklikler yapılamaz.

---

# 20. Etkinliğin İptal Edilmesi

Yayınlanmış bir etkinlik, başlamadan önce iptal edilebilir.

İptal edilen etkinlik:

```text
CANCELLED
```

durumuna geçer.

---

## 20.1. İptal Edilen Etkinlikte Yeni Başvurular

İptal edilen etkinliğe yeni başvuru alınmaz.

---

## 20.2. İptal Edilen Etkinlikte QR

İptal edilen etkinliğin QR kodu ile katılım alınamaz.

---

## 20.3. Mevcut Kayıtlar

Etkinlik iptal edildiğinde mevcut:

* Başvurular
* Kabul kayıtları
* Ret kayıtları
* Bekleme listeleri
* Katılım kayıtları

silinmez.

Etkinlik geçmişte görüntülenebilir ve iptal edildiği açıkça belirtilir.

---

# 21. Etkinliğin Silinmesi

MVP'de etkinlik fiziksel olarak silinmez.

Etkinlik yanlış oluşturulmuş olsa veya artık gerçekleştirilmeyecek olsa bile kayıt tamamen kaldırılmaz.

Bunun yerine uygun durum bilgisi kullanılır.

Özellikle iptal edilen etkinlik:

```text
CANCELLED
```

olarak saklanır.

Böylece geçmiş:

* Etkinlikler
* Başvurular
* Katılım kayıtları
* Kulüp faaliyetleri

korunabilir.

---

# 22. Etkinliği Oluşturan Yönetici

Sistem etkinliği oluşturan yöneticiyi kaydeder.

Örneğin:

```text
createdBy = User
```

Bu bilgi:

* Denetim
* Geçmiş
* Yönetim kayıtları
* İleride oluşturulabilecek raporlar

için kullanılabilir.

Öğrencinin etkinlik ekranında gösterilmesi zorunlu değildir.

---

# 23. Yetki Özeti

| İşlem               |                               Başkan |                    Başkan Yardımcısı |                             Yönetici | Normal Öğrenci |
| ------------------- | -----------------------------------: | -----------------------------------: | -----------------------------------: | -------------: |
| Etkinlik oluşturma  |                                    ✅ |                                    ✅ |                                    ✅ |              ❌ |
| Etkinliği yayınlama |                                    ✅ |                                    ✅ |                                    ✅ |              ❌ |
| Etkinlik düzenleme  |                                    ✅ |                                    ✅ |                                    ✅ |              ❌ |
| Kapasite artırma    |                                    ✅ |                                    ✅ |                                    ✅ |              ❌ |
| Kapasite azaltma    | ✅ (yalnızca başvuru başlamadan önce) | ✅ (yalnızca başvuru başlamadan önce) | ✅ (yalnızca başvuru başlamadan önce) |              ❌ |
| Etkinlik iptali     |                                    ✅ |                                    ✅ |                                    ✅ |              ❌ |

---

# 24. Etkinlik Yaşam Döngüsü

Etkinlik oluşturulduğunda doğrudan yayınlanır.

Temel yaşam döngüsü:

```text
CREATED

   ↓

PUBLISHED

   │

   ├────────────→ CANCELLED

   │

   ↓

COMPLETED
```

`DRAFT` durumu bulunmaz.

Etkinlik oluşturma ve yayınlama tek işlem olduğundan yöneticinin ayrıca "Taslak olarak kaydet" işlemi yapmasına gerek yoktur.

---

# 25. Karar Verilmemiş Konular

Bu use case kapsamında şu anda bilinçli olarak açık bırakılan konular:

### 25.1. Başvuru başlangıç tarihinin sınırları

Başvuru başlangıç tarihinin etkinlik başlangıç tarihine ne kadar yakın olabileceği henüz ayrıca belirlenmemiştir.

### 25.2. İptal sonrası bildirimler

Etkinlik iptal edildiğinde öğrencilere gönderilecek bildirimler henüz belirlenmemiştir.

Bildirim davranışları ayrı bir use case kapsamında ele alınacaktır.

---

# 26. İlgili Use Case'ler

Bu use case aşağıdaki use case'lerle doğrudan ilişkilidir:

* **Use Case 01 — Öğrencinin Etkinliğe Başvurması ve Katılması**
* **Use Case 03 — Kulübün Etkinlik Başvurularını Yönetmesi**
