# Gizlilik metni — kaynak inceleme notu

**Tarih:** 6 Eylül 2026
**İncelenen depo:** `FootballAgent_Mobile`, `main` dalı, HEAD `774ca47`
**Amaç:** `/privacy/pro-football-agent/{en,tr}.html` metinlerini varsayımla değil,
kodun gerçekte yaptığıyla yazmak.

Depoda hiçbir dosya değiştirilmedi, hiçbir build çalıştırılmadı. `keystore.properties`,
`keystore.properties.example`, `local.properties` ve imzalama anahtarları hiç okunmadı;
`android/app/build.gradle` yalnızca reklam/analitik bağımlılığı olup olmadığı için okundu
ve içinde zaten hiçbir gizli değer yok (yorumların dediği gibi, gerçek değerler depo dışında).

---

## 1. Kayıtlar ve tercihler nerede tutuluyor?

**Bulgu:** Her şey cihazdaki uygulama depolamasında. Sunucu yok. Ama **tek bir yerde
değil** — iki ayrı mekanizma var ve gizlilik metni bunları ayrı ayrı anlatmak zorunda:

| Ne | Nerede | Geri düşüş var mı? | Referans |
|---|---|---|---|
| Kariyer kayıtları (3 yuva: `s1`,`s2`,`s3`) | IndexedDB, veritabanı adı `menajer` | Evet → localStorage `menajerSaveV9s1..s3` | `js/store.js:116` (`DB_NAME`), `js/store.js:173` (`LSKEY`), `js/store.js:176-198` |
| Yuva özetleri (ana menü listesi) | Aynı katman: `meta` anahtarı, yani IndexedDB | Evet → localStorage `menajerMetaV1` | `js/saves.js:56-60` (`META`, `metaDirty`), `js/store.js:173` |
| Cihaz tercihleri (tema, dil, ses) | **Doğrudan localStorage:** `menajerPrefsV1` | Hayır — hiç IndexedDB'ye girmiyor | `js/saves.js:35`, `js/saves.js:42-49` |
| Eski sürümlerden göç edilen anahtarlar | `menajerSaveV9` (tek kayıtlı sürüm) — yalnız okunup siliniyor | — | `js/saves.js:33`, `js/saves.js:178-196` |

**Ayrımın nedeni koda yazılmış:** kayıtlar büyük olduğu için localStorage kotasına
sığmıyor ve IndexedDB'ye taşınmışlar (`js/store.js:4-31`); tercihler ise ilk çizimden
önce **senkron** okunmak zorunda olduğu için bilerek localStorage'da bırakılmış
(`js/saves.js:20-22`). Yani tercihler için localStorage bir geri düşüş değil, birincil
ve tek depolama. İlk taslakta ikisi tek cümlede toplanmıştı; düzeltildi.

`store.js` geri düşüşe yalnız IndexedDB açılamazsa ya da `DB_TIMEOUT` (5 sn) dolarsa
geçiyor (`js/store.js:120-128`); bu durumda oyun çalışmaya devam ediyor, yalnız eski
kota sınırı geri geliyor.

> **Bu paragraflar bilerek yalnız burada.** Kayıt boyutu, kota, senkron okuma zorunluluğu
> ve `DB_TIMEOUT` birer geliştirme gerekçesi; kullanıcının gizliliği açısından bir şey
> değiştirmiyorlar. Kullanıcıya görünen metinde yalnız **neyin nerede tutulduğu** kaldı:
> kayıtlar ve yuva özetleri IndexedDB'de (kullanılamıyorsa localStorage'da), tercihler
> her zaman doğrudan localStorage'da.

**Kaydın içinde ne var:** oyun dünyası ve ilerleme. Oyuncunun yazdığı serbest metin
ajans/menajer adı (`js/core.js:409-410`, ajans adı boş bırakılırsa soyaddan türetiliyor).
Kodda gerçek ad, e-posta, telefon, yaş veya konum soran hiçbir alan yok — **ama bu
alanlar serbest metin, oyuncunun oraya gerçek adını yazmasını engelleyen bir şey yok.**
Bu yüzden metinde "bunlar senin uydurduğun değerler" denmiyor; "uygulama bunu istemiyor,
ne yazarsan yaz cihazında kalıyor" deniyor.

**Tercihlerin tam listesi** — üç tane, hepsi `setPref` ile yazılıyor:
`theme` (`js/ui.js:260`), `lang` (`js/ui.js:3451`), `sfxOn` (`js/ui.js:2350`).

## 2. Hesap açma, sunucuya kayıt gönderme, bulut senkronizasyonu?

