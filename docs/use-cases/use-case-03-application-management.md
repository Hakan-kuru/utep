# Use Case 03 — Kulübün Etkinlik Başvurularını Yönetmesi

## 1. Amaç

Bu use case, kulüp yöneticilerinin kendi kulüplerine ait etkinliklere yapılan öğrenci başvurularını görüntülemesini ve başvuruların durumlarını yönetmesini tanımlar.

Başvuruların kabul edilmesi, reddedilmesi, gerekli durumlarda öğrencilerin yedek listeye alınması, yedek listenin otomatik olarak işlenmesi ve gerekli durumlarda daha önce verilmiş kararların düzeltilmesi bu use case kapsamındadır.

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

Sistem ise başvuru tipi, kapasite ve yedek liste gibi kuralları otomatik olarak uygular.

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

WITHDRAWN
```

### PENDING

Başvuru yapılmış ancak henüz kesin bir kabul veya ret kararı verilmemiştir.

`APPROVAL_REQUIRED` etkinliklerde başvurular yönetici tarafından değerlendirilmek üzere `PENDING` durumunda tutulur.

`PUBLIC` etkinliklerde ise başvuru sistemi tarafından otomatik olarak değerlendirilir. Başvuru koşulları uygunsa `ACCEPTED` durumuna geçer; kapasite sınırı bulunan etkinliklerde kapasite dolmuşsa `WAITLISTED` durumuna geçebilir.

### ACCEPTED

Öğrencinin etkinliğe katılımı kabul edilmiştir.

`PUBLIC` etkinliklerde uygun başvurular sistem tarafından otomatik olarak bu duruma geçirilebilir.

`APPROVAL_REQUIRED` etkinliklerde ise yönetici tarafından verilen kabul kararı sonucunda bu duruma geçilir.

### REJECTED

Başvuru reddedilmiştir.

Yönetici gerekli durumlarda `REJECTED` durumundaki bir başvuruyu tekrar kabul edebilir. Bkz. Bölüm 8.

### WAITLISTED

Öğrenci etkinliğe katılmak için yedek listede bulunmaktadır.

Yedek liste özellikle kapasitesi bulunan ve başvuruların yönetici tarafından değerlendirildiği etkinliklerde kullanılabilir.

Yedek listedeki öğrenciler yönetici tarafından belirlenir ve başvuru zamanına göre sıralanır.

Kapasite açıldığında sistem yedek listedeki ilk öğrenciyi otomatik olarak `ACCEPTED` durumuna geçirir.

### WITHDRAWN

Öğrenci başvurusunu etkinlik başlamadan önce geri çekmiştir.

`WITHDRAWN` durumundaki bir başvurunun yönetici tarafından ayrıca işleme alınması gerekmez.

`BAN` bir application status değildir.

`NOT_ATTENDED` bir application status değildir; attendance ayrı bir kavramdır.

---

# 5. Başvuruların Görüntülenmesi

Kulüp yöneticisi kendi etkinliklerinden birini seçerek o etkinliğe ait başvuruları görüntüleyebilir.

Başvurular durumlarına göre filtrelenebilir:

* Bekleyenler (`PENDING`)
* Kabul edilenler (`ACCEPTED`)
* Reddedilenler (`REJECTED`)
* Yedek listedekiler (`WAITLISTED`)
* Geri çekilenler (`WITHDRAWN`)

Başvuru listesinde en azından aşağıdaki bilgiler bulunmalıdır:

* Öğrenci
* Başvuru tarihi
* Başvuru durumu
* Başvuru durumunun değişme tarihi
* Varsa ret nedeni

Yönetici özellikle `APPROVAL_REQUIRED` etkinliklerde `PENDING` başvuruları değerlendirerek öğrencileri kabul edebilir, reddedebilir veya gerekli görürse yedek listeye alabilir.

---

# 6. PENDING Başvurunun Yönetilmesi

Bir başvuru `PENDING` durumundaysa kulüp yöneticisi, etkinliğin başvuru tipine göre işlem yapar.

`APPROVAL_REQUIRED` etkinliklerde yönetici:

```text
PENDING → ACCEPTED

PENDING → REJECTED

