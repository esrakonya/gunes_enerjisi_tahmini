# Günlük Güneş Enerjisi Tahmini

TNC GROUP - Yapay Zeka Staj Programı kapsamında hazırladığım **"Uçtan Uca Yapay Zeka Projesi: Veriden Tahmine"** çalışması.

Sıcaklık, nem, bulutluluk ve ay bilgisine bakarak bir günün güneş radyasyonunu tahmin eden basit bir regresyon modeli ve bu modeli kullanan küçük bir tahmin aracı.

[![Colab'de aç](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/esrakonya/gunes_enerjisi_tahmini/blob/main/gunes_enerjisi_tahmini.ipynb)

## Problem

Günlük hayattan bir şeyi yapay zekaya tahmin ettirmek istedim. Güneş paneli düşünen kişiler için günlük güneş enerjisi tahmini faydalı olabilir.

- **Problem cümlesi:** Bir günün sıcaklık, nem, bulutluluk ve ay bilgisinden o günün güneş radyasyonunu (kWh/m²/gün) tahmin et.
- **Hedef değişken:** `ALLSKY_SFC_SW_DWN` (günlük güneş radyasyonu)
- **Problem türü:** Regresyon

Model "yarın ne kadar güneş olacak?" sorusunu değil, "bu hava koşullarında ne kadar güneş düşer?" sorusunu cevaplar.

## Veri

- **Kaynak:** [NASA POWER](https://power.larc.nasa.gov/)
- **Konum:** Marmaris, Muğla (36.85 enlem, 28.27 boylam)
- **Dönem:** 1 Ocak – 31 Aralık 2025, günlük (365 satır)
- **Kaynaklar:** Radyasyon ve bulutluluk CERES SYN1deg, sıcaklık, nem ve rüzgar MERRA-2

Veri tek bir istasyonun ölçümü değil, uydu ve atmosfer modellerinden gelen bölgesel bir tahmindir.

| Sütun | Anlamı | Rol |
|---|---|---|
| `T2M` | 2 m yükseklikte ortalama sıcaklık (°C) | Özellik |
| `RH2M` | Bağıl nem (%) | Özellik |
| `CLOUD_AMT` | Bulut örtüsü (%) | Özellik |
| `MO` (`month`) | Ay (1-12) | Özellik |
| `WS2M` | Rüzgar hızı (m/s) | Kullanılmadı (hedefle ilişkisi çok zayıf) |
| `ALLSKY_SFC_SW_DWN` | Günlük güneş radyasyonu (kWh/m²/gün) | Hedef |

Kontrollerde eksik değer ve kopya satır çıkmadı.

## Yöntem

1. NASA POWER'dan veri indirme ve `gunes_verisi.csv` olarak kaydetme
2. Veri kontrolü (`isnull`, `duplicated`, `dropna`, `describe`)
3. Keşifçi veri analizi (`groupby`, `value_counts`, 2 grafik)
4. Veriyi %80 eğitim (292 satır) ve %20 test (73 satır) olarak bölme
5. `LinearRegression` modelini eğitme
6. Tahmin, değerlendirme ve `predict_solar()` mini uygulaması

## Bulgular

- Aylık ortalama radyasyon Haziran'da en yüksek (8.15), Aralık'ta en düşük (2.35 kWh/m²/gün). Haziran, Aralık'ın yaklaşık 3,5 katı.
- Hedefle en güçlü ilişkiler: nem (-0.74), bulutluluk (-0.71), sıcaklık (+0.69).
- Yazın günlük değerler birbirine yakın (Haziran'da 7.25 – 8.71), ilkbahar ve sonbaharda aynı ay içinde büyük farklar var (Mart'ta 1.17 – 6.47).

## Sonuçlar

| Ölçüt | Değer |
|---|---|
| Eğitim R² | 0.77 |
| Test R² | 0.71 |
| Ortalama mutlak hata (MAE) | 0.94 kWh/m²/gün (ortalama hedefin yaklaşık %18'i) |
| Ay özelliği olmadan test R² | 0.63 |

Eğitim ve test skorları birbirine yakın olduğu için ciddi bir aşırı öğrenme görünmüyor. Model genel eğilimi iyi yakalıyor ama tek tek günlerde yaklaşık 1 kWh/m² sapabiliyor. Kaba planlama için kullanılabilir, kesin üretim hesabı için uygun değildir.

### Örnek tahminler

| Sıcaklık (°C) | Nem (%) | Bulutluluk (%) | Ay | Tahmin (kWh/m²/gün) |
|---|---|---|---|---|
| 30 | 35 | 5 | Temmuz | 8.03 |
| 15 | 60 | 40 | Nisan | 5.04 |
| 8 | 80 | 90 | Aralık | 0.30 |

Son örnekte tahmin, verideki en düşük gözlemin (0.50) altında kaldı. Doğrusal model uç girdilerde gerçekçi aralığın dışına çıkabiliyor.

## Mini uygulama

`predict_solar()` fonksiyonu sıcaklık, nem, bulutluluk ve ay değerlerini sorar, tahmini bir cümleyle yazdırır:

```
Ortalama sıcaklık (°C): 28
Bağıl nem (%): 30
Bulutluluk (%): 7
Ay (1-12): 8

Bu koşullarda günlük güneş radyasyonu yaklaşık 7.62 kWh/m² olacak.
Panel verimi için çok iyi bir gün.
```

## Nasıl çalıştırılır?

1. Yukarıdaki **Colab'de aç** butonuna tıkla.
2. **Çalışma zamanı > Tümünü çalıştır** de. Veri, ilk hücrede NASA POWER'dan otomatik indirilir ve `gunes_verisi.csv` olarak kaydedilir.
3. Son hücrelerde `predict_solar()` senden değer bekler, kutucuğa yazıp Enter'a bas.

Kullanılan araçlar: Python, Google Colab, Pandas, NumPy, Matplotlib, scikit-learn.

## Dosyalar

| Dosya | Açıklama |
|---|---|
| `gunes_enerjisi_tahmini.ipynb` | Tüm proje: veri, analiz, model, değerlendirme ve mini uygulama |
| `gunes_verisi.csv` | NASA POWER'dan indirilen günlük veri (Marmaris, 2025, 365 satır) |
| `README.md` | Bu dosya |

## Sınırlılıklar ve iyileştirme fikirleri

- Veri tek yıl ve tek konumdan geliyor.
- Günler rastgele bölündüğü için komşu günler hem eğitimde hem testte yer alıyor, test skoru biraz iyimser olabilir.
- Ay sayısal olarak kullanıldığı için Aralık ile Ocak komşuluğu modellenemiyor. Radyasyon yıl içinde ters U şeklinde değişiyor, doğrusal model bunu tam yakalayamıyor.
- İyileştirme olarak ayı döngüsel bir özelliğe çevirmek, doğrusal olmayan bir model denemek ve daha çok yıl ya da konum eklemek düşünülebilir (denenmedi).

## Hazırlayan

Esra Konya, Ekim 2026