**Bulgu: yok.** Kodda oturum, kimlik doğrulama, kullanıcı kaydı ya da uzak
depolama kavramı geçmiyor. Kayıt/okuma yollarının tamamı `js/store.js` üzerinden
IndexedDB/localStorage'a gidiyor.

## 3. Harici ağ istekleri, telemetri, analitik, reklam SDK'sı?

**Bulgu: yok.**

- `js/` altındaki 17 dosyanın tamamında tek bir `fetch(`, `XMLHttpRequest`,
  `sendBeacon`, `WebSocket` ya da `EventSource` çağrısı yok. Depodaki tek `fetch`
  `sw.js:109` — service worker'ın kendi önbelleği için, yalnız uygulamanın
  paketlenmiş dosyalarına gidiyor (`sw.js:6-40`, `SHELL` listesi hep göreli yol).
- **Service worker Android uygulamasında hiç çalışmıyor.** Kayıt koşulu
  `js/main.js:35`: `'serviceWorker' in navigator && location.protocol.startsWith('http') && !window.Capacitor`.
  Capacitor içindeyken atlanıyor.
- Capacitor eklenti listesi boş: `android/app/src/main/assets/capacitor.plugins.json` → `[]`.
  Kurulu paketler yalnız `@capacitor/core`, `@capacitor/cli`, `@capacitor/android`
  (`package.json`, `node_modules/@capacitor/`).
- `android/build.gradle:11` içinde `com.google.gms:google-services` classpath'i var
  (Capacitor şablonunun varsayılanı), ama eklenti **uygulanmıyor**:
  `android/app/build.gradle` sondaki `try` bloğu `google-services.json` varsa
  uyguluyor, o dosya ise depoda yok (`android/app/google-services.json` — mevcut değil).
  Yani Firebase/GMS bağımlılığı derlemeye girmiyor.
- AdMob, Analytics, Crashlytics, billing ya da başka bir SDK'ya kaynakta hiçbir
  referans yok.

**INTERNET izni var ama kullanılmıyor.** `android/app/src/main/AndroidManifest.xml`
sonunda `android.permission.INTERNET` bildirilmiş — Capacitor şablonunun varsayılanı.
Birleştirilmiş manifest'te izinler yalnız bu ikisi:
`android.permission.INTERNET` ve Capacitor'ın kendi
`com.xahke.profootballagent.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION` izni
(`android/app/build/intermediates/merged_manifest/release/.../AndroidManifest.xml:13,19`).
Konum, kamera, mikrofon, rehber, dosya izni yok.

## 4. Android yedekleme yapılandırması neye izin veriyor?

**Bulgu:** `android:allowBackup="true"`, `fullBackupContent` veya
`dataExtractionRules` **tanımlı değil** (`android/app/src/main/AndroidManifest.xml`).
`targetSdkVersion = 36` (`android/variables.gradle`).

Yani Android'in varsayılan *Auto Backup for Apps* davranışı geçerli: uygulamanın
iç depolaması (WebView'in IndexedDB/localStorage verisi dâhil) kullanıcının kendi
Google Drive hesabındaki özel bir klasöre yedeklenebiliyor ve cihazdan cihaza
aktarımda geri yüklenebiliyor. Android'in belgelerine göre kota uygulama başına
25 MB, yalnız en son yedek tutuluyor, kullanıcı sistem ayarlarından kapatabiliyor.

**Uçtan uca şifrelemenin koşulu var.** Android 9+ üzerinde yedek cihazın ekran
kilidiyle uçtan uca şifreleniyor — bu koruma cihazda gerçekten bir PIN/desen/parola
tanımlı olmasına bağlı. İlk taslak "Android 9 ve sonrasında yedek şifrelenir" diye
koşulsuz yazmıştı; düzeltildi.

**Yapılandırma ≠ sınanmış davranış.** Burada doğrulanan şey manifest'in neye izin
verdiği. Yedeğin gerçekten alınıp alınmadığı, ne zaman alındığı ve içine tam olarak
ne girdiği Android'in kendi koşullarına bağlı (yedekleme ayarı, ağ, cihazın boşta
olması). **Bir cihazda yedek alınıp geri yüklenerek sınanmadı.**

> **Sınanmamış olması bilerek yalnız burada yazıyor.** Bu bir geliştirme durumu, bir
> veri işleme gerçeği değil; kullanıcı metnine konduğunda asıl söylenmesi gerekeni
> gölgeliyordu. Kullanıcıya görünen metinde korunan üç şey: verinin yedeğe **girebileceği**,
> yedeği **Android/Google'ın** yönettiği ve geliştiricinin erişemediği, uçtan uca
> şifrelemenin **ekran kilidi tanımlı olmasına bağlı** olduğu. Yedeklemenin bir cihazda
> doğrulanması `pre-launch-checklist.md` içinde açık iş olarak duruyor.

