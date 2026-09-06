# _planning — çalışma notları

Mağaza metni taslakları, kaynak inceleme notları ve yayın öncesi kalan işler
burada duruyor. Sitenin gezinmesinde bu klasöre bağlantı yok — ama bu, gizli
olduğu anlamına gelmiyor.

## Uyarı: burası herkese açık

Depo kökünde `.nojekyll` var. Jekyll devre dışı olduğu için alt çizgiyle başlayan
klasörler de **olduğu gibi sunuluyor**: bu dosya, site yayına alındıktan sonra
`https://xahke.github.io/_planning/README.md` adresinden herkes tarafından
okunabilir olacak. Depo zaten herkese açık; `.nojekyll` yalnızca bu dosyaların
ayrıca site üzerinden de servis edilmesi demek.

Buraya asla şunlar konmayacak: keystore parolaları, `keystore.properties`,
AdMob yayıncı kimliği veya uygulama kimlikleri, Play Console ekran görüntüleri,
kişisel iletişim bilgileri (destek adresi zaten kamuya açık), API anahtarları,
yayımlanmamış sürüm tarihleri ya da paylaşılmasında sakınca olan herhangi bir
şey. Buradaki her dosya, bir yabancının okuyacağı varsayılarak yazıldı.

## İçerik

| Dosya | Ne var |
|---|---|
| `privacy-source-review.md` | Gizlilik metninin dayandığı kaynak kod bulguları, dosya referanslarıyla; danışılan resmî belgeler |
| `pre-launch-checklist.md` | Yayın öncesi kalan işler |
| `store-listing/en-short.txt` | Google Play kısa açıklama (EN) |
| `store-listing/en-full.txt` | Google Play tam açıklama (EN) |
| `store-listing/tr-short.txt` | Google Play kısa açıklama (TR) |
| `store-listing/tr-full.txt` | Google Play tam açıklama (TR) |

## Mağaza metni karakter sayıları

Ölçüm, dosyanın sonundaki satır sonu atılarak yapıldı (mağaza alanına o karakter
girmiyor). Sayılan birim Unicode karakter — Türkçe harfler tek karakter sayılıyor.

| Dosya | Karakter | Sınır | Durum |
|---|---:|---:|---|
| `en-short.txt` | 74 | 80 | uygun |
| `tr-short.txt` | 74 | 80 | uygun |
| `en-full.txt` | 3509 | 4000 | uygun |
| `tr-full.txt` | 3411 | 4000 | uygun |

Yeniden ölçmek için depo kökünde:

```sh
python - <<'PY'
import io
lim = {"short": 80, "full": 4000}
for f in ["en-short.txt","tr-short.txt","en-full.txt","tr-full.txt"]:
    s = io.open("_planning/store-listing/" + f, encoding="utf-8").read().rstrip("\n")
    kind = "short" if "short" in f else "full"
    print("%-14s %5d / %4d" % (f, len(s), lim[kind]))
PY
```

## Metinlerin sınırı

Mağaza metinleri yalnızca **mevcut sürümde çalışan** özellikleri anlatıyor.
Reklam, uygulama içi satın alma, bulut kaydı, çok oyunculu mod, skor tablosu ve
başarımlar hakkında hiçbir vaat yok — çünkü hiçbiri kodda yok. Değerlendirme,
indirme sayısı, ödül ya da basın alıntısı da yok; hiçbiri gerçek değil.

Kısa açıklamalar 80 karakterin altında tutuldu ama sınıra dayanmıyor: Play
arayüzünde uzun kısa açıklamalar bazı yerleşimlerde kırpılıyor.
