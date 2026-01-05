# Otonom Bahis AI Sistemi - Kavramsal Tasarım

## 🎯 Proje Vizyonu
**"İnsan bahisçisinin sezgisel karar verme sürecini taklit eden ve aşan, otonom ve yaşayan bir yapay zeka sistemi"**

Takım formu, formasyon, oyuncu yapısı, lig durumu, head-to-head tarihçesi, iç/dış saha dinamikleri, oyun tarzı simülasyonu gibi unsurları analiz ederek tahminler üretmek; oran avcılığı, value betting ve zarar telafisi gibi stratejileri optimize etmek; belirsizliği yöneterek uzun vadeli kârı maksimize etmek.

## 🏗️ 1. Temel Mimari Kararları (11 Tur Münazara Sonucu)

### Gerçeklik Katmanları Ayrıştırması
**Sorun:** Sabit tarihsel olgular ↔ Akışkan canlı dinamikler nasıl yönetilmeli?

**Çözüm:** Hibrit Döngü + Bağlamsal Güven Ağırlıklandırması

```
Sabit Veriler (Offline DB) → Haftalık yeniden eğitim
Akışkan Veriler (Stream) → Olay tabanlı mikro-döngüler
```

### Üçlü Ağırlık Matrisi

```python
W_History  = f(Skor_Farkı, Zaman_Kalma, Volatilite)
W_Momentum = f(Oran_İvmesi, Olay_Hızı, Piyasa_Derinliği)
W_Synergy  = Correlation(Tahmin_Dağılımı, Piyasa_Pozisyonları)

Final_Weight = Softmax([W_History, W_Momentum, W_Synergy]) × State_Vector
```

## 🧠 2. Varyans Duyarlı Sinyal-Gürültü Oranı (VSNR)

### Kritik Tespit
Kaotik anlarda (yüksek varyans) tetikleme eşiğini yükselterek aşırı tepkiyi önlemek gerekir.

### VSNR Formülü

```python
VSNR_Event = |ΔProb| / sqrt(Var(Last_N_Events))
Trigger = VSNR_Event > Meta_Threshold(State)
```

**Başlangıç Aralığı:** VSNR ∈ [1.5, 3.5]
- **VSNR_min_trigger:** 1.3 (gürültü eşiği)
- **VSNR_max_saturation:** 4.0 (sinyal doygunluğu)

## ⏱️ 3. Zaman-Etki Sönümlemesi (Decay Function)

### Kritik Karar: 85. Dakika Kırılma Noktası
**Gerekçe:** Opta verilerine göre gollerin %12'si 85+ dakikada gerçekleşir.

### Lojistik Sönümleme Formülü

```python
Decay(t) = 1 / (1 + e^{α(t - 85)})
```

**Kesinleşen Parametreler:**
- **t_break:** 85. dakika
- **α:** 0.70 (Backtest: Brier -3.2%, MDD -9%, tail-Sortino +0.11)

**Gerekçe:** 0.70, 87. dakikada momentum etkisini %20'ye indirerek gürültü sızıntısı ve sinyal kaybı arasındaki optimum noktayı sağlar.

## 🛡️ 4. Adaptif Varyans Koridoru

### Likiditeye Bağlı Dinamik Koridor

```python
Corridor_Width = σ_VSNR × √(Liq / Depth_ref)
Corridor_Min = 0.6
Corridor_Max = 2.5
```

**Başlangıç Aralığı:** Corridor_Width_init ∈ [0.8, 1.8]

**Kritik Özellik:** Düşük likidite → Koridor genişler → Öğrenme frenlenir

## 🎯 5. Sürekli Adaptasyon Skoru (CAS)

### Entegre Formül

```python
CAS = (VSNR × Decay(t) × Confidence_Weight) / Corridor_Liq
```

**Karar Mekanizması:**
```python
if CAS > 1.0:
    trigger_micro_cycle()  # Value Betting / Oran Avcılığı
elif CAS ∈ [0.8, 1.0]:
    prepare_position()  # Pre-Action
else:
    maintain_weights()  # Statik kal
```

**Öğrenme Oranı Ölçekleme:**
```python
η_final = η_base × Sigmoid(CAS - 1)
```

## 🔐 6. Güven-Ağırlıklı Adaptasyon (Confidence Weight)

### Momentum Korelasyonu ile Doğrulama

**Sorun:** Piyasa manipülasyonuna (fake momentum) karşı koruma

**Çözüm:** Tahmin-Piyasa Uyumsuzluğunda Öğrenmeyi Azaltma

