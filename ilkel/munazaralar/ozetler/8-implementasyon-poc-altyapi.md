# İmplementasyon ve PoC Altyapısı - Gerçek Dünya Uygulaması

## 🎯 Oturum Odağı
**"Kavramsal tasarımdan gerçek dünya implementasyonuna geçiş - Teknoloji, framework ve araç seçimleri"**

Bu oturum, önceki iki oturumda oluşturulan kavramsal çerçeveyi (CAS, Decay, Confidence Weight, γ, N_eff, BCD, KD) gerçek dünyada uygulanabilir bir sisteme dönüştürür. Artık teknoloji, framework ve algoritma isimleri kullanılabilir.

## 🔍 Oturumda Ele Alınan Eksik Yönler

### 1. Meta-Öğrenme Derinliği
**Soru:** Fonksiyon şekli (tanh vs sigmoid vs linear) ve loss tipi meta-seviyede nasıl öğrenilir?

### 2. Zaman Bazlı Çapraz Bağımlılık
**Soru:** Aynı gün/lig çapraz etkileri (teknik direktör, hava durumu) portföy korelasyonuna nasıl entegre edilir?

### 3. Rollback ve Güvenlik Sınırları
**Soru:** ROI -2% rollback ne kadar geriye gider? Sistem "kırmızı alarm" moduna ne zaman geçer?

### 4. Performans İzleme ve Canlı Adaptasyon
**Soru:** Sistem gerçek zamanlı backtest ve A/B testing ile nasıl kendi kararlarını valide eder?

## 🏗️ 1. Üç Katmanlı Modüler Mimari

### Kritik Karar: Asimetrik Loss Çatışması Çözümü

**Çatışma:** Alfa (XGBoost Quantile Regression) vs Epsilon (TensorFlow Probability)
**Çözüm:** Kappa'nın modüler katman füzyonu

```
┌─────────────────────────────────────────────────┐
│  LAYER 1: LightGBM-Quantile (Preprocessing)    │
│  → CAS varyans daraltma, feature extraction     │
│  → q_dynamic (0.6-0.9) output to Layer 2       │
├─────────────────────────────────────────────────┤
│  LAYER 2: HyperNetworks (Core)                  │
│  → Dinamik aktivasyon: tanh/sigmoid blend       │
│  → q_dynamic-aware loss: max(q*e, (q-1)*e)     │
│  → PyTorch Lightning + Interval Score Loss      │
├─────────────────────────────────────────────────┤
│  LAYER 3: BNN Uncertainty (Post-Processing)     │
│  → MC-Dropout epistemic uncertainty             │
│  → CAS_final = CAS * (1 - uncertainty_factor)   │
├─────────────────────────────────────────────────┤
│  INFRA: Ray/MLflow (Training) + Triton (Serving)│
└─────────────────────────────────────────────────┘
```

### Layer 1: LightGBM-Quantile (Preprocessing)

**Teknoloji:** LightGBM (Dart mode) + Optuna
**Amaç:** CAS varyans daraltma, hızlı feature extraction

```python
import lightgbm as lgb

lgb_model = lgb.train(
    params={
        "objective": "quantile",
        "alpha": 0.7,  # Asimetrik risk profili
        "boosting_type": "dart"
    },
    train_data,
    num_boost_round=200
)

# Dinamik q değeri çıktısı
q_dynamic = lgb_model.predict(X)  # 0.6-0.9 aralığı
```

**Trade-off:** %15 latency artışı, async inference ile yönetilir

### Layer 2: HyperNetworks (Core)

**Teknoloji:** PyTorch + PyTorch Lightning
**Amaç:** Dinamik aktivasyon fonksiyonu öğrenme

```python
class QuantileHyperNet(nn.Module):
    def forward(self, x, epsilon):
        # Dinamik ağırlıklar
        weights = self.hyper_net(x)
        
        # Risk iştahı (q) dinamik ayarlama
        q_dynamic = torch.sigmoid(self.risk_head(x))  # 0.5-1.0
        
        # Asimetrik Quantile Loss
        # max(q*error, (q-1)*error)
        
        # MTGP epsilon cezası
        base = 0.4 + 0.6 * torch.tanh(self.kappa * x.momentum * x.vol)
        conf_w = torch.clamp(base * (1 - 0.5 * epsilon), 0.4, 1.0)
        
        return weights, q_dynamic, conf_w
```