**Metne nasıl girdi:** ayrı bir "Android yedeklemesi" başlığı olarak, işi
Android/Google'ın yaptığı ve geliştiricinin erişimi olmadığı açıkça belirtilerek.
Kayıtların "hiçbir yere gitmediği" iddiası bu bölüm olmadan yanlış olurdu; silme
bölümündeki kesinlik de bu yüzden yalnız **yerel** veriyle sınırlandı.

## 5. Kullanıcı kayıtlarını nasıl siliyor?

- **Tek kariyer:** ana menüdeki yuva kartında "Kariyeri sil" (`js/ui.js:225`,
  `cmDeleteSlot` → `js/ui.js:48`) veya kariyer içindeyken Ayarlar ekranının altındaki
  aynı düğme (`js/ui.js:2369`, `askDeleteCareer` → `js/ui.js:55`).
  İkisi de `deleteSlot(n)` çağırıyor (`js/saves.js:114-116`): META'dan düşürüyor ve
  depolama katmanından siliyor (`js/store.js:190`).
- **Her şey:** Android ayarlarından uygulama verilerini temizlemek ya da uygulamayı kaldırmak.
- Uygulama içinde "tüm verileri sil" diye tek bir düğme yok; üç yuva ayrı ayrı siliniyor.
  Metin bunu olduğu gibi anlatıyor, olmayan bir düğmeyi tarif etmiyor.
- **Bu yolların hepsi yalnız cihazdaki veriye ulaşıyor.** Yedek kopyası varsa ona
  dokunmuyorlar; o ayrıca yönetiliyor (bkz. 4. madde). İlk taslaktaki "veri gerçekten
  gidiyor" ifadesi bu ayrımı bulanıklaştırıyordu; kesinlik yerel veriyle sınırlandı.
- **Uygulama depolamasının yalıtımı platformdan geliyor.** Android her uygulamaya
  başka uygulamaların olağan yollarla erişemediği kendi alanını veriyor; oyun bunun
  üstüne ek bir koruma (şifreleme vb.) koymuyor. İlk taslaktaki "başka uygulamalar
  bunu okuyamaz" ifadesi bunu oyunun bir özelliği gibi gösteriyordu; düzeltildi.

## 6. Destek e-postasıyla ne paylaşılabiliyor?

Oyunda iletişim formu, "hata bildir" akışı veya otomatik e-posta gönderen bir kod yok
(kaynakta `mailto:` ya da e-posta gönderimi geçmiyor). Yani oyun kendiliğinden hiçbir
şey göndermiyor; e-posta ancak kullanıcı yazarsa var oluyor.

**Ama yazıldığında bu gerçek bir veri işleme.** Geliştirici kullanıcının e-posta
adresini ve mesaj içeriğini alıp okuyor. Gizlilik metni bunu üstü kapalı geçmiyor:
"geliştirici hiçbir şey almıyor" cümlesi bu istisnayla birlikte veriliyor ve özet,
saklama, silme ve yaş bölümlerinin hepsinden buraya bağlantı var.

**"Kimseyle paylaşılmıyor" denemez.** Posta kutusu bir Gmail hesabı; Google
sağlayıcı olarak mesajı alıyor ve işliyor. İlk taslak aynı paragrafta hem "kimseyle
paylaşılmıyor" hem "Google işliyor" diyordu — kendi içinde çelişkiliydi. Düzeltilmiş
hâl: pazarlama için kullanılmıyor, üçüncü taraflara *kendi amaçları için*
aktarılmıyor, ama Google'ın sağlayıcı olarak işlediği açıkça söyleniyor.

**"E-posta göndermek bunu kabul etmek anlamına gelir" cümlesi kaldırıldı.** Bir rıza
iddiasıydı; kullanıcının yazmaktan başka seçeneği olmayan bir kanal için rıza almış gibi
görünüyordu ve dayanağı yoktu. Gmail'in sağlayıcı olarak işlediği açıklaması olduğu gibi
duruyor — bilgilendirme kalıyor, rıza iddiası gidiyor.

