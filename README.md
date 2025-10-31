# Wind_Turbine_Scada
# 💨 Rüzgar Türbini Kısa Vadeli Enerji Üretim Tahmini Projesi

## 1.  Problem Tanımı

Rüzgar enerjisi, rüzgar hızı ve yönündeki sürekli değişimler nedeniyle öngörülmesi zor bir enerji kaynağıdır.  
Enerji şebekelerinin istikrarlı yönetimi, üretim planlaması ve ticari optimizasyon için **kısa vadeli (örneğin, sonraki 7 gün)** enerji üretim tahminlerinin yüksek doğrulukta yapılması hayati önem taşır.

Bu projenin temel zorluğu, gelecekteki enerji üretimini tahmin ederken bilinmeyen iki faktör zincirini kırmaktır:

1. Gelecekteki rüzgar hızının tahmin edilmesi  
2. Tahmin edilen rüzgar hızına ve diğer çevresel koşullara dayanarak güç üretiminin tahmin edilmesi

---

## 2.  Proje Amacı ve Yaklaşım

Bu proje, bir rüzgar türbininin net güç üretimini (**LV ActivePower**) iki aşamalı entegre bir **zaman serisi + makine öğrenimi** modeli ile tahmin etmeyi amaçlar.

###  Entegre Modelleme Akışı

| Aşama | Hedef | Kullanılan Metot | Çıktı |
|-------|--------|------------------|--------|
| **Aşama 1: Öncül Tahmin** | Gelecekteki Rüzgar Hızını Tahmin Etmek | SARIMA (Mevsimsel Otoregresif Bütünleşik Hareketli Ortalama) | Tahmini Rüzgar Hızı Serisi |
| **Aşama 2: Nihai Tahmin** | Güç Çıktısını Tahmin Etmek | Random Forest Regressor | Tahmini Güç Üretimi (kW) |

---

## 3.  Veri Seti (`T1.csv`)

Projede kullanılan veri seti, tek bir endüstriyel rüzgar türbininden **bir yıllık (2018)** süre boyunca toplanan **SCADA** verilerini içerir.  
Veriler başlangıçta **10 dakikalık aralıklarla** toplanmıştır.

| Sütun Adı | Açıklama | Birim | Rolü |
|------------|-----------|--------|------|
| **Date/Time** | Verinin kaydedildiği zaman damgası | Zaman | İndeks |
| **LV ActivePower (kW)** | Türbinin şebekeye sağladığı gerçek güç | kW | Hedef Değişken (Y) |
| **Wind Speed (m/s)** | Ölçülen rüzgar hızı | m/s | Ana Özellik, SARIMA Hedefi |
| **Theoretical_Power_Curve (KWh)** | Rüzgar hızına göre beklenen teorik güç | KWh | Özellik |
| **Wind Direction (°)** | Rüzgarın estiği yön | Derece | Özellik |

###  Ön İşleme Adımları

- **Resampling (Yeniden Örnekleme):**  
  10 dakikalık veriler, analiz kolaylığı ve modelleme için **saatlik ortalamalara** düşürülmüştür.

- **Kayıp Veri Yönetimi:**  
  Bütünlüğü korumak amacıyla **NaN değerler** (eksik sensör okumaları veya türbinin durduğu anlar) `0` ile doldurulmuştur.

- **Özellik Mühendisliği:**  
  Model performansını artırmak için aşağıdaki döngüsel ve zamansal özellikler eklenmiştir:
  - `hour`, `dayofweek`, `month`
  - `Wind_Direction_sin`, `Wind_Direction_cos` (rüzgar yönünün sinüs ve kosinüs bileşenleri)

---

## 4.  Modelleme Detayları

### 4.1. Aşama 1: Rüzgar Hızı Tahmini (SARIMA)

**Model Tipi:** `SARIMA(1, 0, 1) × (1, 1, 1, 24)`  
- Mevsimsel Olmayan: `(p=1, d=0, q=1)` → Seri durağan olduğu için `d=0`  
- Mevsimsel: `(P=1, D=1, Q=1, S=24)` → Günlük mevsimselliği (24 saat) yakalamak için

**Eğitim Seti:** Verinin ilk ~8200 saati  
**Test Seti:** Verinin son 7 günü (168 saat)

---

### 4.2. Aşama 2: Güç Üretimi Tahmini (Random Forest)

Bu modelde, tahminin pratik değerini göstermek için iki farklı senaryo karşılaştırılmıştır:

| Senaryo | Rüzgar Hızı Kaynağı | Amaç |
|----------|----------------------|-------|
| **A (Benchmark)** | Gerçek `Wind_Speed_ms` (Test Seti) | Modelin ulaşabileceği **en iyi teorik performansı** ölçmek |
| **B (Entegre)** | SARIMA Tahmini `Wind_Speed_ms` (Aşama 1'den) | Gerçek hayatta, rüzgar hızının bilinmediği koşullarda elde edilen **tahmini performansı** ölçmek |

---

## 5.  Sonuçlar ve Performans

| Metrik | Model | Değer | Yorum |
|--------|--------|--------|--------|
| **SARIMA RMSE** | Rüzgar Hızı Tahmini | `[KOD ÇIKTISI]` m/s | Bu değer, güç tahmininin temel hatasını belirler. |
| **RF RMSE (Benchmark)** | Gerçek Rüzgar Hızı | `[KOD ÇIKTISI]` kW | Modelin ideal koşullardaki performansı. |
| **RF RMSE (Entegre)** | SARIMA Tahmini Rüzgar Hızı | `[KOD ÇIKTISI]` kW | Nihai senaryoda, gelecekteki güç üretimi için beklenen hata payı. |

---

## 6.  Kullanılan Teknolojiler

- **Python 3.11+**
- `pandas`, `numpy`
- `matplotlib`, `statsmodels`
- `scikit-learn`
- `SARIMAX` (Zaman Serisi Modelleme)
- `RandomForestRegressor` (Makine Öğrenimi)

---

## 7.  Çalıştırma Adımları

```bash
# 1. Gerekli kütüphaneleri yükle
pip install pandas numpy matplotlib statsmodels scikit-learn

# 2. Dosyayı çalıştır
python wind_turbine_forecast.py

# 3. Tahmin sonuçlarını incele
# - Rüzgar hızı tahmin grafiği
# - Güç üretim tahmin grafiği
# - RMSE değerleri
