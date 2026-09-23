# İş Kuralları — Etkinlik ve Başvuru Sistemi

## 1. Etkinlik Kuralları

* Etkinlikler **PHYSICAL** veya **ONLINE** olabilir.
* Etkinlik oluşturma ve yayınlama tek işlemdir.
* Etkinlik oluşturulduğunda **PUBLISHED** durumundadır.
* Etkinlik daha sonra **COMPLETED** veya **CANCELLED** olabilir.
* Etkinlik adı yayınlandıktan sonra değiştirilemez.
* Açıklama, görsel, konum/link, başlangıç ve bitiş zamanı etkinlik başlamadan önce değiştirilebilir.
* Etkinlik türü ve başvuru türü yayınlandıktan sonra değiştirilemez.

## 2. Başvuru Türleri

Etkinliklerin iki başvuru türü vardır:

* **PUBLIC**
* **APPROVAL_REQUIRED**

## 3. Kapasite Kuralları

* Etkinliğin kapasitesi olabilir veya kapasite sınırı olmayabilir.
* Etkinliğe başvuruda kapasite sınırı yoktur.
* Başvuru başladıktan sonra kapasite azaltılamaz.
* Başvuru başladıktan sonra kapasite artırılabilir.
* Başvuru başlamadan önce kapasite artırılabilir veya azaltılabilir.

## 4. PUBLIC Başvurular

### Kapasitesiz etkinlik

Başvuran öğrencinin başvurusu otomatik olarak **ACCEPTED** olur.

### Kapasiteli etkinlik

* Kapasite dolana kadar başvurular **ACCEPTED** olur.
* Kapasite dolduktan sonra gelen başvurular **WAITLISTED** olur.
* Öğrencinin başvurması kapasite dolu olsa bile engellenmez.
* Bir **ACCEPTED** başvuru ayrıldığında, bekleme listesindeki ilk uygun başvuru otomatik olarak **ACCEPTED** olur.

## 5. APPROVAL_REQUIRED Başvurular

### Kapasitesiz etkinlik

* Başvuran öğrencilerin başvuruları **PENDING** olur.
* Yönetici başvuruları **ACCEPTED** veya **REJECTED** yapabilir.
* Bekleme listesi kullanılmaz.

### Kapasiteli etkinlik

* Başvuran öğrencilerin başvuruları **PENDING** olur.
* Kapasite dolu olsa bile başvuru yapılabilir.
* Yönetici başvuruları **ACCEPTED**, **REJECTED** veya **WAITLISTED** yapabilir.
* WAITLISTED başvurular yönetici tarafından seçilir.
* WAITLISTED başvurular kapasite açıldığında başvuru tarihine göre sırayla **ACCEPTED** olur.

## 6. Başvuru Durumları

Bir başvuru aşağıdaki durumlardan birinde olabilir:

* **PENDING**
* **ACCEPTED**
* **REJECTED**
* **WAITLISTED**
* **WITHDRAWN**

**NOT_ATTENDED** başvuru durumu değildir. Katılım ayrı olarak takip edilir.

## 7. Başvuru Geçişleri

Geçerli geçişler:

* PENDING → ACCEPTED
* PENDING → REJECTED
* PENDING → WAITLISTED
* PENDING → WITHDRAWN
* WAITLISTED → ACCEPTED
* WAITLISTED → WITHDRAWN
* ACCEPTED → REJECTED
* ACCEPTED → WITHDRAWN
* REJECTED → ACCEPTED

**PENDING → WAITLISTED** yalnızca kapasitesi olan **APPROVAL_REQUIRED** etkinliklerde ve yönetici kararıyla gerçekleşebilir.

**WAITLISTED → ACCEPTED** sistem tarafından kapasite açıldığında gerçekleştirilir.

## 8. Başvuru Geri Çekme

* Öğrenci, etkinlik başlamadan önce **PENDING**, **WAITLISTED** veya **ACCEPTED** başvurusunu geri çekebilir.
* Başvuru **WITHDRAWN** olur.
* ACCEPTED başvuru geri çekilirse kapasite açılır.
* Açılan kapasite varsa bekleme listesindeki ilk uygun başvuru otomatik olarak ACCEPTED olur.
* Öğrenci, başvuru süresi devam ediyorsa geri çektiği etkinliğe yeniden başvurabilir.
* Eski WITHDRAWN başvuru kaydı korunur.