PENDING → WAITLISTED
```

geçişlerinden uygun olanını gerçekleştirebilir.

### 6.1. Kabul

Yönetici başvuruyu kabul ettiğinde:

```text
PENDING → ACCEPTED
```

durumu oluşur.

Kabul işlemi etkinliğin kapasite kurallarına tabidir.

Yönetici kapasite dahilindeki öğrencileri kabul edebilir.

### 6.2. Ret

Yönetici başvuruyu reddettiğinde:

```text
PENDING → REJECTED
```

durumu oluşur.

Ret nedeni konusu henüz kesinleştirilmemiştir.

> **Açık karar:** Normal bir ret işleminde yöneticinin ret nedeni girmesinin zorunlu olup olmayacağı daha sonra belirlenecektir.

### 6.3. Yedek Listeye Alma

Yönetici kapasite dolduktan sonra bazı `PENDING` başvuruları yedek listeye alabilir:

```text
PENDING → WAITLISTED
```

Yönetici yedek listeye aldığı öğrencileri başvuru zamanına göre sıralar.

Yedek listeye alınan öğrenciler etkinlik için uygun görülmüş ancak mevcut kontenjan içerisinde doğrudan kabul edilmemiş öğrencilerdir.

Yedek liste yöneticinin manuel olarak belirlediği bir listedir. Yönetici öğrencileri yedek listeye alırken başvuru zaman sırası esas alınır.

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

### ACCEPTED → REJECTED Sonrası Kapasite

`ACCEPTED → REJECTED` geçişi kapasiteyi boşaltır.

Boşalan kapasite varsa yedek listedeki ilk öğrenci sistem tarafından otomatik olarak kabul edilir.

```text
ACCEPTED → REJECTED
        ↓
Kapasite açıldı
        ↓
WAITLISTED sıradaki aday
        ↓
ACCEPTED
```

Yedek listedeki öğrencilerin sırası başvuru zamanına göre korunur.

---

## 7.1. ACCEPTED → PENDING Yasaktır

Kabul edilmiş bir başvuru tekrar `PENDING` durumuna getirilemez.

```text
ACCEPTED → PENDING ❌
```

---

# 8. REJECTED Başvurunun Yönetilmesi

Reddedilmiş bir başvuru için öğrenci yeniden başvuru yapamaz.

Ancak kulüp yöneticisi önceki ret kararını düzeltebilir:

```text
REJECTED → ACCEPTED
```

Bu işlem yeni bir başvuru oluşturmaz. Mevcut başvurunun durumu doğrudan `ACCEPTED` yapılır.

### Kapasite Kontrolü

`REJECTED → ACCEPTED` işlemi kapasite kurallarına tabidir.

Etkinlik kapasitesi doluysa bu işleme izin verilmez.

```text
Kapasite: 50

ACCEPTED: 50

REJECTED → ACCEPTED ❌
```

Yönetici önce etkinlik kapasitesini artırmalıdır veya mevcut kapasitede yer açılmış olmalıdır.

```text
Kapasite: 50 → 51

        ↓

REJECTED → ACCEPTED ✅
```

Kapasite aşımına izin verilmez.

### REJECTED → PENDING Yasaktır

```text
REJECTED → PENDING ❌
```

Yönetici öğrenciyi tekrar değerlendirmek istiyorsa `PENDING` durumuna geri döndürmek yerine doğrudan kabul edebilir.

---

# 9. WAITLISTED Başvurular

Yedek liste bir **yedek katılımcı listesidir**.

Yedek listedeki öğrenciler:

* Yönetici tarafından belirlenir.
* Başvuru zamanına göre sıralanır.
* Yönetici tarafından manuel olarak yeniden sıralanamaz.
* Kapasite açıldığında sistem tarafından otomatik olarak işlenir.

Öncelik:

```text
Daha erken başvuru
        ↓
Daha yüksek öncelik
```

Yönetici yedek listeyi oluştururken öğrencilerin başvuru zamanını esas alır.

Yedek listeye alınmış bir öğrencinin etkinliğe uygun görülmesi, öğrencinin doğrudan kabul edildiği anlamına gelmez. Öğrenci ancak kapasite açıldığında kabul edilir.

---

# 10. Kapasite Açıldığında Otomatik İşlem

Kapasitede yer açıldığında sistem yedek listesini başvuru zamanına göre işler.

Yedek listedeki ilk öğrenci:

```text
WAITLISTED → ACCEPTED
```

durumuna geçirilir.

Bu işlemde yönetici tarafından ayrıca öğrenci seçilmesine gerek yoktur.

### Örnek

Etkinlik kapasitesi:

```text
50 kişi
```

Mevcut durum:

```text
50 ACCEPTED
20 WAITLISTED
```

Kabul edilmiş bir öğrenci etkinliğe katılamayacağını bildirir ve başvurusunu geri çeker:

```text
49 ACCEPTED
20 WAITLISTED
```

Sistem yedek listedeki ilk öğrenciyi kabul eder:

```text
1. WAITLISTED → ACCEPTED
```

Sonuç:

```text
50 ACCEPTED
19 WAITLISTED
```

Aynı kural yönetici tarafından kabul edilmiş bir başvurunun reddedilmesi sonucunda açılan kapasite için de geçerlidir.

---

# 11. Yedek Listenin Oluşturulması

`APPROVAL_REQUIRED` ve kapasite sınırı bulunan etkinliklerde öğrencilerin tamamı başvuru süresi boyunca `PENDING` durumunda bulunabilir.

Örneğin:

```text
Kapasite: 50