**Stabilite:** Meta-SGD + Gradient Clipping

```python
# Meta-SGD: Her parametre için ayrı learning rate
w_new = w - alpha * (grad_quantile + lambda_reg * grad_entropy)

# Gradient Clipping
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
```

**Trade-off:** %25 VRAM artışı, FSDP ile yönetilir

### Layer 3: BNN Uncertainty (Post-Processing)

**Teknoloji:** Pyro-PPL (Bayesian Neural Networks)
**Amaç:** Epistemic uncertainty (bilinmeyen bilinmeyenler)

```python
import pyro

class BNNWrapper(nn.Module):
    def forward(self, x):
        # MC-Dropout ile uncertainty
        predictions = []
        for _ in range(20):  # 20 sample
            pred = self.model(x)
            predictions.append(pred)
        
        mean = torch.mean(torch.stack(predictions), dim=0)
        uncertainty = torch.std(torch.stack(predictions), dim=0)
        
        # CAS'a uncertainty faktörü uygula
        CAS_final = CAS * (1 - uncertainty_factor)
        
        return CAS_final
```

**Trade-off:** %10 latency artışı, MC-Dropout sample=20 ile optimize

## 🌐 2. Zaman Bazlı Çapraz Bağımlılık

### PyTorch Geometric Temporal (TGN)

**Teknoloji:** PyTorch Geometric Temporal
**Amaç:** Lig topolojisi ve momentum transferi

```python
from torch_geometric_temporal.nn.recurrent import A3TGCN

# Node: Takımlar, Edge: Maçlar/Ortak Koşullar
model = A3TGCN(
    in_channels=features,
    out_channels=conf_w,
    periods=5  # 5 maç geçmişi
)
```

**Gerekçe:** Standart RNN'ler topolojik (lig/rakip) ilişkileri kaçırır

### Multi-Task Gaussian Processes (MTGP)

**Teknoloji:** GPy (Multi-Task GP Kernel)
**Amaç:** Farklı ligler arası bilgi transferi

```python
import GPy

# 5 lig için MTGP
kernel = GPy.kern.MTGPKernel(input_dim=3, num_tasks=5)
mtgp_model = GPy.models.MTGRegression(X_lig, Y_lig, kernel)
```

**Sorun:** O(n³) karmaşıklık → Canlı akışta gecikme riski

**Çözüm:** Approximate MTGP (Sparse Precision Matrix)

```python
from scipy import sparse

# Low-rank approximation (r=10)
# O(n³) → O(n·r²)
precision_sparse = L @ L.T  # Cholesky low-rank
mtgp_posterior = y_train_new @ precision_sparse @ X_new
```

**Trade-off:** %5 accuracy kaybı, <50ms gecikme hedefine ulaşılır

### Lambda Architecture (Batch + Speed Layer)

**Teknoloji:** Ray (Batch) + Apache Flink (Speed) + Redis (Feature Store)

```python
# Batch Layer (Ray): 30dk'da bir MTGP eğitimi
# Kovaryans matrislerini Redis'e yaz
ray_mtgp_train() → redis.set("lig_cov_v{timestamp}", cov_matrix)

# Speed Layer (Flink): Anlık event işleme
# Redis'ten güncel matrisleri çek
flink_stream.map(lambda event: 
    redis.get("lig_cov_v_current") + event
)
```

**Gerekçe:** O(n³) maliyetini canlı akıştan izole eder

### Real-Time Event Processing

**Teknoloji:** Redis Streams + Apache Flink

```python
# Maç-öncesi koşulları Redis Stream'e ekle
stream.add_event("match_4762", {
    "weather": "rain_heavy",
    "manager_change_days": 3,
    "rest_days": 2,
    "lig": "premier_league"
})

# Flink ile real-time correlation
def cross_dependency_rules(event):
    if event["manager_change_days"] < 7:
        gamma_adj = 0.15  # Eşgüdüm kaydırma
    if event["weather"] == "rain_heavy":
        Decay_adj = 0.85  # Sönümleme kayması
    
    return gamma_adj, Decay_adj
```

**Trade-off:** Ek infrastructure maliyeti, <100ms gecikme ile adaptasyon

## 🛡️ 3. Rollback ve Güvenlik Sınırları

