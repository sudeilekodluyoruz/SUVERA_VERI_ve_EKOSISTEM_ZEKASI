# SUVERA — Data & Ecosystem Intelligence

> **The world of AI, through data.**

SUVERA, 2016–2025 dönemindeki küresel özel yapay zekâ yatırımını ülke bazında inceleyen etkileşimli bir **Streamlit** web uygulamasıdır. Yatırım eğilimlerini, Türkiye'nin dünya içindeki konumunu ve makine öğrenmesiyle üretilen yatırım senaryolarını tek arayüzde sunar.

![Genel Bakış]

## İçindekiler

- [Öne çıkanlar](#öne-çıkanlar)
- [Sayfalar](#sayfalar)
- [Veri kaynakları ve veri modeli](#veri-kaynakları-ve-veri-modeli)
- [Tahmin modeli](#tahmin-modeli)
- [Kurulum](#kurulum)
- [Proje yapısı](#proje-yapısı)
- [Sınırlamalar](#sınırlamalar)
- [Lisans](#lisans)

## Öne çıkanlar

| | |
|---|---|
| **1,73 trilyon $** | 107 ülkede kayıtlı toplam özel yapay zekâ yatırımı |
| **67** | Pozitif büyük ölçekli yapay zekâ model kaydı (20 ülke) |
| **%31,3** | Nihai testte (2023–2025) medyan baseline'a göre MAE iyileşmesi |
| **34. / 107** | Türkiye'nin yatırım sıralaması (toplam 1,3 milyar $, kişi başı 1,53 $) |

## Sayfalar

### Genel Bakış
Toplam yatırım, ülke sayısı ve dönem özet kartları; yıllık yatırım eğilimi ve en çok yatırım alan ülkeler (ABD payı %62,0, ilk 10 ülkenin payı %89,6).

### Türkiye Profili
Türkiye'nin toplam ve kişi başı yapay zekâ yatırımı, dünya değeriyle karşılaştırma ve 2023 kalkınma göstergelerinin dünya medyanıyla kıyaslanması. Kaynakta olmayan veri sıfır sayılmaz, **"veri yok"** olarak gösterilir.

![Türkiye Profili](docs/screenshots/02-turkiye-profili.png)

### Yatırım Tahmini
Ülke ve tahmin yılı seçilir; önceki yılın kalkınma göstergeleri forma otomatik gelir ve düzenlenebilir. Eksik göstergeler eğitim verisinin medyanıyla doldurulur ve kullanıcıya açıkça bildirilir.

![Yatırım Tahmini](docs/screenshots/03-yatirim-tahmini.png)

![Senaryo tahmini](docs/screenshots/04-tahmin-sonucu.png)

### Model Performansı
Modellerin zamansal doğrulama ve nihai test sonuçları (MAE, RMSE, R²), gerçek–tahmin dağılım grafiği ve en yüksek mutlak hataya sahip 15 kayıt.

![Model Performansı](docs/screenshots/05-model-performansi.png)

![Gerçek ve tahmin edilen yatırım tutarları](docs/screenshots/06-gercek-tahmin-grafigi.png)

### Veri Merkezi
Analizin dayandığı beş tabloyu listeler; yıl ve ülke koduna göre filtreleme ve CSV indirme sağlar.

![Veri Merkezi](docs/screenshots/08-veri-merkezi.png)

## Veri kaynakları ve veri modeli

| Kaynak | İçerik |
|---|---|
| [CSET](https://cset.georgetown.edu/) | Özel yapay zekâ yatırımı (sabit 2021 $) — tahmin hedefi |
| [Epoch AI](https://epoch.ai/) | Kümülatif büyük ölçekli yapay zekâ sistemi sayısı (20 ülke, 2019–2025) |
| [Dünya Bankası WDI](https://data.worldbank.org/) | Nüfus, Ar-Ge harcaması, araştırmacı sayısı, internet kullanımı, geniş bant, kişi başı GSYH, yükseköğretim kaydı |

Veriler **yıldız şema** ile düzenlenmiştir (`3_Veri/` klasöründeki CSV dosyaları):

| Tablo | Tür | Satır | Açıklama |
|---|---|---:|---|
| `FactYatirim` | Olgu | 697 | Pozitif yatırım kaydı bulunan ülke-yıllar (107 ülke) |
| `FactKalkinma` | Olgu | 2.104 | Kalkınma göstergeleri ve nüfus (217 ülke) |
| `FactModel` | Olgu | 140 | Büyük ölçekli yapay zekâ model kayıtları (20 ülke) |
| `DimUlke` | Boyut | 217 | Ülke bilgileri (Türkçe ad, kıta, gelir grubu) |
| `DimYil` | Boyut | 10 | Yıl boyutu (2016–2025) |

> Boş değer **"veri yok"** demektir. Yatırım veya model tablosunda satırın bulunmaması sıfır yatırım ya da "büyük model yok" anlamına gelmez. Ayrıntılar için [`3_Veri/Veri_Sozlugu.md`](3_Veri/Veri_Sozlugu.md).

## Tahmin modeli

**Problem:** Bir ülkenin hedef yıldaki pozitif özel yapay zekâ yatırım tutarını (`YatirimUSD`), bir önceki yılın kalkınma ve büyük model göstergeleriyle tahmin etmek.

- **Aday modeller:** Extra Trees, Random Forest ve Ridge; her biri "Temel" ve "Büyük Model Ekli" özellik setiyle, ayrıca medyan baseline
- **Zamansal ayrım:** 2017–2020 eğitim · 2021–2022 doğrulama (model seçimi) · 2023–2025 nihai test (234 örnek). Seçilen model eğitim + doğrulama verisiyle yeniden eğitilir.
- **Ön işleme:** Hedef `log(1 + USD)` dönüşümüyle eğitilir, metrikler USD ölçeğinde raporlanır; eksik sayısal değerler medyanla doldurulur, ülke kodu One-Hot kodlanır.

| Model (nihai test, 2023–2025) | MAE ($) | RMSE ($) | R² |
|---|---:|---:|---:|
| Extra Trees (Büyük Model Ekli) | 2.321.282.570 | 19.613.672.599 | 0,249 |
| Medyan baseline | 3.379.525.030 | 22.870.151.218 | -0,022 |

Ayrıntılı notlar: [`5_Web_Uygulamasi/MODEL_NOTLARI.md`](5_Web_Uygulamasi/MODEL_NOTLARI.md)

## Kurulum

**Gereksinimler:** Python 3.10+ (streamlit, pandas, numpy, scikit-learn, plotly)

```bash
git clone https://github.com/<kullanici-adi>/<repo-adi>.git
cd <repo-adi>

python -m venv .venv
```

Windows (PowerShell):

```powershell
.venv\Scripts\Activate.ps1
```

macOS / Linux:

```bash
source .venv/bin/activate
```

Ardından bağımlılıkları yükleyip uygulamayı başlatın:

```bash
pip install -r 5_Web_Uygulamasi/requirements.txt
python -m streamlit run 5_Web_Uygulamasi/app.py
```

Uygulama varsayılan olarak <http://localhost:8501> adresinde açılır.

### Verileri yeniden üretme (isteğe bağlı)

```bash
cd 4_Kod
pip install pandas scipy numpy
python 01_hazirla_yildiz_sema.py   # ham_veri -> ../3_Veri
python 02_temel_bulgular.py        # temel sayıları yazdırır
```

## Proje yapısı

```text
.
├── 1_Rapor/              # Bitirme projesi raporu (PDF)
├── 2_Power_BI/           # Power BI projesi (.pbip) ve kurulum rehberi
├── 3_Veri/               # Yıldız şema CSV tabloları ve veri sözlüğü
├── 4_Kod/                # Veri hazırlama betikleri ve ham kaynak dosyaları
├── 5_Web_Uygulamasi/     # Streamlit uygulaması
│   ├── app.py
│   ├── requirements.txt
│   ├── MODEL_NOTLARI.md
│   └── src/
│       ├── data.py       # veri yükleme
│       └── model.py      # model eğitimi ve değerlendirme
└── docs/screenshots/     # README ekran görüntüleri
```

## Sınırlamalar

- Model yatırımın **gerçekleşip gerçekleşmeyeceğini** değil, pozitif kayıt bulunan ülke-yıllarda **tutarını** tahmin eder.
- R² düşüktür (0,249); çıktılar kesin finansal öngörü değil, **senaryo tabanlı analitik tahmindir**.
- Yatırım tutarları çok çarpık dağılır; az sayıdaki çok büyük yatırım (ör. ABD 2025) hata metriklerini belirgin biçimde etkiler.
- Epoch AI verisi yalnızca 20 ülkeyi kapsar; Türkiye bu tabloda yer almaz ve "veri yok" olarak ele alınır.
- Gelecek yıl için gösterge verisi yoksa en güncel mevcut yıl kullanılır ve bu kullanıcıya bildirilir.

## Lisans

Lisans bilgisini buraya ekleyin (ör. MIT). Veri kaynaklarının kendi kullanım koşulları geçerlidir.
