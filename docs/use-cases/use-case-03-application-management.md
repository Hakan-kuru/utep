# Use Case 03 — Kulübün Etkinlik Başvurularını Yönetmesi

## 1. Amaç

Bu use case, kulüp yöneticilerinin kendi kulüplerine ait etkinliklere yapılan öğrenci başvurularını görüntülemesini ve başvuruların durumlarını yönetmesini tanımlar.

Başvuruların kabul edilmesi, reddedilmesi, bekleme listelerinin otomatik olarak işlenmesi ve gerekli durumlarda daha önce verilmiş kararların düzeltilmesi bu use case kapsamındadır.

Bildirimlerin gönderilmesi bu use case'in kapsamında değildir. Bildirim davranışları ayrı bir use case'te ele alınacaktır.

---

## 2. Aktörler

### Birincil aktörler

* Kulüp Başkanı (`PRESIDENT`)
* Başkan Yardımcısı (`VICE_PRESIDENT`)
* Kulüp Yöneticisi (`MANAGER`)

Bu üç rol de kulübün etkinlik başvurularını yönetebilir.

### İkincil aktörler

* Öğrenci
* Sistem

Öğrenci başvurunun sonucundan etkilenir ancak başvuru yönetimini gerçekleştirmez.

Sistem ise kapasite, bekleme listesi ve ban gibi kuralları otomatik olarak uygular.

---

## 3. Ön Koşullar

* Kullanıcının sisteme giriş yapmış olması gerekir.
* Kullanıcının ilgili kulüpte aktif bir yönetici rolüne sahip olması gerekir.
* Yönetici tarafından işlem yapılacak etkinliğin ilgili kulübe ait olması gerekir.
* Etkinliğin iptal edilmemiş olması gerekir.

---

# 4. Başvuru Durumları

Bir etkinlik başvurusu aşağıdaki durumlardan birinde bulunabilir:

```text
PENDING
ACCEPTED
REJECTED
WAITLISTED
```

### PENDING

Başvuru yapılmış ancak kulüp tarafından henüz kabul veya ret kararı verilmemiştir.

`APPROVAL_REQUIRED` etkinliklerde kullanılır.

### ACCEPTED

Öğrencinin etkinliğe katılımı kabul edilmiştir.

`PUBLIC` etkinliklerde kapasite uygunsa başvuru doğrudan bu duruma geçebilir.

### REJECTED

Başvuru reddedilmiştir.

Reddedilen öğrenci aynı etkinliğe tekrar başvuramaz.

### WAITLISTED

Etkinliğin kapasitesi dolu olduğu için öğrenci bekleme listesine alınmıştır.

Bekleme listesindeki öğrenciler manuel olarak seçilmez. Kapasite açıldığında sistem başvuru zamanına göre otomatik olarak işlem yapar.

---

# 5. Başvuruların Görüntülenmesi

Kulüp yöneticisi kendi etkinliklerinden birini seçerek o etkinliğe ait başvuruları görüntüleyebilir.

Başvurular durumlarına göre filtrelenebilir:

* Bekleyenler
* Kabul edilenler
* Reddedilenler
* Bekleme listesindekiler

Başvuru listesinde en azından aşağıdaki bilgiler bulunmalıdır:

* Öğrenci
* Başvuru tarihi
* Başvuru durumu
* Başvuru durumunun değişme tarihi
* Varsa ret nedeni

---

# 6. PENDING Başvurunun Yönetilmesi

Bir başvuru `PENDING` durumundaysa kulüp yöneticisi iki işlem yapabilir:

```text
PENDING → ACCEPTED
PENDING → REJECTED
```

### 6.1. Kabul

Yönetici başvuruyu kabul ettiğinde:

```text
PENDING → ACCEPTED
```

durumu oluşur.

Sistem kapasite kurallarını dikkate almalıdır.

### 6.2. Ret

Yönetici başvuruyu reddettiğinde:

```text
PENDING → REJECTED
```

durumu oluşur.

Ret nedeni konusu henüz kesinleştirilmemiştir.

> **Açık karar:** Normal bir ret işleminde yöneticinin ret nedeni girmesinin zorunlu olup olmayacağı daha sonra belirlenecektir.

---

# 7. ACCEPTED Başvurunun Yönetilmesi

Kabul edilmiş bir başvuru, etkinlik başlamadan önce yönetici tarafından reddedilebilir.

```text
ACCEPTED → REJECTED
```

Bu işlem yalnızca etkinlik başlamadan önce yapılabilir.