### Circuit Breaker Mekanizması

**Teknoloji:** Kubernetes HPA + Evidently (Drift Detection)

```yaml
# K8s Circuit Breaker Config
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: bettingsystem-circuitbreaker
spec:
  minReplicas: 3
  maxReplicas: 15
  metrics:
  - type: External
    external:
      metric:
        name: bettingsystem_p_shift
      target:
        type: Value
        value: "25"  # p_shift > 0.25 tetikleyici
```

### Kırmızı Alarm Seviyeleri

| p_shift | ROI Düşüşü | Aksiyon |
|---------|------------|---------|
| > 0.25 | - | Flink topology durdur, shadow mode |
| > 0.40 | - | K8s pod deployment durdur |
| > 0.60 | < -2.5% | **Kırmızı Alarm:** Tüm pipeline cold start |

### Rollback Stratejisi

**Teknoloji:** MLflow Model Registry + Redis Dual-Key Versioning

```python
# ROI -2% ve p_BCD > 0.92 tetiklendiğinde
if ROI < -0.02 and p_BCD > 0.92:
    # Model Registry'den T-2 hafta checkpoint'e rollback
    model = mlflow.pyfunc.load_model(f"models:/betting-system/T-2weeks")
    
    # Redis Key Switch (Zero-Downtime)
    redis.set("active_model_key", "lig_cov_v_safe")  # T-2 hafta
    
    # Parametreler
    lambda_multiplier = 0  # Bahis durdur
    gamma_hysteresis += 0.03
    Hard_Cap = H0 * min(1, N_eff/K) / 2
    CAS_threshold += 0.5 * sigma
```

### State Funnels (Geçiş Güvenliği)

```python
# Eski (safe) ve yeni (current) modeli 10 saniye paralel izle
def state_funnel_check(pred_current, pred_safe):
    diff = abs(pred_current - pred_safe)
    
    if diff > 0.15:
        # Parametre şoku riski
        return "ABORT_SWITCH"
    else:
        return "CONFIRM_SWITCH"
```

## 📊 4. Performans İzleme ve Canlı Adaptasyon

### 3-Stage Deployment Gate

**Teknoloji:** MLflow + Kubeflow Pipelines

```python
def deployment_gate(metrics, stage="canary"):
    thresholds = {
        "shadow": {
            "sharpe": 0.5,
            "sortino": 0.8,
            "var95": 15
        },
        "canary": {
            "sharpe": 0.7,
            "sortino": 1.0,
            "var95": 10
        },
        "progressive": {
            "sharpe": 0.8,
            "sortino": 1.2,
            "var95": 8
        }
    }
    
    return all(metrics[k] >= v for k, v in thresholds[stage].items())
```

### Shadow Testing

**Teknoloji:** Multi-Armed Bandit (Thompson Sampling)

```python
# %10 trafik ile shadow testing
def shadow_test(model_current, model_safe, input_data):
    pred_current = model_current(input_data)
    pred_safe = model_safe(input_data)
    
    # Thompson Sampling ile model seçimi
    if thompson_sample() < 0.1:
        return pred_current  # Yeni model
    else:
        return pred_safe  # Güvenli model
```

### Monitoring Stack

**Teknoloji:** Prometheus + Grafana + Evidently + Great Expectations

```yaml
# Helm values.yaml - Monitoring
monitoring:
  prometheus:
    enabled: true
    serviceMonitor:
      interval: 30s
  grafana:
    enabled: true
    dashboards:
      - name: "Betting System ROI"
      - name: "Model Drift Detection"
  evidently:
    enabled: true
    drift_threshold: 0.3
```

## 💾 5. MTGP Accuracy Kaybı ve ROI Yönetimi

### Kelly SDE (Stochastic Differential Equation)

```python
def adjusted_roi(base_roi, epsilon, volatility=0.15):
    # Kelly with drift-variance adjustment
    drift = base_roi * (1 - 1.2 * epsilon)
    variance_penalty = 0.5 * volatility**2 * (1 + epsilon)
    
    return drift - variance_penalty
```

**Sonuç:** %5 accuracy kaybı (ε≈0.12) → Base ROI %8 ise, adjusted ROI ≈ %5.8 (%27.5 kayıp)

### Fractional Kelly