**Saklama süresi ve güvenlik önlemi iddiası yok.** İlk taslaktaki "destek için yararlı
olduğu sürece saklanıyor" ifadesi doğrulanamaz bir politika iddiasıydı. Metin şimdi
şunu söylüyor: mesajlar **otomatik silinmiyor**, tanımlı bir saklama süresi
**uygulanmıyor**, elle silinene kadar duruyorlar, silme talebi aynı adrese yazılarak
iletilebiliyor. 90 gün gibi bir süre ya da otomatik bir temizlik **hiçbir yerde ima
edilmedi** — çünkü böyle bir işleyiş kurulmuş değil. Şifreleme, erişim denetimi gibi
bir güvenlik önlemi de iddia edilmedi.

**Açık iş:** destek yazışmalarının saklanması ve silinmesi için gerçek bir işleyişin
geliştirici tarafından belirlenmesi gerekiyor. Metin bunun henüz netleşmediğini
kullanıcıya da söylüyor ve madde `pre-launch-checklist.md` içinde duruyor.

---

## Danışılan resmî kaynaklar

| Kaynak | Neyi doğruladı |
|---|---|
| Google Play — **User Data** (`support.google.com/googleplay/android-developer/answer/10144311`) | Play'in kullanıcı verisi politikası. Gizlilik metninin aktif, herkese açık, coğrafi kısıtsız bir URL'de olması (PDF olmaz); geliştirici bilgisi ve bir iletişim noktası içermesi; erişim/toplama/kullanım/paylaşımı kapsamlı anlatması; saklama ve silme yollarını açıklaması; Data safety bölümündeki bildirimle tutarlı olması. Sayfanın kendi başlıkları arasında "Data safety section", "Privacy Policy" ve "Account Deletion Requirement" var |
| Google Play Console yardımı — **Prepare your app for review** (`support.google.com/googleplay/android-developer/answer/9859455`) | İnceleme öncesi hazırlık sayfası. Gizlilik metninin hem mağaza sayfasında hem uygulama içinde bağlanması; hangi tür taraflarla paylaşıldığının yazılması; çocuklara yönelik uygulamalarda hiç kişisel/hassas veri işlenmese bile gizlilik metninin zorunlu olması |
| Android Developers — Auto Backup for Apps (`developer.android.com/identity/data/autobackup`) | 4. maddedeki varsayılan yedekleme davranışı, 25 MB kota, Android 9+ uçtan uca şifreleme, kullanıcı kontrolü |
| GitHub Docs — What is GitHub Pages (`docs.github.com/en/pages/...`) | GitHub Pages ziyaretlerinde ziyaretçi IP adresinin güvenlik amacıyla kaydedildiği |

## Metinde bilerek yapılmayanlar

- **AdMob entegre edilmiş gibi yazılmadı.** Hesap açıldı, SDK ve UMP kodda yok;
  metin bunu ayrı bir "Planlanan değişiklikler" başlığında böyle söylüyor.
- **"Hiç veri toplamıyoruz" sözü verilmedi.** İddia iki yönden de daraltıldı:
  zaman olarak (*mevcut sürüm* anlatıldı, gelecekteki reklamlı sürüm için açık bir
  uyarı konuldu) ve kapsam olarak (oyunun *kendiliğinden* geliştiriciye veri
  göndermediği söylendi — destek e-postası ayrı tutuldu). "Oyun hiç kişisel bilgi
  toplamıyor" gibi genel cümleler metinde bırakılmadı.
- **GitHub Pages günlükleri oyundan ayrıldı.** Ayrı başlık, ayrı sorumlu.
- **13+ hukuki uygunluk garantisi gibi sunulmadı.** "Geliştiricinin hedef kitle kararı",
  "resmî mağaza derecelendirmesi değil", "anket doldurulmadı" denildi.
- **Uydurma saklama süresi, güvenlik önlemi ve hukuki iddia yok.** GDPR/KVKK madde
  numarası, veri sorumlusu sıfatı, denetim otoritesi, gün cinsinden saklama süresi ya
  da "verileriniz şifrelenerek saklanır" türü bir önlem iddiası yazılmadı — hiçbiri
  doğrulanabilir değil.
- **Oyuncunun yazdığı adların hayalî olduğu varsayılmadı.** Ajans/menajer alanları
  serbest metin; metin yalnız uygulamanın bunu istemediğini ve ne yazılırsa yazılsın
  cihazda kaldığını söylüyor.
- **Data Safety formu tamamlanmış gibi sunulmadı.** Metin forma atıfta bulunuyor ama
  doldurulduğunu iddia etmiyor.

## Yeniden incelenmesi gereken an

Bu metin **taslak**. AdMob/UMP entegre edildiği anda 3. ve 7. maddeler geçersiz olur.
O noktada `pre-launch-checklist.md` içindeki reklam maddeleri işletilmeli ve iki dildeki
gizlilik sayfası da, "Son güncelleme" tarihi de yenilenmeli.