Bu durumda ret nedeni zorunludur.

Ret nedeni öğrenci tarafından görüntülenebilir.

### Örnek

```text
Öğrenci: Ahmet
Durum: ACCEPTED

Yönetici:
"Etkinlik kontenjanı değişti."

Sonuç:
ACCEPTED → REJECTED
Ret nedeni: "Etkinlik kontenjanı değişti."
```

---

## 7.1. ACCEPTED → PENDING Yasaktır

Kabul edilmiş bir başvuru tekrar `PENDING` durumuna getirilemez.

```text
ACCEPTED → PENDING ❌
```

---

# 8. REJECTED Başvurunun Yönetilmesi

Reddedilmiş bir başvuru için öğrenci yeniden başvuru yapamaz.

Ancak kulüp yöneticisi önceki ret kararını düzeltebilir.

```text
REJECTED → ACCEPTED
```

Bu işlem yeni bir başvuru oluşturmaz.

Mevcut başvurunun durumu doğrudan `ACCEPTED` yapılır.

### REJECTED → PENDING Yasaktır

```text
REJECTED → PENDING ❌
```

Yönetici öğrenciyi tekrar değerlendirmek istiyorsa `PENDING` durumuna geri döndürmek yerine doğrudan kabul edebilir.

---

# 9. WAITLISTED Başvurular

Bekleme listesi yalnızca etkinliğin kapasitesi dolduğunda kullanılır.

Bekleme listesindeki öğrenciler:

* Başvuru zamanına göre sıralanır.
* Yönetici tarafından tek tek seçilmez.
* Kapasite açıldığında sistem tarafından otomatik olarak işlenir.

Öncelik:

```text
Daha erken başvuru
        ↓
Daha yüksek öncelik
```

---

# 10. Kapasite Açıldığında Otomatik İşlem

Kapasitede yer açıldığında sistem bekleme listesini başvuru zamanına göre işler.

### PUBLIC etkinlik

```text
WAITLISTED → ACCEPTED
```

### APPROVAL_REQUIRED etkinlik

```text
WAITLISTED → PENDING
```

Bu işlemde bekleme listesindeki ilk uygun öğrenci önceliklidir.

### Örnek

Etkinlik kapasitesi:

```text
50 kişi
```

Mevcut durum:

```text
50 ACCEPTED
10 WAITLISTED
```

Kabul edilmiş 5 öğrencinin başvurusu etkinlik başlamadan önce reddedilirse:

```text
5 kişilik kapasite açılır
        ↓
Bekleme listesindeki ilk 5 kişi alınır
```

Sonuç:

`PUBLIC` etkinlikte:

```text
5 WAITLISTED → ACCEPTED
```

`APPROVAL_REQUIRED` etkinlikte:

```text
5 WAITLISTED → PENDING
```

---

# 11. Kapasitenin Artırılması

Etkinliğin kapasitesi artırıldığında da aynı otomatik bekleme listesi kuralı uygulanır.

Örneğin:

```text
Kapasite: 50 → 70

20 kişilik yeni kapasite oluşur.
```

Bekleme listesindeki ilk 20 uygun başvuru otomatik olarak işlenir.

`PUBLIC`:

```text
WAITLISTED → ACCEPTED
```

`APPROVAL_REQUIRED`:

```text
WAITLISTED → PENDING
```

---

# 12. Başvuru Tarihi Sıralaması

Bekleme listesindeki öncelik başvurunun oluşturulma zamanına göre belirlenir.

Örneğin:

```text
09:01  → Ahmet
09:05  → Mehmet
09:12  → Ayşe
09:20  → Zeynep
```

Kapasite 2 kişi açarsa:

```text
Ahmet
Mehmet
```

öncelikli olarak işlenir.

Yönetici bu sırayı manuel olarak değiştiremez.

---

# 13. Başvuru Sonrası Ban Durumu

Bir öğrenci kulüp tarafından etkinlik tarihi için geçerli olacak şekilde banlanmışsa sistem başvuru durumunu otomatik olarak değerlendirir.

Banın geçerlilik tarihi esas alınır.

### Öğrenci başvurduğunda ban etkinlik tarihinde geçerliyse

Başvuru:

```text
→ REJECTED
```

olur.

Ret nedeni:

```text
BAN
```

olarak kaydedilir.

### Öğrenci daha önce ACCEPTED durumundaysa

Ban etkinlik tarihinde geçerli hale gelirse:

```text
ACCEPTED → REJECTED
```