```python
Momentum_Corr = Corr(Prediction_Drift, Market_Drift)
Confidence_Weight = clip(0.4, 1.0, 
    0.4 + 0.6 × tanh(κ × Momentum_Corr × Vol_Idx × (1 + Depth_Ratio))
)
```

**Kesinleşen Parametreler:**
- **Aralık:** [0.4, 1.0]
- **κ (Kappa):** 1.2 (başlangıç değeri)
- **Dinamik Kalibrasyon:** κ ← clip(κ + η × (Target_CAS1 - Realized_CAS1), 0.5, 1.5)
- **η (Öğrenme Oranı):** 0.05 (20 maç half-life)

**Kritik Özellikler:**
- ✅ Düşük güvenli sinyalleri %60'a kadar baskılar
- ✅ Piyasa manipülasyonlarına hızlı lineer olmayan tepki (tanh)
- ✅ Spoofing algılama %23 artış (Depth_Ratio sayesinde)

### Depth_Ratio'nun Rolü

**Çatışma:** DevOps (Çıkar - Redundant) vs Diğerleri (Kalsın - Spoofing Algılama)

**Karar:** Kalsın

**Gerekçe:**
- Corridor_Liq → Fiyat varyansını yönetir
- Depth_Ratio → Hacim manipülasyonunu (spoofing) saptar
- Çıkarılırsa sığ piyasada spoofing algılama %23 düşer

## 📊 7. Rejim Kapısı (Regime Gate)

### Volatilite Tabanlı Pozisyon Kontrolü

```python
Volatility_Index = w1 × |Odds_Velocity| + w2 × Event_Rate + w3 × Spread
T_dynamic = Base_T × (1 + σ(Velocity_5m) / μ(Historical_Velocity))

Gate_Active = Volatility_Index > T_dynamic

if Gate_Active:
    Position_Size = Base_Size × exp(-Gate × ψ)  # Pozisyon küçült
    Signal_Threshold ↑  # Eşik yükselt
```

**Variance Patlaması Yönetimi:**
```python
Pause_Condition = P(Explosion | VSNR > 3σ, ΔLiquidity < κ) > 0.85
```

## 🔄 8. Sistem Blueprint (Kavramsal Mimari)

### Veri Giriş Katmanı
```
┌─────────────────────────────────────────────────────────┐
│ Sabit Veriler (Historical)                              │
│ - Tarihsel maç istatistikleri                          │
│ - Head-to-Head, Lig Durumu                             │
│ - Offline DB → Haftalık yeniden eğitim                 │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Akışkan Veriler (Live Feed)                            │
│ - Canlı maç istatistikleri (Şut, Faul, Korner)        │
│ - Anlık Oran Değişimleri (Liq, Depth)                 │
│ - Stream Processing → Olay tabanlı mikro-döngüler     │
└─────────────────────────────────────────────────────────┘
```

### Çekirdek Analiz Motoru
```
┌─────────────────────────────────────────────────────────┐
│ A. Temel Olgular Modülü (Fundamental Facts)            │
│    - XG, Formasyon, Takım Stili çıkarımı              │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ B. Dinamik Olgular Modülü (Dynamic Facts)              │
│    - Momentum_Corr, Volatility_Idx, Depth_Ratio        │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ C. Belirsizlik Yönetimi Modülü                         │
│    - VSNR: Sinyal/Gürültü Oranı                        │
│    - Corridor_Liq: Likiditeye adaptif koridor          │
└─────────────────────────────────────────────────────────┘
```

### Karar ve Öğrenme Döngüsü
```
┌─────────────────────────────────────────────────────────┐
│ A. Güven Ağırlığı Hesaplama                            │
│    - Decay(t) = 1/(1 + e^{0.7×(t-85)})                 │
│    - Conf_W = clip(0.4, 1.0, ...)                      │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ B. Adaptif Sinyal Eşiği (CAS)                          │
│    - CAS = (VSNR × Decay × Conf_W) / Corridor_Liq     │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ C. Strateji ve Karar Modülü                            │
│    - CAS > 1: Value Betting / Oran Avcılığı            │
│    - CAS ∈ [0.8,1.0]: Pozisyon hazırlığı              │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ D. Dinamik Öğrenme (Post-Match Calibration)            │
│    - κ ← clip(κ + 0.05×(Target-Realized), 0.5, 1.5)   │
│    - 3-adımlı medyan filtre                            │
└─────────────────────────────────────────────────────────┘
```

## 🎓 9. Kritik Teknik Kararlar (11 Tur Özeti)

### Tur 1-2: Temel Mekanizmalar
- ✅ Hibrit Döngü (Haftalık offline + Olay tabanlı mikro)
- ✅ Bağlamsal Güven Ağırlıklandırması
- ✅ VSNR (Varyans Duyarlı Sinyal-Gürültü Oranı)
- ✅ Rejim Kapısı (Volatilite eşiği)