Başvuru sayısı: 100

PENDING: 100
```

Başvuru süresi sona erdikten sonra yönetici başvuruları değerlendirir.

Örneğin yönetici:

```text
50 → ACCEPTED

20 → WAITLISTED

30 → REJECTED
```

şeklinde sonuçlandırabilir.

Bu durumda:

```text
ACCEPTED: 50
WAITLISTED: 20
REJECTED: 30
PENDING: 0
```

olur.

Yönetici yedek listeye aldığı 20 öğrenciyi başvuru zamanına göre sıralar.

Örneğin:

```text
09:01 → Ahmet
09:05 → Mehmet
09:12 → Ayşe
...
```

şeklinde bir sıra oluşur.

Daha sonra bir kişilik kapasite açıldığında:

```text
Ahmet → ACCEPTED
```

olur.

Yedek liste:

```text
19 kişi
```

olarak devam eder.

### Başvuruların Tamamlanması

Yönetici kabul ve yedek liste kararlarını verdikten sonra, artık kontenjan kalmadığı ve yeterli yedek aday belirlendiği durumda kalan `PENDING` başvurular `REJECTED` durumuna geçirilebilir.

Bu işlem sistem tarafından otomatik olarak da gerçekleştirilebilir.

Örneğin:

```text
Kapasite: 50

100 PENDING
```

Yönetici:

```text
50 ACCEPTED
20 WAITLISTED
15 REJECTED
```

belirledikten sonra kalan:

```text
15 PENDING
```

başvuru sistem tarafından:

```text
15 PENDING → REJECTED
```

olarak sonuçlandırılabilir.

Böylece etkinliğin başvuru değerlendirme sürecinde `PENDING` durumda başvuru kalmaz.

---

# 12. Kapasitenin Artırılması

Etkinliğin kapasitesi artırıldığında mevcut yedek liste sistem tarafından başvuru zamanına göre işlenir.

Örneğin:

```text
Kapasite: 50 → 70

20 kişilik yeni kapasite oluştu.
```

Yedek listede öğrenciler varsa ilk sıradaki öğrenciler otomatik olarak kabul edilir.

```text
WAITLISTED → ACCEPTED
```

Örneğin:

```text
WAITLISTED: 20

Kapasite +20

        ↓

20 WAITLISTED → ACCEPTED
```

Sonuç:

```text
ACCEPTED: 70
WAITLISTED: 0
```

`APPROVAL_REQUIRED` etkinlikte ise kapasite artışı, başvuruların henüz yönetici tarafından değerlendirilmediği bir aşamadaysa yöneticinin daha fazla `PENDING` başvuruyu kabul etmesine olanak sağlar.

Başvurular sonuçlandırılıp yedek liste oluşturulduktan sonra kapasite artışı olması durumunda ise yedek listedeki öğrenciler sırayla `ACCEPTED` durumuna geçirilir.

---

# 13. Başvuru Tarihi Sıralaması

Yedek listedeki öncelik başvurunun oluşturulma zamanına göre belirlenir.

Örneğin:

```text
09:01 → Ahmet

09:05 → Mehmet

09:12 → Ayşe

09:20 → Zeynep
```

Kapasite 2 kişi açarsa:

```text
Ahmet

Mehmet
```

öncelikli olarak kabul edilir.

Yönetici bu sırayı manuel olarak değiştiremez.

Bu zaman sıralaması, öğrencilerin yedek listeye alınması sırasında da esas alınır.

---

# 14. Başvuru Geri Çekme (WITHDRAWN)

Öğrenci başvurusunu etkinlik başlamadan önce geri çekebilir.

Geçerli geri çekme geçişleri:

```text
PENDING    → WITHDRAWN

WAITLISTED → WITHDRAWN

ACCEPTED   → WITHDRAWN
```

`REJECTED` durumundaki başvurunun geri çekilmesine gerek yoktur.

Etkinlik başladıktan sonra başvuru geri çekilemez.

### ACCEPTED → WITHDRAWN Sonrası Kapasite

```text
ACCEPTED → WITHDRAWN
        ↓
Kapasite açıldı
        ↓
WAITLISTED sıradaki aday
        ↓
