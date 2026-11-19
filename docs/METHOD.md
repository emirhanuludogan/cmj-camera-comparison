# Yöntem ve sonuçların yorumu

## Bu kod neyi ölçüyor?

CMJ, sıçramadan önce aşağı yönlü hazırlık hareketi yapılan dikey sıçramadır.
MediaPipe, videodaki vücut noktalarını tahmin eder. Kod, sol ve sağ kalça
noktalarının görünürlük ağırlıklı dikey ortalamasını izler. Kodda bu sinyal
`com_y` olarak adlandırılır; segment kütleleriyle hesaplanan bütün vücut
kütle merkezi değildir.

## İşlem sırası

1. Video kareleri okunur; zaman, kare numarası / FPS ile hesaplanır.
   FPS okunamazsa kod 60 FPS varsayar. Bu varsayım ölçümü etkiler.
2. Kalça ve ayak bileği konumları, görünürlük ağırlıklarıyla piksele dönüştürülür.
   Poz bulunmayan kareler sinyale eklenmez; zaman damgaları korunur.
3. Kalça sinyalinin en küçük Y değeri tepe olarak alınır. Görüntüde Y aşağı doğru artar.
4. İlk 30 ayak bileği örneğinin ortalaması ve sinyal aralığının %15'i kullanılarak
   bir eşik kurulur; tepenin çevresinde uçuş aralığı aranır.
5. Uçuşun iki ucundan ikişer örnek kırpılır; yeterli veri kalmazsa aralığın tamamı kullanılır.
6. `y(t) = a*t² + b*t + c` uydurulur. `g = 9.81 m/s²` ile ölçek
   `pixel_per_meter = 2*a/g` hesaplanır. Pozitif olmayan `a` reddedilir.
7. Kalkış öncesindeki başlangıç konumu ile tepe arasındaki fark ölçeğe bölünür.

Başlangıç konumu aranırken ilk 30 örnek atlanır ve 15 örneklik ortalamalar
kullanılır. Kısa kayıtlarda kalkış öncesi ortalamaya dönülür. Bu pencereler
saniye tabanlı değildir; FPS ve poz kayıpları etkilerini değiştirir.

## İki farklı kalibrasyon

Önceki saha açıklamasında manuel Kinovea ölçümleri için 100 cm toplam ve 40 cm
ara işaretli bir referans çubuğu belirtilmiştir. Kod bu çubuğu algılamaz.
Modelin piksel/metre ölçeği uçuş parabolünden, karşılaştırma referansları ise
`src/main.py` içindeki manuel kayıtlardan gelir. Referans ölçümünün kare seçimi
ve ölçekleme kaydı ayrıca belgelenmelidir.

## Mevcut karşılaştırma

Kaynak: [metrics_output.csv](../results/metrics_output.csv). Altı deneme,
iki katılımcı kodu ve her deneme için ön/yan açı kaydı bulunur.

| Gösterge | Ön açı | Yan açı |
| --- | ---: | ---: |
| Karşılaştırma sayısı | 6 | 6 |
| Ortalama mutlak fark | 1.26 cm | 3.25 cm |
| En küçük mutlak fark | 0.19 cm | 0.66 cm |
| En büyük mutlak fark | 2.35 cm | 5.75 cm |

Ortalama mutlak fark, `mean(abs(model_cm - reference_cm))` olarak hesaplanır.
Bu tablo depodaki CSV'nin özetidir; yeni bir video analizi yapılmış olduğu
anlamına gelmez.

Bu küçük örneklemde ön açıdaki fark daha düşüktür. Bağımsız bir doğrulama
kümesi ve ölçüm belirsizliği kaydı olmadan sonuçlar tüm sporculara, kameralara
ve sıçrama türlerine genellenemez. Kinovea değerleri burada manuel karşılaştırma
referansıdır; kusursuz gerçek değer olarak ele alınmaz.

## Bilinen sınırlılıklar

- Kalça noktası hareketi ile bütün vücut kütle merkezinin hareketi aynı değildir.
- Perspektif, kamera hareketi, örtüşme ve değişken kare hızı ölçümü etkileyebilir.
- Kalça sinyaline parabol uydurmak, ideal serbest uçuş varsayımını yaklaşık olarak kullanır.
- Sabit piksel eşikleri ve örnek sayıları farklı videolarda ayrıca incelenmelidir.
- Boş poz verisi ve bazı kısa sinyaller için ek koruma gerekir.
- Görselleştiricideki örtüşme uyarısı dosya adındaki `yan` sözcüğüne bağlıdır;
  ölçülmüş bir örtüşme dedektörünün çıktısı değildir.
- Tekrar ölçüm güvenilirliği için ICC, SEM, MDC gibi analizler bu repoda henüz yoktur.