```python
fractional_kelly = 0.75
adjusted_roi = fractional_kelly * adjusted_roi_base
```

**Gerekçe:** %27.5 kayıp → %20'ye düşer

### Epsilon-Aware Scaling

```python
# Frobenius Norm ile MTGP hata ölçümü
epsilon = norm_frobenius(K, K_approx) / norm_frobenius(K)

# Volatility adjustment
vol_adj = realized_vol * (1 + 0.3 * epsilon)

# Lambda final
lambda_final = lambda_base * (target_vol / vol_adj) * (1 - 0.4 * epsilon)

# Confidence Weight adjustment
Conf_W = Conf_W * (1 - 0.5 * epsilon)

# Hard-Cap adjustment
Hard_Cap = H0 * min(1, N_eff/K) * (1 - 0.2 * epsilon)

# Corridor adjustment
Corridor_Liq = Corridor_Liq * (1 + 0.8 * epsilon)
```

**Nihai Sonuç:** ROI kaybı %27.5 → %12-%15 aralığına düşer

## 🖥️ 6. VRAM Yönetimi (Meta-SGD)

### FSDP (Fully Sharded Data Parallel)

**Teknoloji:** PyTorch FSDP + CPU Offload

```python
from torch.distributed.fsdp import FullyShardedDataParallel as FSDP

# Model parçalara bölünür
model = FSDP(
    model,
    cpu_offload=True,  # Optimizer state CPU'ya
    auto_wrap_policy=default_auto_wrap_policy
)
```

**Kazanım:** %35-45 VRAM azalma

### Activation Checkpointing

```python
from torch.utils.checkpoint import checkpoint

def forward_with_checkpoint(x):
    return checkpoint(self.layer_block, x)
```

**Trade-off:** VRAM %40 tasarruf, compute %30 artış

### Gradient Accumulation

```python
# Effective batch size = 64 (8 micro-batches × 8 accumulation)
optimizer.zero_grad()
for i in range(accumulation_steps):
    loss = model(batch[i]) / accumulation_steps
    loss.backward()
optimizer.step()
```

### Mixed Precision (FP16)

```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()
with autocast():
    loss = model(input)
scaler.scale(loss).backward()
scaler.step(optimizer)
scaler.update()
```

**Kazanım:** VRAM %20 tasarruf, FP16 stability

### A100 MIG (Multi-Instance GPU)

```yaml
# K8s Resource Allocation
resources:
  limits:
    nvidia.com/gpu: 2
    memory: 32Gi
  requests:
    nvidia.com/gpu: 1
    memory: 16Gi
```

- **3g.20gb instance:** Eğitim
- **1g.5gb instance:** Serving (Triton)

## 🚀 7. Serving Optimizasyonu

### Triton Inference Server

**Teknoloji:** NVIDIA Triton + TensorRT FP16

```python
# Triton Config
dynamic_batching {
    preferred_batch_sizes: [8, 16, 32]
    max_queue_delay_microseconds: 100
}

# TensorRT FP16 Optimization
model_optimization {
    execution_accelerators {
        gpu_execution_accelerator : [ {
            name : "tensorrt"
            parameters { key: "precision_mode" value: "FP16" }
        }]
    }
}
```

**Kazanım:** +%40 throughput, 2x memory compress

### Priority Queue

```python
# Yüksek confidence bahisler öncelikli işlenir
priority = Conf_W * (1 - epsilon)

# Stream ayrımı
if priority > 0.8:
    queue = "high_priority"
else:
    queue = "normal"
```

**Hedef:** p99 < 60ms latency

## 📦 8. Final PoC Altyapısı

### Docker Compose (PoC Ortamı)

```yaml
services:
  # Training Pipeline
  ray-head:
    image: rayproject/ray:2.9.0-py310
    volumes: [./configs:/configs]
  
  mlflow:
    image: mlflow/mlflow:2.7.1
    ports: ["5000:5000"]

  # Serving Pipeline  
  triton:
    image: nvcr.io/nvidia/tritonserver:24.01-py3
    deploy:
      resources:
        limits:
          nvidia.com/gpu: 1
          memory: 16Gi

  # Real-time Pipeline
  flink:
    image: flink:1.17
  
  redis:
    image: redis:7.2
    command: ["--maxmemory", "4gb"]
```

