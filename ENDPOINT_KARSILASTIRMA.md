# API Endpoint Karşılaştırma Raporu

## 📊 Genel Endpoint Sayıları

| Dosya | Toplam Endpoint | Detaylı Dokümante Edilmiş |
|-------|----------------|--------------------------|
| **api-kullanim.txt** | ~25-30 | ~25 |
| **cursor-api.md** | 36 | 36 |

---

## 🔍 Endpoint Listesi ve Karşılaştırma

### 1. Timezone Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /timezone` | ❌ Yok | ✅ Var |

**Sonuç:** cursor-api.md'de var, api-kullanim.txt'de yok.

---

### 2. Countries Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /countries` | ❌ Yok | ✅ Var |

**Sonuç:** cursor-api.md'de var, api-kullanim.txt'de yok.

---

### 3. Leagues Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /leagues` | ✅ Var | ✅ Var |
| `GET /leagues/seasons` | ❌ Yok | ✅ Var |

**Sonuç:** 
- `/leagues` → Her ikisinde de var
- `/leagues/seasons` → Sadece cursor-api.md'de var

---

### 4. Teams Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /teams` | ✅ Var | ✅ Var |
| `GET /teams/statistics` | ✅ Var | ✅ Var |
| `GET /teams/seasons` | ✅ Var | ✅ Var |
| `GET /teams/countries` | ✅ Var | ✅ Var |

**Sonuç:** Tüm team endpoint'leri her iki dosyada da var.

---

### 5. Venues Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /venues` | ❌ Yok | ✅ Var |

**Sonuç:** cursor-api.md'de var, api-kullanim.txt'de yok.

---

### 6. Standings Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /standings` | ✅ Var | ✅ Var |

**Sonuç:** Her ikisinde de var.

---

### 7. Fixtures Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /fixtures` | ✅ Var | ✅ Var |
| `GET /fixtures/rounds` | ✅ Var | ✅ Var |
| `GET /fixtures/headtohead` | ✅ Var | ✅ Var |
| `GET /fixtures/statistics` | ✅ Var | ✅ Var |
| `GET /fixtures/events` | ✅ Var | ✅ Var |
| `GET /fixtures/lineups` | ✅ Var | ✅ Var |
| `GET /fixtures/players` | ✅ Var | ✅ Var |

**Sonuç:** Tüm fixture endpoint'leri her iki dosyada da var.

---

### 8. Injuries Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /injuries` | ✅ Var | ✅ Var |

**Sonuç:** Her ikisinde de var.

---

### 9. Predictions Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /predictions` | ✅ Var | ✅ Var |

**Sonuç:** Her ikisinde de var.

---

### 10. Coaches Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /coaches` | ❌ Yok | ✅ Var |

**Sonuç:** cursor-api.md'de var, api-kullanim.txt'de yok.

---

### 11. Players Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /players` | ✅ Var | ✅ Var |
| `GET /players/seasons` | ✅ Var | ✅ Var |
| `GET /players/profiles` | ✅ Var | ✅ Var |
| `GET /players/statistics` | ✅ Var | ✅ Var |
| `GET /players/squads` | ✅ Var | ✅ Var |
| `GET /players/teams` | ✅ Var | ✅ Var |
| `GET /players/topscorers` | ✅ Var | ✅ Var |
| `GET /players/topassists` | ✅ Var | ✅ Var |
| `GET /players/topyellowcards` | ✅ Var | ✅ Var |
| `GET /players/topredcards` | ✅ Var | ✅ Var |

**Not:** api-kullanim.txt'de bazı endpoint'ler farklı isimlerle geçiyor:
- `Players/Teams Endpoint` → `/players/teams`
- `Top Scorers Endpoint` → `/players/topscorers`
- `Top Assists Endpoint` → `/players/topassists`
- `Top Yellow Cards Endpoint` → `/players/topyellowcards`
- `Top Red Cards Endpoint` → `/players/topredcards`

**Sonuç:** Tüm player endpoint'leri her iki dosyada da var (isimlendirme farklılıkları var).

---

### 12. Transfers Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /transfers` | ❌ Yok | ✅ Var |

**Sonuç:** cursor-api.md'de var, api-kullanim.txt'de yok.

---

### 13. Trophies Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /trophies` | ❌ Yok | ✅ Var |

**Sonuç:** cursor-api.md'de var, api-kullanim.txt'de yok.

---

### 14. Sidelined Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /sidelined` | ❌ Yok | ✅ Var |

**Sonuç:** cursor-api.md'de var, api-kullanim.txt'de yok.

---

### 15. Odds Endpoint'leri

