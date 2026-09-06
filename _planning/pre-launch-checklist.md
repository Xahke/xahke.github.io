# Yayın öncesi kalan işler

Durum: **hiçbiri yapılmadı.** Bu bir yapılacaklar listesi, bir ilerleme raporu değil.
Aşağıdaki hiçbir madde tamamlanmış sayılmamalı.

## Site

- [ ] `feat/developer-website` dalını incele, `main`'e al ve depoya gönder.
- [ ] GitHub Pages'i depo ayarlarından aç (Settings → Pages, kaynak: `main`, kök dizin).
- [ ] Yayına aldıktan sonra canlı adreslerin hepsini bir kez aç:
      `/`, `/tr/`, `/pro-football-agent/`, `/tr/pro-football-agent/`,
      `/privacy/pro-football-agent/en.html`, `/privacy/pro-football-agent/tr.html`.
- [ ] Ekran görüntüleri hazır olduğunda oyun sayfalarındaki "henüz ekran görüntüsü yok"
      notunu gerçek görsellerle değiştir.
- [ ] Oyun yayımlandığında "Yakında / Coming soon" rozetlerini ve
      "indirilecek bir şey yok" notlarını gerçek Google Play bağlantısıyla değiştir.
      Bağlantı çalışmadan önce düğme konmayacak.

## app-ads.txt

- [ ] Gerçek AdMob yayıncı kimliği (`pub-...`) elde olduğunda site köküne
      `/app-ads.txt` ekle. AdMob'un verdiği satır aynen kopyalanacak, elle yazılmayacak.
- [ ] Dosya `https://xahke.github.io/app-ads.txt` adresinden düz metin olarak
      erişilebilir olmalı.
- [ ] Play Console'daki geliştirici web sitesi alanı `https://xahke.github.io`
      olarak ayarlanmalı; AdMob taraması dosyayı bu alandan buluyor.

Şu an bu dosya **oluşturulmadı**. Sahte bir `pub-...` değeri koymak, doğrulamayı
sessizce başarısız kılar ve düzeltilmesi gecikir.

## Reklam / AdMob entegrasyonu

- [ ] AdMob SDK ve UMP (rıza yönetimi) oyuna entegre et.
- [ ] Entegrasyondan **sonra** iki dildeki gizlilik metnini yeniden yaz:
      toplanan tanımlayıcılar, reklam ortakları, rıza akışı, veri paylaşımı.
      `privacy-source-review.md` içindeki 3. ve 7. maddeler o noktada geçersiz olur.
- [ ] Oyun sayfalarındaki "mevcut sürümde reklam SDK'sı yok" notunu güncelle.
- [ ] Gizlilik metninin "Son güncelleme" tarihini değiştir.

## Gizlilik metni — netleştirilecek noktalar

- [ ] **Destek yazışmalarının saklanması ve silinmesi.** Şu anda tanımlı bir işleyiş
      yok: mesajlar otomatik silinmiyor, bir saklama süresi uygulanmıyor ve silme
      yalnız talep üzerine elle yapılıyor. Gizlilik metni bunu olduğu gibi söylüyor ve
      "yayın öncesi belirginleştirilecek" diyor. Karar verilmesi gerekenler:
      yazışma ne kadar tutulacak, silme talebi nasıl karşılanacak, çözülen konular
      düzenli olarak temizlenecek mi. Karar verildikten sonra iki dildeki
      "Destek e-postası yazarsan" bölümü kesin ifadeyle güncellenecek.
      **90 gün gibi bir süre ya da otomatik temizlik, gerçekten kurulmadan yazılmayacak.**
- [ ] **Android yedeklemesini bir cihazda doğrula.** Manifest yedeklemeye izin veriyor
      (`allowBackup="true"`, kural dosyası yok) ama yedek alınıp geri yüklenerek
      sınanmadı. Yedeğe gerçekten kayıtların girip girmediği görülmeli.
      Alternatif bir karar: yedeklemeyi kapatmak ya da açık `dataExtractionRules`
      yazmak — o zaman metin çok daha kesin olabilir.
- [ ] Karar sonrası her iki gizlilik sayfasının "Son güncelleme" tarihini yenile.

## Google Play Console

- [ ] Data safety (Veri güvenliği) formunu doldur. **Henüz doldurulmadı.**
      Doldurulduğunda gizlilik metniyle birebir tutarlı olması gerekiyor;
      tutarsızlık tek başına ret sebebi.
- [ ] İçerik derecelendirme anketini doldur. Sitedeki "13 yaş ve üzeri" ifadesi
      geliştiricinin hedef kitle kararı; anketin sonucu farklı çıkabilir.
      Çıkarsa site metinleri buna göre düzeltilecek.
- [ ] Hedef kitle ve içerik (Target audience) bölümünü doldur.
- [ ] Gizlilik metni URL'sini Play Console'a gir:
      `https://xahke.github.io/privacy/pro-football-agent/en.html`
      (Türkçe sürüm mağaza yerelleştirmesine bağlanacak).
- [ ] Gizlilik metnine **uygulama içinden de** bir bağlantı ekle — Play'in User Data
      politikası hem mağaza sayfasında hem uygulama içinde bağlantı istiyor.
      Oyunda şu an böyle bir bağlantı yok; Ayarlar ekranına eklenmesi gerekiyor.
- [ ] Mağaza metinlerini `store-listing/` klasöründen kopyala (karakter sayıları
      `README.md` içinde ölçülü).
- [ ] Mağaza görselleri: uygulama ikonu (512×512, hazır), öne çıkan görsel
      (1024×500, **yok**), telefon ekran görüntüleri (en az 2, **yok**).

## Sürüm

- [ ] `versionCode`/`versionName` gözden geçir (`android/app/build.gradle` şu an 1 / 1.0.0).
- [ ] İmzalı AAB üret ve yükle. İmzalama yapılandırması depoda değil; anahtar ve
      parolalar depo dışında kalacak.

## Karar bekleyen içerik noktaları

- Oyunun mağazadaki adı ile uygulama içindeki adı ayrışıyor: Android uygulama adı
  **Pro Football Agent** (`android/.../strings.xml`), web/PWA manifest'i ve oyun içi
  başlık ise hâlâ **Menajer** (`manifest.json`, `index.html`). Site her yerde
  "Pro Football Agent" diyor. Mağazaya çıkmadan önce hangisinin görüneceğine karar
  verilmeli.
- Site şu an yalnız `xahke.github.io` üzerinde. Özel alan adı düşünülüyorsa,
  gizlilik metni URL'si Play Console'a girilmeden önce karara bağlanmalı —
  sonradan değiştirmek formun yeniden incelenmesini gerektiriyor.