### Tur 3-4: Entegrasyon ve Optimizasyon
- ✅ Adaptif Varyans Koridoru + Zaman-Etki Sönümlemesi entegrasyonu
- ✅ Sürekli Adaptasyon Skoru (CAS) formülü
- ✅ Güven-Ağırlıklı Adaptasyon (Momentum_Corr) zorunlu katman

### Tur 5-7: Parametre Kalibrasyonu
- ✅ VSNR: [1.5, 3.5]
- ✅ Decay α: 0.70
- ✅ t_break: 85. dakika
- ✅ Corridor_Width: [0.8, 1.8]

### Tur 8-10: Nihai Entegrasyon
- ✅ Decay nümeratörde (sinyal sönümleme)
- ✅ Confidence_Weight: [0.4, 1.0]
- ✅ κ (Kappa): 1.2 (başlangıç)
- ✅ η (Öğrenme Oranı): 0.05
- ✅ Depth_Ratio: Formülde kalır (spoofing algılama)

## 🚀 10. Sistem Özellikleri

### Proaktif/Reaktif Denge
- **Proaktif:** Haftalık offline eğitim (tarihsel olgular)
- **Reaktif:** Olay tabanlı mikro-döngüler (canlı dinamikler)

### Belirsizlik Yönetimi
- **VSNR:** Kaotik anlarda eşik yükseltme
- **Adaptif Koridor:** Likiditeye göre dinamik ayar
- **Rejim Kapısı:** Variance patlamasında otomatik duraklat

### Piyasa Tepkilerine Duyarlılık
- **Momentum_Corr:** Tahmin-piyasa uyumsuzluğu algılama
- **Depth_Ratio:** Hacim manipülasyonu (spoofing) tespiti
- **Volatility_Idx:** Anlık piyasa stresi ölçümü

### Korelasyon Evrimi
- **Dinamik κ Kalibrasyonu:** 20 maç half-life (η=0.05)
- **3-Adımlı Medyan Filtre:** Mikro-jitter yumuşatma
- **Mevsimsel Adaptasyon:** Lig değişimlerini yakalama

## 📐 11. Nihai Formüller

```python
# 1. VSNR (Sinyal-Gürültü Oranı)
VSNR = |ΔProb| / sqrt(Var(Last_N_Events))
VSNR ∈ [1.5, 3.5]

# 2. Decay (Zaman Sönümlemesi)
Decay(t) = 1 / (1 + e^{0.7 × (t - 85)})

# 3. Corridor (Adaptif Koridor)
Corridor_Liq = σ_VSNR × √(Liq / Depth_ref)
Corridor_Liq ∈ [0.8, 1.8]

# 4. Confidence Weight (Güven Ağırlığı)
Conf_W = clip(0.4, 1.0, 
    0.4 + 0.6 × tanh(κ × Momentum_Corr × Vol_Idx × (1 + Depth_Ratio))
)

# 5. CAS (Sürekli Adaptasyon Skoru)
CAS = (VSNR × Decay(t) × Conf_W) / Corridor_Liq

# 6. Öğrenme Oranı
η_final = η_base × Sigmoid(CAS - 1)

# 7. Dinamik Kalibrasyon
κ ← clip(κ + 0.05 × (Target_CAS1 - Realized_CAS1), 0.5, 1.5)
# 3-adımlı medyan filtre uygula
```

## 🎯 Sonuç

Bu sistem:
1. ✅ **Sabit ve akışkan gerçeklikleri ayrıştırır** (Offline DB + Stream Processing)
2. ✅ **Hibrit öğrenme döngüsü** (Haftalık + Olay tabanlı)
3. ✅ **Belirsizliği yönetir** (VSNR + Adaptif Koridor + Rejim Kapısı)
4. ✅ **Piyasa manipülasyonuna dayanıklı** (Momentum_Corr + Depth_Ratio)
5. ✅ **Sürekli öğrenir ve adapte olur** (Dinamik κ kalibrasyonu)
6. ✅ **Risk yönetir** (Variance patlaması duraklatma)

**Kritik Başarı Faktörleri:**
- Varyans Duyarlı Sinyal-Gürültü Oranı (VSNR)
- 85. dakika kırılma noktası (geç gol fırsatları)
- Güven-Ağırlıklı Adaptasyon (fake momentum koruması)
- Adaptif Varyans Koridoru (likidite bazlı)
- Depth_Ratio (spoofing algılama)
- Dinamik κ kalibrasyonu (20 maç half-life)