| Endpoint | api-kullanim.txt | cursor-api.md |
|----------|------------------|---------------|
| `GET /odds` | ✅ Var | ✅ Var |
| `GET /odds/live` | ✅ Var | ✅ Var |
| `GET /odds/live/bets` | ✅ Var | ✅ Var |
| `GET /odds/bookmakers` | ✅ Var | ✅ Var |
| `GET /odds/bets` | ✅ Var | ✅ Var |
| `GET /odds/mapping` | ✅ Var | ✅ Var |

**Sonuç:** Tüm odds endpoint'leri her iki dosyada da var.

---

## 📈 Özet Tablo

### Endpoint Kapsamı

| Kategori | api-kullanim.txt | cursor-api.md | Fark |
|----------|------------------|---------------|------|
| **Timezone** | 0 | 1 | +1 cursor-api.md |
| **Countries** | 0 | 1 | +1 cursor-api.md |
| **Leagues** | 1 | 2 | +1 cursor-api.md |
| **Teams** | 4 | 4 | Eşit |
| **Venues** | 0 | 1 | +1 cursor-api.md |
| **Standings** | 1 | 1 | Eşit |
| **Fixtures** | 7 | 7 | Eşit |
| **Injuries** | 1 | 1 | Eşit |
| **Predictions** | 1 | 1 | Eşit |
| **Coaches** | 0 | 1 | +1 cursor-api.md |
| **Players** | 10 | 10 | Eşit |
| **Transfers** | 0 | 1 | +1 cursor-api.md |
| **Trophies** | 0 | 1 | +1 cursor-api.md |
| **Sidelined** | 0 | 1 | +1 cursor-api.md |
| **Odds** | 6 | 6 | Eşit |
| **TOPLAM** | **31** | **36** | **+5 cursor-api.md** |

---

## ✅ Sadece cursor-api.md'de Olan Endpoint'ler

1. ✅ `GET /timezone` - Timezone listesi
2. ✅ `GET /countries` - Ülke listesi
3. ✅ `GET /leagues/seasons` - Sezon listesi
4. ✅ `GET /venues` - Stadyum bilgileri
5. ✅ `GET /coaches` - Teknik direktör bilgileri
6. ✅ `GET /transfers` - Transfer bilgileri
7. ✅ `GET /trophies` - Kupa/trofe bilgileri
8. ✅ `GET /sidelined` - Kadro dışı oyuncular

**Toplam:** 8 endpoint sadece cursor-api.md'de var.

---

## ❌ Sadece api-kullanim.txt'de Olan Endpoint'ler

**Yok** - Tüm api-kullanim.txt'deki endpoint'ler cursor-api.md'de de mevcut.

---

## 🔍 Endpoint Detay Karşılaştırması

### Örnek: `/odds` Endpoint

#### api-kullanim.txt
```
4. /odds Endpoint (Maç Öncesi Oranlar):
● Amaç: Belirli bir maç, lig veya tarihe göre maç öncesi oranları sağlar.
● Özellikler:
○ Maçtan 1-14 gün öncesine kadar olan oranlar görüntülenebilir.
○ Her 3 saatte bir güncellenir.
○ 10 sonuç/sayfa paginasyon vardır.
● Parametreler:
○ fixture: Maç ID'si
○ league: Lig ID'si
○ season: Sezon
○ date: Tarih (YYYY-MM-DD formatında)
○ timezone: Zaman dilimi
○ bookmaker: Bahis şirketi ID'si
○ bet: Bahis türü ID'si
○ page: Sayfa numarası (pagination)
```

**Özellikler:**
- ✅ Amaç açıklaması var
- ✅ Özellikler listelenmiş
- ✅ Güncelleme sıklığı belirtilmiş
- ✅ Parametreler listelenmiş
- ❌ Parametre tipleri yok
- ❌ Zorunlu/opsiyonel bilgisi yok
- ❌ Python kodu yok
- ❌ Response örneği yok
- ❌ HTTP status kodları yok

#### cursor-api.md
```markdown
#### 16.1. Get Odds (Pre-Match)

##### Endpoint
```
GET /odds
```

##### Açıklama
Maç öncesi bahis oranlarını getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `fixture` | integer | Hayır | Maç ID'si |
| `league` | integer | Hayır | Lig ID'si |
| `season` | integer | Hayır | Sezon yılı (YYYY) |
| `date` | string | Hayır | Tarih (YYYY-MM-DD) |
| `timezone` | string | Hayır | Timezone |
| `page` | integer | Hayır | Sayfa numarası |
| `bookmaker` | integer | Hayır | Bahis şirketi ID'si |
| `bet` | integer | Hayır | Bahis tipi ID'si |

##### Python Örneği
```python
import requests

url = "https://v3.football.api-sports.io/odds"
headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}
params = {
    "fixture": 157201
}
response = requests.get(url, headers=headers, params=params)
data = response.json()
print(data)
```

##### Response Örneği (200 OK)
```json
{
  "get": "odds",
  "parameters": {
    "fixture": "157201"
  },
  "errors": [],
  "results": 1,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [...]
}
```

##### Response Kodları
- **200 OK**: Başarılı, bahis oranları döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası
```