## 9. Bekleme Listesi

* PUBLIC etkinliklerde kapasite dolduğunda bekleme listesi sistem tarafından oluşturulur.
* APPROVAL_REQUIRED etkinliklerde bekleme listesi yönetici tarafından belirlenir.
* APPROVAL_REQUIRED etkinliklerde kapasitenin dolu olması tek başına başvuruyu WAITLISTED yapmaz.
* Bekleme listesindeki adayların önceliği başvuru tarihine göre belirlenir.
* Yönetici bu sırayı değiştiremez.
* Kapasite açıldığında ilk sıradaki aday ACCEPTED olur.

## 10. Kapasite Artırma

Kapasite artırıldığında açılan yeni kontenjanlar mevcut bekleme listesi üzerinden değerlendirilir.

* PUBLIC etkinliklerde bekleme listesindeki adaylar sırayla ACCEPTED olur.
* APPROVAL_REQUIRED etkinliklerde daha önce yönetici tarafından WAITLISTED seçilmiş adaylar sırayla ACCEPTED olur.
* APPROVAL_REQUIRED etkinlikte henüz değerlendirilmemiş PENDING başvurular varsa yönetici yeni kapasiteyi dikkate alarak değerlendirme yapabilir.

## 11. Son Başvuru Tarihi

* Son başvuru tarihinden sonra yeni başvuru alınmaz.
* Mevcut başvuruların yönetici tarafından değerlendirilmesi devam edebilir.
* APPROVAL_REQUIRED etkinliklerde yönetici PENDING başvuruları ACCEPTED, REJECTED veya WAITLISTED olarak sonuçlandırabilir.
* Kapasite dolduğunda ve gerekli bekleme listesi oluşturulduğunda kalan kararsız başvurular REJECTED olarak sonuçlandırılabilir.

## 12. Aynı Etkinliğe Birden Fazla Başvuru

* Öğrencinin aynı etkinlikte aynı anda yalnızca bir aktif başvurusu olabilir.
* Aktif başvuru durumları: PENDING, ACCEPTED, WAITLISTED.
* REJECTED durumundaki öğrenci yeniden başvuramaz.
* WITHDRAWN durumundaki öğrenci, başvuru süresi devam ediyorsa yeniden başvurabilir.

## 13. Etkinlik Başladıktan Sonra

Etkinlik başladıktan sonra:

* Yeni başvuru alınamaz.
* Başvuru kararları değiştirilemez.
* Katılım işlemleri başvuru durumundan bağımsız olarak yürütülür.

## 14. Katılım

* ACCEPTED olmak etkinliğe katılmış olmak anlamına gelmez.
* Katılım ayrı bir kayıt olarak tutulur.
* Katılım bilgisi yönetici tarafından manuel olarak eklenebilir, silinebilir veya düzeltilebilir.
* Fiziksel etkinliklerde QR ile katılım alınması planlanmaktadır.

## 15. Yetki Kuralları

### Öğrenci

* Etkinlikleri görüntüleyebilir.
* Başvuru yapabilir.
* Kendi başvurusunu geri çekebilir.
* Kendi başvuru durumunu görüntüleyebilir.

### Kulüp Yöneticisi

* Kendi kulübünün etkinliklerini yönetebilir.
* Başvuruları görüntüleyebilir.
* Başvuruları kabul veya reddedebilir.
* Uygun durumlarda başvuruyu bekleme listesine alabilir.
* Kabul edilmiş başvuruyu reddedebilir.
* Reddedilmiş başvuruyu tekrar kabul edebilir.
* Katılım kayıtlarını yönetebilir.

## 16. Bildirimler

Bildirim sistemi MVP kapsamında değildir.

Bildirimler, temel etkinlik ve başvuru işlemleri çalıştıktan sonra **V2** kapsamında eklenebilir.

## 17. MVP Dışı

Aşağıdaki özellikler mevcut MVP kapsamında değildir:

* Bildirim sistemi
* Kullanıcı engelleme / ban
* Etkinlik puanlama
* Kulüp / etkinlik takip sistemi
* Gelişmiş keşif ve öneri sistemi
* E-posta / SMS bildirimleri