otomatik olarak gerçekleşir.

Ret nedeni:

```text
BAN
```

olur.

Bu işlem yöneticinin manuel karar vermesini gerektirmez.

---

# 14. Başvuru Sonrası Banın Etkinlik Tarihine Göre Değerlendirilmesi

Ban kontrolünde başvurunun yapıldığı tarih değil, **etkinliğin gerçekleşeceği tarih** esas alınır.

Örneğin:

```text
Başvuru tarihi: 1 Mayıs
Etkinlik tarihi: 20 Mayıs

Ban:
10 Mayıs → 15 Mayıs
```

Ban etkinlik tarihinde aktif olmadığı için başvuru bu ban nedeniyle reddedilmez.

Ancak:

```text
Ban:
10 Mayıs → 25 Mayıs
```

ise ban etkinlik tarihinde aktif olduğundan sistem ilgili başvuruyu reddeder.

---

# 15. Başvuru Son Başvuru Tarihinden Sonra

Başvuru süresi sona erdiğinde yeni öğrenci başvuruları alınmaz.

Ancak bu durum mevcut başvuruların yönetilmesini engellemez.

Yönetici:

* `PENDING` başvuruları kabul edebilir.
* `PENDING` başvuruları reddedebilir.
* Kapasiteyi artırabilir.
* Açılan kapasite sonucunda bekleme listesinin otomatik olarak işlenmesini sağlayabilir.

Yani:

```text
Başvuru süresi bitti
        ↓
Yeni başvuru ❌
        ↓
Mevcut başvuruları yönetme ✅
```

---

# 16. Etkinlik Başladıktan Sonra

Etkinlik başladıktan sonra başvuru kararlarında aşağıdaki işlemler yapılamaz:

```text
ACCEPTED → REJECTED ❌
```

Başvuru yönetimi etkinliğin başlamasından sonra sonlandırılır.

Etkinlik başladıktan sonra öğrencinin etkinliğe gerçekten katılıp katılmadığı **Attendance** sistemi üzerinden takip edilir.

---

# 17. Katılım ile Başvurunun Ayrılması

Başvurunun `ACCEPTED` olması öğrencinin etkinliğe katıldığı anlamına gelmez.

Örneğin:

```text
Application:
ACCEPTED

Attendance:
katılmadı
```

veya:

```text
Application:
ACCEPTED

Attendance:
katıldı
```

olabilir.

Bu nedenle:

```text
Application Status
```

ile

```text
Attendance
```

ayrı tutulur.

Etkinlik sonrasında kullanıcı arayüzünde kabul edilmiş bir başvuru:

* Katıldı
* Katılmadı

şeklinde gösterilebilir.

Ancak `NOT_ATTENDED` ayrı bir application status değildir.

---

# 18. Katılım Kayıtlarının Yönetilmesi

Kulüp yöneticileri gerektiğinde katılım kayıtlarını manuel olarak düzeltebilir.

Yetkili roller:

* Başkan
* Başkan Yardımcısı
* Yönetici

Yönetici:

* Katılım kaydı ekleyebilir.
* Hatalı katılım kaydını kaldırabilir.
* Katılım durumunu düzeltebilir.

Bu işlem QR ile otomatik oluşturulan katılım kaydının hatalı olması veya öğrencinin QR okutamaması gibi durumlarda kullanılabilir.

---

# 19. Başvuru Geçişleri

## Geçerli geçişler

```text
PENDING → ACCEPTED
PENDING → REJECTED

REJECTED → ACCEPTED

WAITLISTED → ACCEPTED
WAITLISTED → PENDING

ACCEPTED → REJECTED
```

Ayrıca ban nedeniyle sistem tarafından:

```text
ACCEPTED → REJECTED
```

geçişi otomatik olarak yapılabilir.

---

## Geçersiz geçişler

```text
ACCEPTED → PENDING ❌
REJECTED → PENDING ❌
```

Öğrenci tarafından:

```text
REJECTED → yeni başvuru ❌
```

yapılamaz.

---

# 20. Öğrencinin Aynı Etkinliğe Tekrar Başvurması

Bir öğrenci aynı etkinlik için yalnızca bir başvuru kaydına sahip olabilir.

Başvuru `REJECTED` olsa bile aynı etkinliğe yeni başvuru oluşturamaz.

Yönetici kararını değiştirmek isterse:

```text
REJECTED → ACCEPTED
```

geçişini kullanır.

Yeni bir başvuru oluşturulmaz.

---

# 21. Başvuru Geri Çekme