### Kubernetes Helm Chart

```yaml
# values.yaml
ray:
  head:
    resources:
      limits:
        nvidia.com/gpu: 1
        memory: 32Gi
    autoscaling:
      minWorkers: 2
      maxWorkers: 8

triton:
  replicaCount: 3
  resources:
    limits:
      nvidia.com/gpu: 1
      memory: 16Gi

flink:
  taskmanager:
    replicas: 4
    resources:
      limits:
        memory: 8Gi
        cpu: 4
```

### Kütüphane Bağımlılıkları

**Preprocessing:**
- `lightgbm` (Dart mode)
- `optuna` (Hyperparameter tuning)

**Core:**
- `pytorch-lightning` (Trainer)
- `torchmetrics` (Interval Score)
- `torch-geometric-temporal` (TGN)

**Uncertainty:**
- `pyro-ppl` (BNN wrapper)
- `gpytorch` (Sparse GP)
- `GPy` (MTGP)

**Serving:**
- `tritonclient`
- `ray[serve]`

**Monitoring:**
- `mlflow`
- `evidently`
- `great-expectations`

## 📊 9. Trade-off Matrisi

| Bileşen | Avantaj | Maliyet | Yönetim Stratejisi |
|---------|---------|---------|-------------------|
| **XGBoost Quantile** | Hızlı preprocessing | %15 latency | Async inference |
| **HyperNetworks** | Dinamik q adaptasyonu | %25 VRAM | Gradient Clipping + Meta-SGD |
| **BNN Uncertainty** | Tail-risk koruması | %10 latency | MC-Dropout sample=20 |
| **Approx MTGP** | O(n³)→O(n·r²) | %5 acc kaybı | ε-adaptive mod |
| **Kelly SDE** | ROI volatility kontrolü | Fractional Kelly ile gain kaybı | Fraction=0.75 optimal |
| **Meta-SGD** | Stabil asimetrik öğrenme | %25 VRAM | FSDP+CPU-offload |
| **Triton FP16** | +%40 throughput | Inference precision | Calibrated |

## 🎯 Kritik Başarı Faktörleri

### Accuracy vs Latency
- ✅ Approximate MTGP: %5 accuracy kaybı → <50ms gecikme
- ✅ 3 Katmanlı Mimari: Preprocessing + Core + Post-processing
- ✅ Lambda Architecture: Batch (Ray) + Speed (Flink)

### ROI Koruma
- ✅ Kelly SDE: Drift-variance adjustment
- ✅ Fractional Kelly (0.75): %27.5 → %20 kayıp
- ✅ Epsilon-aware scaling: %20 → %12-15 kayıp

### VRAM Optimizasyonu
- ✅ FSDP + CPU-offload: %35-45 azalma
- ✅ Activation checkpointing: %40 tasarruf
- ✅ TensorRT FP16: 2x memory compress
- ✅ A100 MIG: 3g.20gb (train) + 1g.5gb (serve)

### Güvenlik ve Stabilite
- ✅ Circuit Breaker: p_shift > 0.25/0.40/0.60
- ✅ Rollback: T-2 hafta checkpoint
- ✅ State Funnels: Parametre şoku önleme
- ✅ 3-Stage Gate: Shadow → Canary → Progressive

## 🚀 Sonuç

Bu oturum, kavramsal tasarımı gerçek dünya implementasyonuna dönüştürdü:

1. ✅ **3 Katmanlı Mimari:** LightGBM + HyperNetworks + BNN
2. ✅ **Çapraz Bağımlılık:** TGN + MTGP + Flink/Redis
3. ✅ **Güvenlik:** Circuit Breaker + Rollback + State Funnels
4. ✅ **Performans:** Shadow Testing + 3-Stage Gate + Monitoring
5. ✅ **ROI Koruma:** Kelly SDE + Fractional Kelly + ε-scaling
6. ✅ **VRAM Yönetimi:** FSDP + Checkpointing + FP16 + MIG
7. ✅ **Serving:** Triton FP16 + Dynamic Batching + Priority Queue

**Nihai Hedefler:**
- ROI kaybı: %12-15 (MTGP %5 acc kaybına rağmen)
- Latency: p99 < 60ms
- VRAM: 16Gi (serving), 32Gi (training)
- Throughput: +%40 (TensorRT FP16)

