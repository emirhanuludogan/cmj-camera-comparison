# CMJ: video tabanlı sıçrama ölçümü karşılaştırması

Python ve MediaPipe ile dikey sıçrama videosundan kalça hareketini izleyen,
uçuş parabolünden ölçek tahmin eden ve sonuçları manuel Kinovea ölçümleriyle
karşılaştıran staj projesi.

**Durum:** Geliştirme ve yöntem değerlendirmesi. Mevcut sonuçlar altı denemelik
bir ön karşılaştırmadır; kapsamlı güvenilirlik doğrulaması henüz yapılmadı.

## Projeyi ilk kez inceliyorsanız

- **Amaç:** Aynı sıçramanın ön ve yan kamera kayıtlarında hesaplanan yüksekliği karşılaştırmak.
- **Girdi:** Ön/yan videolar ve her açıya ait manuel referans yüksekliği.
- **Çıktı:** Deneme bazında model yüksekliği, referans farkı ve açılar arası fark içeren CSV.
- **Yöntem:** MediaPipe Pose Lite → görünürlük ağırlıklı kalça sinyali → uçuş aralığı →
  parabolik ölçek → başlangıçtan tepeye yer değiştirme.
- **Sınır:** Kodda “CoM” adı kullanılsa da izlenen sinyal iki kalça noktasından türetilir;
  bütün vücudun kütle merkezi doğrudan hesaplanmaz.

## Mevcut sonuç

[Depodaki CSV](results/metrics_output.csv) üzerinden hesaplanan ortalama mutlak
fark ön açı için **1.26 cm**, yan açı için **3.25 cm**'dir; her açı için altı
karşılaştırma bulunur. Bu değerler yeni bir analiz koşusundan alınmamıştır.
Yöntem, varsayımlar ve sonuçların sınırları [yöntem notunda](docs/METHOD.md) açıklanır.

## Kurulum

Mevcut yerel ortam Python 3.10.11 kullanır. Temiz ortamda bağımlılık kurulumu
henüz doğrulanmadı; `requirements.txt` mevcut ortamın sürüm listesidir.

Windows PowerShell, Python 3.10:

```powershell
git clone https://github.com/emirhanuludogan/cmj-reliability-test.git
cd cmj-reliability-test
py -3.10 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

## Kullanım

1. Videoları `data/raw_videos/` klasörüne yerleştirin. Ham kayıtlar bu repoyla dağıtılmaz.
2. `src/main.py` içindeki `test_sessions` sözlüğünde dosya adlarını ve referansları
   düzenleyin. `front` ön, `side` yan kamera kaydıdır.
   `kinovea_ref_front_m` ve `kinovea_ref_side_m` **metre** cinsindedir;
   örneğin 30 cm için `0.30` girilir. Çıktı santimetredir.
3. Repo kökünden çalıştırın:

```powershell
.\.venv\Scripts\python.exe src/main.py
```

Başarılı denemeler `results/metrics_output.csv` dosyasına yazılır.
Yeni başarılı koşu mevcut CSV'nin üzerine yazar; karşılaştırma kaydınızı önce kopyalayın.
Dosyası eksik olan veya analizden hata dönen denemeler atlanır.

İskelet ve kalça takibini video üzerinde görmek için:

```powershell
.\.venv\Scripts\python.exe src/visualizer.py
```

Bu komut pencere açar ve `results/SUNUM_*.mp4` dosyaları üretir.
`q` tuşu mevcut videoyu atlar. Yan açı için gösterilen örtüşme uyarısı dosya
adına dayanır; otomatik örtüşme ölçümü değildir.

## Dosya rehberi

| Yol | Görevi |
| --- | --- |
| `src/main.py` | Deneme tanımları, toplu analiz ve CSV çıktısı |
| `src/pose_extractor.py` | Karelerden kalça ve ayak bileği sinyallerini çıkarma |
| `src/jump_analyzer.py` | Uçuş aralığı, parabolik ölçek ve yükseklik hesabı |
| `src/visualizer.py` | Takibin videoya çizilmesi |
| `data/raw_videos/` | Yerel ham video klasörü |
| `results/metrics_output.csv` | Kayıtlı ön karşılaştırma sonuçları |
| `docs/METHOD.md` | Yöntem ve bilinen sınırlılıklar |
| `docs/ROADMAP.md` | Tamamlanan ve sıradaki geliştirmeler |

## İlgili çalışmalar

- [Biyomekanik Sıçrama Analizi](https://github.com/emirhanuludogan/Biyomekanik-Sicrama-Analizi):
  farklı kişi dedektörleri ve sıçrama metrikleri üzerine daha geniş deney projesi.
- [EOD-reports](https://github.com/emirhanuludogan/EOD-reports):
  staj araştırma ve toplantı arşivi; erişim izni gerektirir.