ACCEPTED
```

Yedek liste başvuru zamanına göre normal kurallara göre işlenir.

### Geri Çekilen Öğrencinin Tekrar Başvurması

`WITHDRAWN` durumuna geçen öğrenci, başvuru süreci hâlâ açıksa aynı etkinliğe tekrar başvurabilir.

Bu yeni başvuru, eski başvurunun devamı değildir; yeni bir application kaydı olarak değerlendirilir.

Eski `WITHDRAWN` başvuru kaydı silinmez; başvuru geçmişi korunur.

---

# 15. Başvuru Son Başvuru Tarihinden Sonra

Başvuru süresi sona erdiğinde yeni öğrenci başvuruları alınmaz.

Ancak bu durum mevcut başvuruların yönetilmesini engellemez.

`APPROVAL_REQUIRED` etkinliklerde yönetici:

* `PENDING` başvuruları kabul edebilir.
* `PENDING` başvuruları reddedebilir.
* Uygun gördüğü öğrencileri yedek listeye alabilir.
* Kapasiteyi artırabilir.
* Açılan kapasite sonucunda mevcut yedek listenin otomatik olarak işlenmesini sağlayabilir.

Örneğin:

```text
Başvuru süresi bitti

        ↓

Yeni başvuru ❌

        ↓

Mevcut başvuruları yönetme ✅
```

Başvurular yönetici tarafından sonuçlandırıldığında artık `PENDING` durumda başvuru bırakılmaması hedeflenir.

---

# 16. Etkinlik Başladıktan Sonra

Etkinlik başladıktan sonra başvuru kararlarında aşağıdaki işlemler yapılamaz:

```text
ACCEPTED → REJECTED ❌

PENDING  → ACCEPTED ❌

PENDING  → REJECTED ❌

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
PENDING → WAITLISTED   (APPROVAL_REQUIRED, yönetici tarafından)
PENDING → WITHDRAWN    (öğrenci tarafından)

REJECTED → ACCEPTED     (yönetici tarafından, kapasite uygunsa)

WAITLISTED → ACCEPTED   (kapasite açıldığında, sistem tarafından)

WAITLISTED → WITHDRAWN  (öğrenci tarafından)

ACCEPTED → REJECTED     (yönetici tarafından, etkinlik başlamadan önce)

ACCEPTED → WITHDRAWN    (öğrenci tarafından, etkinlik başlamadan önce)
```

`PUBLIC` etkinliklerde sistem tarafından gerçekleştirilen otomatik başvuru sonucu:

```text
PENDING → ACCEPTED
```

ve kapasite dolduğunda:

```text
PENDING → WAITLISTED
```

olabilir.

## Geçersiz geçişler

```text
ACCEPTED  → PENDING       ❌

REJECTED  → PENDING       ❌

REJECTED  → WAITLISTED    ❌

REJECTED  → yeni başvuru  ❌
```

`WITHDRAWN` sonrası öğrenci yeni bir başvuru oluşturabilir (eski kayıt silinmez). Bkz. Bölüm 14.

---

# 20. Öğrencinin Aynı Etkinliğe Başvurması

Bir öğrenci aynı etkinlik için aynı anda yalnızca bir aktif başvuru kaydına sahip olabilir.

Aktif başvuru durumları:

```text
PENDING

ACCEPTED

WAITLISTED
```

`REJECTED` durumundaki bir başvurudan sonra öğrenci yeni başvuru oluşturamaz.

Yönetici kararını değiştirmek isterse:

```text
REJECTED → ACCEPTED
```

geçişini kullanır.

Yeni bir başvuru oluşturulmaz.

`WITHDRAWN` durumundaki başvurudan sonra öğrenci, başvuru süresi açıksa yeni başvuru oluşturabilir.

Bu yeni başvuru eski kaydın devamı değildir; ayrı bir application kaydıdır.

Eski kayıt silinmez.

---

# 21. Yetki Özeti

| İşlem                                  | Başkan | Başkan Yardımcısı | Yönetici |
| -------------------------------------- | -----: | ----------------: | -------: |
| Başvuruları görüntüleme                |      ✅ |                 ✅ |        ✅ |
| PENDING → ACCEPTED                     |      ✅ |                 ✅ |        ✅ |
| PENDING → REJECTED                     |      ✅ |                 ✅ |        ✅ |
| PENDING → WAITLISTED                   |      ✅ |                 ✅ |        ✅ |
| ACCEPTED → REJECTED                    |      ✅ |                 ✅ |        ✅ |
| REJECTED → ACCEPTED (kapasite uygunsa) |      ✅ |                 ✅ |        ✅ |
| Bekleme listesini oluşturma            |      ✅ |                 ✅ |        ✅ |
| Bekleme listesini manuel sıralama      |      ❌ |                 ❌ |        ❌ |
| Katılım kaydı ekleme                   |      ✅ |                 ✅ |        ✅ |
| Katılım kaydı silme/düzeltme           |      ✅ |                 ✅ |        ✅ |

Yedek listedeki öğrencilerin sırası başvuru zamanına göre belirlendiği için yöneticilerin bu sırayı manuel olarak değiştirmesine izin verilmez.

Başvuru geri çekme (`WITHDRAWN`) öğrenci tarafından gerçekleştirilir, yönetici tarafından değil.

---

# 22. Sistem Tarafından Otomatik Gerçekleştirilen İşlemler

Sistem aşağıdaki işlemleri otomatik olarak gerçekleştirir:

### PUBLIC başvurularının sonuçlandırılması

```text
Başvuru

   ↓

