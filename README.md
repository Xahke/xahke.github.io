# xahke.github.io

Xahke games — official website and support.

Muhammed Berkin Karaca'nın bağımsız oyun geliştirici sitesi. Düz HTML ve CSS;
framework, bundler, npm bağımlılığı ve arka uç yok. Harici font, analitik,
reklam, takip pikseli, çerez banner'ı ve iletişim formu da yok — sistem fontları
ve depodaki yerel dosyalar yeterli.

Planlanan adres: <https://xahke.github.io>

## Yerel önizleme

Sayfalar kök göreli yollar kullanıyor (`/assets/css/site.css`), bu yüzden
`index.html` dosyasını doğrudan çift tıklayarak açmak yeterli değil — stiller ve
görseller yüklenmez. Depo kökünde bir HTTP sunucusu çalıştır:

```sh
# Python 3 (kurulum gerekmez)
python -m http.server 8000
```

```sh
# ya da Node
npx --yes serve . -l 8000
```

Sonra tarayıcıda:

| Sayfa | Adres |
|---|---|
| Ana sayfa (EN) | <http://localhost:8000/> |
| Ana sayfa (TR) | <http://localhost:8000/tr/> |
| Pro Football Agent (EN) | <http://localhost:8000/pro-football-agent/> |
| Pro Football Agent (TR) | <http://localhost:8000/tr/pro-football-agent/> |
| Gizlilik metni (EN) | <http://localhost:8000/privacy/pro-football-agent/en.html> |
| Gizlilik metni (TR) | <http://localhost:8000/privacy/pro-football-agent/tr.html> |

Durdurmak için `Ctrl+C`.

## Yapı

```
index.html                          İngilizce ana sayfa
tr/index.html                       Türkçe ana sayfa
pro-football-agent/index.html       Oyun tanıtımı (EN)
tr/pro-football-agent/index.html    Oyun tanıtımı (TR)
privacy/pro-football-agent/en.html  Gizlilik metni (EN)
privacy/pro-football-agent/tr.html  Gizlilik metni (TR)
assets/css/site.css                 Sitenin tek stil dosyası
assets/img/                         İkonlar (oyun deposundan üretildi)
_planning/                          Çalışma notları — sitede bağlantısı yok ama
                                    yayımlanıyor; aşağıya bak
.nojekyll                           Jekyll devre dışı
```

### Görseller

`assets/img/` içindeki üç PNG, `FootballAgent_Mobile` deposundaki
`store-assets/google-play/icon-512.png` dosyasından ölçeklendirildi. Kaynak
dosyaya dokunulmadı. 6 MB'lik `store-assets/source/pro-football-agent-icon-master.png`
siteye hiç kopyalanmadı — sayfaya yüklenmesi gereksiz olurdu.

Yeniden üretmek için depo kökünde (Pillow gerekiyor):

```sh
python - <<'PY'
from PIL import Image
src = r"..\FootballAgent_Mobile\store-assets\google-play\icon-512.png"
im = Image.open(src).convert("RGB")
for size, name in [(256, "pro-football-agent-icon-256.png"),
                   (96,  "pro-football-agent-icon-96.png"),
                   (32,  "favicon-32.png")]:
    im.resize((size, size), Image.LANCZOS).save("assets/img/" + name, optimize=True)
PY
```

Oyunun ekran görüntüsü henüz yok; sahte oyun ekranı üretilmedi.

## Yayınlama

Site GitHub Pages ile yayımlanacak; ayrı bir derleme adımı yok, depodaki dosyalar
olduğu gibi sunuluyor.

1. Değişiklikleri `main` dalına al ve `origin`'e gönder.
2. GitHub'da **Settings → Pages** bölümünde kaynağı `main` dalı ve `/` (kök)
   olarak seç.
3. Birkaç dakika içinde site <https://xahke.github.io> adresinde yayına girer.

`.nojekyll` dosyası Jekyll'i devre dışı bırakıyor: dosyalar hiçbir işlemden
geçmeden sunuluyor ve alt çizgiyle başlayan klasörler de yayımlanıyor.

## `_planning/` herkese açık

`.nojekyll` yüzünden `_planning/` de yayımlanıyor:
`https://xahke.github.io/_planning/README.md` gibi adresler yayına girdikten sonra
okunabilir olacak. Bu klasörde mağaza metni taslakları, kaynak inceleme notu ve
yayın öncesi kalan işler duruyor. **Buraya gizli hiçbir şey konmayacak** — keystore
parolası, AdMob yayıncı kimliği, Play Console ekran görüntüsü, API anahtarı yok.

## `app-ads.txt` henüz yok

Bu depoda `app-ads.txt` **bilerek oluşturulmadı**. Gerçek AdMob yayıncı kimliği
(`pub-...`) henüz paylaşılmadı ve sahte bir değerle dosya koymak doğrulamayı
sessizce başarısız kılardı.

Gerçek satır elde edildiğinde dosya site köküne `/app-ads.txt` olarak eklenecek —
yani `https://xahke.github.io/app-ads.txt` adresinden düz metin olarak
erişilebilecek. Satır AdMob arayüzünden aynen kopyalanacak, elle yazılmayacak.
Bu eksiklik sitenin geri kalanını etkilemiyor.

## Gizlilik metinleri taslaktır

`privacy/pro-football-agent/` altındaki iki sayfa, Pro Football Agent'ın
**henüz yayımlanmamış mevcut sürümünün** kaynak kodu incelenerek yazıldı;
varsayımla doldurulmadı. Hangi bulgunun hangi dosyadan geldiği ve hangi resmî
belgelere bakıldığı `_planning/privacy-source-review.md` içinde yazıyor.

Bu metinler yayın öncesinde yeniden incelenecek. Özellikle **AdMob ve UMP oyuna
entegre edildikten sonra iki dilde de yeniden yazılmaları gerekiyor**: reklam
SDK'ları tipik olarak cihaz ve reklam tanımlayıcıları topluyor ve mevcut metin
bunu anlatmıyor. Kalan işlerin tamamı `_planning/pre-launch-checklist.md` içinde.

## Destek

<xahkeee@gmail.com>