MVP kapsamında öğrencinin yaptığı başvuruyu geri çekmesi desteklenmez.

Dolayısıyla:

```text
PENDING → CANCELLED ❌
WAITLISTED → CANCELLED ❌
ACCEPTED → CANCELLED ❌
```

gibi öğrenci tarafından yapılan bir başvuru iptali bulunmaz.

Bu özellik ileride ayrıca değerlendirilebilir.

---

# 22. Yetki Özeti

| İşlem                            | Başkan | Başkan Yardımcısı | Yönetici |
| -------------------------------- | -----: | ----------------: | -------: |
| Başvuruları görüntüleme          |      ✅ |                 ✅ |        ✅ |
| PENDING → ACCEPTED               |      ✅ |                 ✅ |        ✅ |
| PENDING → REJECTED               |      ✅ |                 ✅ |        ✅ |
| ACCEPTED → REJECTED              |      ✅ |                 ✅ |        ✅ |
| REJECTED → ACCEPTED              |      ✅ |                 ✅ |        ✅ |
| Bekleme listesini manuel yönetme |      ❌ |                 ❌ |        ❌ |
| Katılım kaydı ekleme             |      ✅ |                 ✅ |        ✅ |
| Katılım kaydı silme/düzeltme     |      ✅ |                 ✅ |        ✅ |

Bekleme listesi sistem tarafından otomatik yönetildiği için yöneticilerin manuel olarak öğrenci seçmesine gerek yoktur.

---

# 23. Sistem Tarafından Otomatik Gerçekleştirilen İşlemler

Sistem aşağıdaki işlemleri otomatik olarak gerçekleştirir:

### Kapasite açılması

```text
Kapasite açıldı
      ↓
WAITLISTED öğrenciler sıralanır
      ↓
PUBLIC → ACCEPTED
APPROVAL_REQUIRED → PENDING
```

### Ban kontrolü

```text
Ban etkinlik tarihinde geçerli
      ↓
Başvuru etkilenir
      ↓
REJECTED + BAN
```

### Etkinlik başlangıcı

```text
Etkinlik başladı
      ↓
Başvuru kararları artık değiştirilemez
      ↓
Attendance süreci devam eder
```

---

# 24. Use Case Akışı

Temel akış:

```text
Kulüp yöneticisi
      ↓
Etkinliği seçer
      ↓
Başvuruları görüntüler
      ↓
Başvuru durumunu inceler
      ↓
┌─────────────────────────────┐
│                             │
PENDING                   ACCEPTED
│                             │
├─ Kabul → ACCEPTED            └─ Ret → REJECTED
│
└─ Ret → REJECTED

REJECTED
   │
   └─ Yönetici düzeltmesi
          ↓
      ACCEPTED

WAITLISTED
   │
   └─ Kapasite açılması
          ↓
   PUBLIC → ACCEPTED
   APPROVAL_REQUIRED → PENDING
```

---

# 25. Karar Verilmemiş Konular

Aşağıdaki konular bu use case kapsamında henüz kesinleştirilmemiştir:

* Normal `PENDING → REJECTED` işleminde ret nedeninin zorunlu olup olmayacağı.
* Ret nedenlerinin serbest metin mi yoksa önceden belirlenmiş seçenekler mi olacağı.
* Başvuru listesinde hangi öğrenci bilgilerinin yöneticilere gösterileceği.
* Başvuru geçmişinin ne kadar ayrıntılı tutulacağı.
* Attendance düzeltmelerinin audit/history kayıtlarının kullanıcıya gösterilip gösterilmeyeceği.

Bu kararlar sonraki aşamada netleştirilecektir.

---

# 26. Kapsam Dışı Konular

Bu use case aşağıdaki konuları kapsamaz:

* Öğrencinin etkinlik keşfetmesi.
* Öğrencinin etkinliğe ilk kez başvurması.
* Etkinlik oluşturma ve yayınlama.
* Kulüp oluşturma ve kulüp doğrulama.
* Bildirimlerin gönderilmesi.
* Etkinlik puanlama/değerlendirme.
* Kulüp takip etme.
* Global admin işlemleri.

Bu işlemler ilgili use case'lerde ele alınacaktır.

---

# 27. İlgili Use Case'ler

* **UC01 — Öğrencinin Etkinliğe Başvurması ve Katılması**
* **UC02 — Kulübün Etkinlik Oluşturması ve Yayınlaması**
* **UC04 — Bildirimlerin Yönetilmesi** *(planlandı)*