Başvuru koşulları kontrol edilir

   ↓

Kapasite uygunsa
PENDING → ACCEPTED

Kapasite doluysa
PENDING → WAITLISTED
```

### Kapasite açılması

```text
Kapasite açıldı

      ↓

WAITLISTED öğrenciler başvuru zamanına göre sıralanır

      ↓

İlk öğrenci

      ↓

ACCEPTED
```

### Başvuru değerlendirmesinin tamamlanması

`APPROVAL_REQUIRED` etkinliklerde yönetici başvuruları değerlendirir.

```text
PENDING

   ├──→ ACCEPTED
   │
   ├──→ REJECTED
   │
   └──→ WAITLISTED
```

Yeterli sayıda kabul ve yedek liste belirlendikten sonra sonuçlandırılmamış kalan başvurular sistem tarafından `REJECTED` durumuna geçirilebilir.

### Etkinlik başlangıcı

```text
Etkinlik başladı

      ↓

Başvuru kararları artık değiştirilemez

      ↓

Attendance süreci devam eder
```

---

# 23. Use Case Akışı

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
┌──────────────────────────────────────┐
│                                      │
PENDING                            ACCEPTED
│                                      │
├─ Kabul → ACCEPTED                    └─ Ret → REJECTED
│                                             ↓
├─ Ret → REJECTED                    (Kapasite açılır)
│                                             ↓
└─ Yedek → WAITLISTED                WAITLISTED işlenir
                                              ↓
                                         ACCEPTED

WAITLISTED
   │
   └─ Kapasite açılması
          ↓
      ACCEPTED
```

`PUBLIC` etkinliklerde başvurular sistem tarafından otomatik olarak sonuçlandırılır:

```text
Başvuru
   ↓
PENDING
   │
   ├── Kapasite uygunsa → ACCEPTED
   │
   └── Kapasite doluysa → WAITLISTED
```

`APPROVAL_REQUIRED` etkinliklerde:

```text
Başvuru
   ↓
PENDING
   ↓
Yönetici değerlendirir
   │
   ├──→ ACCEPTED
   │
   ├──→ REJECTED
   │
   └──→ WAITLISTED
```

Yedek listeden kapasite açıldığında:

```text
WAITLISTED
   ↓
Başvuru zamanına göre sıradaki öğrenci
   ↓
ACCEPTED
```

---

# 24. Karar Verilmemiş Konular

Aşağıdaki konular bu use case kapsamında henüz kesinleştirilmemiştir:

* Normal `PENDING → REJECTED` işleminde ret nedeninin zorunlu olup olmayacağı.
* Ret nedenlerinin serbest metin mi yoksa önceden belirlenmiş seçenekler mi olacağı.
* Ret nedeni yapısının veri modelinde nasıl tutulacağı (serbest metin, kategori vb.).
* Başvuru listesinde hangi öğrenci bilgilerinin yöneticilere gösterileceği.
* Başvuru geçmişinin ne kadar ayrıntılı tutulacağı.
* Öğrencinin `ACCEPTED → WITHDRAWN` işlemini etkinlik başlamadan ne kadar süre öncesine kadar gerçekleştirebileceği.
* Attendance düzeltmelerinin audit/history kayıtlarının kullanıcıya gösterilip gösterilmeyeceği.

Bu kararlar sonraki aşamada netleştirilecektir.

---

# 25. Gelecekte Planlanmış Özellikler

Ban/engelleme sistemi bu aşamada MVP kapsamında değildir.

Gelecekte bir kulüp bir kullanıcıyı engellerse bu durumun başvuru akışına etkisi ayrıca tasarlanacaktır.

Bu konu `requirements.md` Bölüm 10'da "Gelecekte Engelleme Sistemi" başlığı altında tanımlanmıştır.

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