**Özellikler:**
- ✅ Amaç açıklaması var
- ✅ Parametreler tablo formatında
- ✅ Parametre tipleri belirtilmiş
- ✅ Zorunlu/opsiyonel bilgisi var
- ✅ Python kodu var
- ✅ Response örneği var
- ✅ HTTP status kodları var
- ❌ Güncelleme sıklığı belirtilmemiş (bazı endpoint'lerde var)

---

## 📊 Detay Seviyesi Karşılaştırması

### Parametre Bilgileri

| Özellik | api-kullanim.txt | cursor-api.md |
|---------|------------------|---------------|
| Parametre isimleri | ✅ Var | ✅ Var |
| Parametre tipleri | ❌ Yok | ✅ Var |
| Zorunlu/opsiyonel | ❌ Yok | ✅ Var |
| Açıklamalar | ✅ Var | ✅ Var |
| Format | Liste | Tablo |

**Kazanan:** cursor-api.md (daha yapılandırılmış)

---

### Kod Örnekleri

| Özellik | api-kullanim.txt | cursor-api.md |
|---------|------------------|---------------|
| Python örnekleri | ❌ Yok | ✅ Var |
| Çalışan kod | ❌ Yok | ✅ Var |
| Farklı senaryolar | ❌ Yok | ✅ Var |
| Helper class | ❌ Yok | ✅ Var |

**Kazanan:** cursor-api.md (tamamen üstün)

---

### Response Örnekleri

| Özellik | api-kullanim.txt | cursor-api.md |
|---------|------------------|---------------|
| JSON örnekleri | ❌ Yok | ✅ Var |
| 200 OK örnekleri | ❌ Yok | ✅ Var |
| Response yapısı | ❌ Yok | ✅ Var |

**Kazanan:** cursor-api.md (tamamen üstün)

---

### HTTP Status Kodları

| Özellik | api-kullanim.txt | cursor-api.md |
|---------|------------------|---------------|
| Status kodları | ❌ Yok | ✅ Var |
| Hata açıklamaları | ❌ Yok | ✅ Var |
| Hata yönetimi | ❌ Yok | ✅ Var |

**Kazanan:** cursor-api.md (tamamen üstün)

---

### Güncelleme Sıklığı

| Özellik | api-kullanim.txt | cursor-api.md |
|---------|------------------|---------------|
| Güncelleme bilgisi | ✅ Var (her endpoint için) | ⚠️ Bazı endpoint'lerde var |
| Önerilen çağrı sıklığı | ✅ Var | ⚠️ Bazı endpoint'lerde var |

**Kazanan:** api-kullanim.txt (daha tutarlı)

---

## 🎯 Sonuç ve Değerlendirme

### Endpoint Kapsamı

**cursor-api.md kazanıyor:**
- ✅ 36 endpoint (api-kullanim.txt'de 31)
- ✅ 8 ekstra endpoint var
- ✅ Tüm api-kullanim.txt endpoint'leri içeriyor

### Teknik Detay

**cursor-api.md kazanıyor:**
- ✅ Parametre tipleri ve zorunluluk bilgisi
- ✅ Python kod örnekleri
- ✅ Response örnekleri
- ✅ HTTP status kodları
- ✅ Yapılandırılmış format

### Güncelleme Bilgileri

**api-kullanim.txt kazanıyor:**
- ✅ Her endpoint için güncelleme sıklığı
- ✅ Önerilen çağrı sıklığı bilgisi
- ✅ Daha tutarlı

---

## 💡 Öneriler

1. **cursor-api.md'ye eklenebilir:**
   - Her endpoint için güncelleme sıklığı bilgisi
   - Önerilen çağrı sıklığı bilgisi

2. **api-kullanim.txt'ye eklenebilir:**
   - Eksik 8 endpoint
   - Python kod örnekleri
   - Response örnekleri
   - HTTP status kodları
   - Parametre tipleri

3. **İdeal durum:**
   - cursor-api.md formatını koru
   - api-kullanim.txt'deki güncelleme bilgilerini ekle
   - Her iki dosyanın güçlü yönlerini birleştir

---

## 📝 Özet

| Kriter | Kazanan | Fark |
|--------|---------|------|
| **Endpoint Sayısı** | cursor-api.md | +5 endpoint |
| **Teknik Detay** | cursor-api.md | Çok daha detaylı |
| **Kod Örnekleri** | cursor-api.md | Tamamen üstün |
| **Güncelleme Bilgileri** | api-kullanim.txt | Daha tutarlı |

**Genel Sonuç:** cursor-api.md endpoint açısından **daha kapsamlı ve teknik olarak daha detaylı**.

---

**Rapor Tarihi:** 2024-12-17


