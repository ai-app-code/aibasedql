# API-Football v3 - Tüm Endpoint Dokümantasyonu

Bu dokümantasyon, API-Football v3 API'sinin tüm endpoint'lerini, parametrelerini, response formatlarını ve Python örneklerini içerir.

## 📋 İçindekiler

1. [Genel Bilgiler](#genel-bilgiler)
2. [Authentication (Kimlik Doğrulama)](#authentication)
3. [Endpoint'ler](#endpointler)
   - [Timezone](#1-timezone)
   - [Countries](#2-countries)
   - [Leagues](#3-leagues)
   - [Seasons](#4-seasons)
   - [Teams](#5-teams)
   - [Venues](#6-venues)
   - [Standings](#7-standings)
   - [Fixtures](#8-fixtures)
   - [Injuries](#9-injuries)
   - [Predictions](#10-predictions)
   - [Coaches](#11-coaches)
   - [Players](#12-players)
   - [Transfers](#13-transfers)
   - [Trophies](#14-trophies)
   - [Sidelined](#15-sidelined)
   - [Odds](#16-odds)

---

## Genel Bilgiler

### Base URL
```
https://v3.football.api-sports.io
```

### HTTP Metodu
Tüm endpoint'ler **GET** metodu kullanır.

### Rate Limiting
- **X-RateLimit-Limit**: Dakika başına maksimum API çağrısı
- **X-RateLimit-Remaining**: Dakika başına kalan API çağrısı
- **x-ratelimit-requests-limit**: Günlük tahsis edilen istek sayısı
- **x-ratelimit-requests-remaining**: Günlük kalan istek sayısı

### Response Formatı
Tüm endpoint'ler aynı response formatını kullanır:

```json
{
  "get": "endpoint_name",
  "parameters": {},
  "errors": [],
  "results": 0,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": []
}
```

### HTTP Status Kodları

| Kod | Açıklama |
|-----|----------|
| **200** | Başarılı istek |
| **204** | İçerik yok (No Content) |
| **499** | Zaman aşımı (Time Out) |
| **500** | Sunucu hatası (Internal Server Error) |

---

## Authentication

### Header
Tüm isteklerde aşağıdaki header kullanılmalıdır:

```python
headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}
```

**Önemli Notlar:**
- API sadece GET isteklerini kabul eder
- Sadece `x-rapidapi-key` header'ına izin verilir
- Bazı framework'ler (özellikle JS, NodeJS) otomatik olarak ekstra header'lar ekler, bunları kaldırmanız gerekir

---

## Endpoint'ler

### 1. Timezone

#### Endpoint
```
GET /timezone
```

#### Açıklama
Tüm mevcut timezone'ları listeler.

#### Parametreler
Bu endpoint parametre almaz.

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/timezone"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

response = requests.get(url, headers=headers)
data = response.json()

print(data)
```

#### Response Örneği (200 OK)

```json
{
  "get": "timezone",
  "parameters": [],
  "errors": [],
  "results": 425,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [
    "Africa/Abidjan",
    "Africa/Accra",
    "Africa/Addis_Ababa",
    "Africa/Algiers",
    "Africa/Asmara"
  ]
}
```

#### Response Kodları
- **200 OK**: Başarılı, timezone listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 2. Countries

#### Endpoint
```
GET /countries
```

#### Açıklama
Ülkeleri listeler. İsim veya kod ile filtreleme yapılabilir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `name` | string | Hayır | Ülke adı (örn: "england") |
| `code` | string | Hayır | Ülke kodu (2-6 karakter, örn: "GB", "FR") |
| `search` | string | Hayır | Arama terimi (min 3 karakter) |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/countries"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Tüm ülkeleri getir
response = requests.get(url, headers=headers)

# İsme göre filtrele
params = {"name": "england"}
response = requests.get(url, headers=headers, params=params)

# Koda göre filtrele
params = {"code": "GB"}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Örneği (200 OK)

```json
{
  "get": "countries",
  "parameters": {
    "name": "england"
  },
  "errors": [],
  "results": 1,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [
    {
      "name": "England",
      "code": "GB",
      "flag": "https://media.api-sports.io/flags/gb.svg"
    }
  ]
}
```

#### Response Kodları
- **200 OK**: Başarılı, ülke listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 3. Leagues

#### Endpoint
```
GET /leagues
```

#### Açıklama
Mevcut lig ve kupa listesini getirir. Lig ID'leri API'de benzersizdir ve tüm sezonlarda aynı kalır.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `id` | integer | Hayır | Lig ID'si |
| `name` | string | Hayır | Lig adı |
| `country` | string | Hayır | Ülke adı |
| `code` | string | Hayır | Ülke kodu (2-6 karakter) |
| `season` | integer | Hayır | Sezon yılı (YYYY formatında) |
| `team` | integer | Hayır | Takım ID'si |
| `type` | string | Hayır | Lig tipi: "league" veya "cup" |
| `current` | string | Hayır | Aktif sezonlar için: "true" veya "false" |
| `search` | string | Hayır | Arama terimi (min 3 karakter) |
| `last` | integer | Hayır | Son eklenen X lig/kupa (max 2) |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/leagues"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Belirli bir ligi getir
params = {"id": 39}  # Premier League
response = requests.get(url, headers=headers, params=params)

# Ülkeye göre filtrele
params = {"country": "England"}
response = requests.get(url, headers=headers, params=params)

# Sezon ve ülkeye göre filtrele
params = {
    "country": "England",
    "season": 2023
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Örneği (200 OK)

```json
{
  "get": "leagues",
  "parameters": {
    "id": "39"
  },
  "errors": [],
  "results": 1,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [
    {
      "league": {
        "id": 39,
        "name": "Premier League",
        "type": "League",
        "logo": "https://media.api-sports.io/football/leagues/39.png"
      },
      "country": {
        "name": "England",
        "code": "GB",
        "flag": "https://media.api-sports.io/flags/gb.svg"
      },
      "seasons": [
        {
          "year": 2023,
          "start": "2023-08-11",
          "end": "2024-05-19",
          "current": true,
          "coverage": {
            "fixtures": {
              "events": true,
              "lineups": true,
              "statistics_fixtures": true,
              "statistics_players": true
            },
            "standings": true,
            "players": true,
            "top_scorers": true,
            "top_assists": true,
            "top_cards": true,
            "injuries": true,
            "predictions": true,
            "odds": true
          }
        }
      ]
    }
  ]
}
```

#### Response Kodları
- **200 OK**: Başarılı, lig listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

**Notlar:**
- Lig logoları için: `https://media.api-sports.io/football/leagues/{league_id}.png`
- Coverage değerleri, o an için mevcut verileri gösterir
- Güncelleme sıklığı: Günde birkaç kez
- Önerilen çağrı sıklığı: Saatte 1 kez

---

### 4. Seasons

#### Endpoint
```
GET /leagues/seasons
```

#### Açıklama
Tüm mevcut sezon yıllarını listeler.

#### Parametreler
Bu endpoint parametre almaz.

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/leagues/seasons"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

response = requests.get(url, headers=headers)
data = response.json()

print(data)
```

#### Response Örneği (200 OK)

```json
{
  "get": "leagues/seasons",
  "parameters": [],
  "errors": [],
  "results": 16,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [
    2008,
    2010,
    2011,
    2012,
    2013,
    2014,
    2015,
    2016,
    2017,
    2018,
    2019,
    2020,
    2021,
    2022,
    2023,
    2024
  ]
}
```

#### Response Kodları
- **200 OK**: Başarılı, sezon listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 5. Teams

#### Endpoint
```
GET /teams
```

#### Açıklama
Takım bilgilerini getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `id` | integer | Hayır | Takım ID'si |
| `name` | string | Hayır | Takım adı |
| `league` | integer | Hayır | Lig ID'si |
| `season` | integer | Hayır | Sezon yılı (YYYY) |
| `country` | string | Hayır | Ülke adı |
| `code` | string | Hayır | Ülke kodu (2-6 karakter) |
| `venue` | integer | Hayır | Stadyum ID'si |
| `search` | string | Hayır | Arama terimi (min 3 karakter) |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/teams"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Belirli bir takımı getir
params = {"id": 33}  # Manchester United
response = requests.get(url, headers=headers, params=params)

# Lig ve sezona göre filtrele
params = {
    "league": 39,
    "season": 2023
}
response = requests.get(url, headers=headers, params=params)

# İsme göre arama
params = {"search": "manchester"}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Örneği (200 OK)

```json
{
  "get": "teams",
  "parameters": {
    "id": "33"
  },
  "errors": [],
  "results": 1,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [
    {
      "team": {
        "id": 33,
        "name": "Manchester United",
        "code": "MUN",
        "country": "England",
        "founded": 1878,
        "national": false,
        "logo": "https://media.api-sports.io/football/teams/33.png"
      },
      "venue": {
        "id": 11,
        "name": "Old Trafford",
        "address": "Sir Matt Busby Way",
        "city": "Manchester",
        "capacity": 74879,
        "surface": "grass",
        "image": "https://media.api-sports.io/football/venues/11.png"
      }
    }
  ]
}
```

#### Response Kodları
- **200 OK**: Başarılı, takım listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 6. Venues

#### Endpoint
```
GET /venues
```

#### Açıklama
Stadyum bilgilerini getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `id` | integer | Hayır | Stadyum ID'si |
| `name` | string | Hayır | Stadyum adı |
| `city` | string | Hayır | Şehir adı |
| `country` | string | Hayır | Ülke adı |
| `search` | string | Hayır | Arama terimi (min 3 karakter) |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/venues"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Belirli bir stadyumu getir
params = {"id": 11}  # Old Trafford
response = requests.get(url, headers=headers, params=params)

# Şehre göre filtrele
params = {"city": "Manchester"}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Örneği (200 OK)

```json
{
  "get": "venues",
  "parameters": {
    "id": "11"
  },
  "errors": [],
  "results": 1,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [
    {
      "id": 11,
      "name": "Old Trafford",
      "address": "Sir Matt Busby Way",
      "city": "Manchester",
      "country": "England",
      "capacity": 74879,
      "surface": "grass",
      "image": "https://media.api-sports.io/football/venues/11.png"
    }
  ]
}
```

#### Response Kodları
- **200 OK**: Başarılı, stadyum listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 7. Standings

#### Endpoint
```
GET /standings
```

#### Açıklama
Lig veya takım için puan durumunu getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `league` | integer | Evet | Lig ID'si |
| `season` | integer | Evet | Sezon yılı (YYYY) |
| `team` | integer | Hayır | Takım ID'si |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/standings"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Lig puan durumu
params = {
    "league": 39,
    "season": 2023
}
response = requests.get(url, headers=headers, params=params)

# Belirli bir takımın durumu
params = {
    "league": 39,
    "season": 2023,
    "team": 33
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Örneği (200 OK)

```json
{
  "get": "standings",
  "parameters": {
    "league": "39",
    "season": "2023"
  },
  "errors": [],
  "results": 1,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [
    {
      "league": {
        "id": 39,
        "name": "Premier League",
        "country": "England",
        "logo": "https://media.api-sports.io/football/leagues/39.png",
        "flag": "https://media.api-sports.io/flags/gb.svg",
        "season": 2023,
        "standings": [
          [
            {
              "rank": 1,
              "team": {
                "id": 50,
                "name": "Manchester City",
                "logo": "https://media.api-sports.io/football/teams/50.png"
              },
              "points": 89,
              "goalsDiff": 61,
              "group": "Premier League",
              "form": "WWWWW",
              "status": "same",
              "description": "Promotion - Champions League (Group Stage: )",
              "all": {
                "played": 38,
                "win": 28,
                "draw": 5,
                "lose": 5,
                "goals": {
                  "for": 94,
                  "against": 33
                }
              },
              "home": {
                "played": 19,
                "win": 17,
                "draw": 1,
                "lose": 1,
                "goals": {
                  "for": 60,
                  "against": 17
                }
              },
              "away": {
                "played": 19,
                "win": 11,
                "draw": 4,
                "lose": 4,
                "goals": {
                  "for": 34,
                  "against": 16
                }
              },
              "update": "2024-05-19T00:00:00+00:00"
            }
          ]
        ]
      }
    }
  ]
}
```

#### Response Kodları
- **200 OK**: Başarılı, puan durumu döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 8. Fixtures

#### 8.1. Get Fixtures

##### Endpoint
```
GET /fixtures
```

##### Açıklama
Maç bilgilerini getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `id` | integer | Hayır | Maç ID'si |
| `live` | string | Hayır | Canlı maçlar: "all" veya "id-id-id" |
| `date` | string | Hayır | Tarih (YYYY-MM-DD) |
| `league` | integer | Hayır | Lig ID'si |
| `season` | integer | Hayır | Sezon yılı (YYYY) |
| `team` | integer | Hayır | Takım ID'si |
| `last` | integer | Hayır | Son X maç (max 99) |
| `next` | integer | Hayır | Sonraki X maç (max 99) |
| `from` | string | Hayır | Başlangıç tarihi (YYYY-MM-DD) |
| `to` | string | Hayır | Bitiş tarihi (YYYY-MM-DD) |
| `round` | string | Hayır | Hafta/tur |
| `status` | string | Hayır | Maç durumu (FT, NS, LIVE, etc.) |
| `venue` | integer | Hayır | Stadyum ID'si |
| `timezone` | string | Hayır | Timezone (örn: "Europe/London") |

##### Python Örneği

```python
import requests
from datetime import datetime, timedelta

url = "https://v3.football.api-sports.io/fixtures"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Belirli bir maç
params = {"id": 157201}
response = requests.get(url, headers=headers, params=params)

# Bugünün maçları
today = datetime.now().strftime("%Y-%m-%d")
params = {"date": today}
response = requests.get(url, headers=headers, params=params)

# Canlı maçlar
params = {"live": "all"}
response = requests.get(url, headers=headers, params=params)

# Lig ve sezona göre
params = {
    "league": 39,
    "season": 2023
}
response = requests.get(url, headers=headers, params=params)

# Tarih aralığı
params = {
    "from": "2023-08-01",
    "to": "2023-08-31"
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

##### Response Örneği (200 OK)

```json
{
  "get": "fixtures",
  "parameters": {
    "id": "157201"
  },
  "errors": [],
  "results": 1,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": [
    {
      "fixture": {
        "id": 157201,
        "referee": "Kevin Friend, England",
        "timezone": "UTC",
        "date": "2019-12-26T17:30:00+00:00",
        "timestamp": 1577381400,
        "periods": {
          "first": 1577381400,
          "second": 1577385000
        },
        "venue": {
          "id": 11,
          "name": "Old Trafford",
          "city": "Manchester"
        },
        "status": {
          "long": "Match Finished",
          "short": "FT",
          "elapsed": 90
        }
      },
      "league": {
        "id": 39,
        "name": "Premier League",
        "country": "England",
        "logo": "https://media.api-sports.io/football/leagues/39.png",
        "flag": "https://media.api-sports.io/flags/gb.svg",
        "season": 2019,
        "round": "Regular Season - 19"
      },
      "teams": {
        "home": {
          "id": 33,
          "name": "Manchester United",
          "logo": "https://media.api-sports.io/football/teams/33.png",
          "winner": true
        },
        "away": {
          "id": 49,
          "name": "Chelsea",
          "logo": "https://media.api-sports.io/football/teams/49.png",
          "winner": false
        }
      },
      "goals": {
        "home": 4,
        "away": 0
      },
      "score": {
        "halftime": {
          "home": 2,
          "away": 0
        },
        "fulltime": {
          "home": 4,
          "away": 0
        },
        "extratime": {
          "home": null,
          "away": null
        },
        "penalty": {
          "home": null,
          "away": null
        }
      }
    }
  ]
}
```

#### 8.2. Get Rounds

##### Endpoint
```
GET /fixtures/rounds
```

##### Açıklama
Lig için tüm hafta/tur bilgilerini getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `league` | integer | Evet | Lig ID'si |
| `season` | integer | Evet | Sezon yılı (YYYY) |
| `current` | string | Hayır | Sadece mevcut hafta: "true" |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/fixtures/rounds"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

params = {
    "league": 39,
    "season": 2023
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### 8.3. Get Head to Head

##### Endpoint
```
GET /fixtures/headtohead
```

##### Açıklama
İki takım arasındaki geçmiş maçları getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `h2h` | string | Evet | Takım ID'leri (örn: "33-49") |
| `date` | string | Hayır | Tarih (YYYY-MM-DD) |
| `league` | integer | Hayır | Lig ID'si |
| `season` | integer | Hayır | Sezon yılı (YYYY) |
| `last` | integer | Hayır | Son X maç (max 10) |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/fixtures/headtohead"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

params = {
    "h2h": "33-49"  # Manchester United vs Chelsea
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### 8.4. Get Statistics

##### Endpoint
```
GET /fixtures/statistics
```

##### Açıklama
Maç istatistiklerini getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `fixture` | integer | Evet | Maç ID'si |
| `team` | integer | Hayır | Takım ID'si (belirli takım için) |
| `type` | string | Hayır | İstatistik tipi |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/fixtures/statistics"

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

#### 8.5. Get Events

##### Endpoint
```
GET /fixtures/events
```

##### Açıklama
Maç olaylarını (goller, kartlar, değişiklikler) getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `fixture` | integer | Evet | Maç ID'si |
| `team` | integer | Hayır | Takım ID'si |
| `player` | integer | Hayır | Oyuncu ID'si |
| `type` | string | Hayır | Olay tipi |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/fixtures/events"

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

#### 8.6. Get Lineups

##### Endpoint
```
GET /fixtures/lineups
```

##### Açıklama
Maç kadrolarını getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `fixture` | integer | Evet | Maç ID'si |
| `team` | integer | Hayır | Takım ID'si |
| `player` | integer | Hayır | Oyuncu ID'si |
| `type` | string | Hayır | Kadro tipi |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/fixtures/lineups"

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

#### 8.7. Get Player Statistics

##### Endpoint
```
GET /fixtures/players
```

##### Açıklama
Maçtaki oyuncu istatistiklerini getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `fixture` | integer | Evet | Maç ID'si |
| `team` | integer | Hayır | Takım ID'si |
| `player` | integer | Hayır | Oyuncu ID'si |
| `type` | string | Hayır | İstatistik tipi |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/fixtures/players"

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

#### Response Kodları (Tüm Fixture Endpoint'leri)
- **200 OK**: Başarılı, veri döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 9. Injuries

#### Endpoint
```
GET /injuries
```

#### Açıklama
Sakatlık bilgilerini getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `fixture` | integer | Hayır | Maç ID'si |
| `team` | integer | Hayır | Takım ID'si |
| `league` | integer | Hayır | Lig ID'si |
| `season` | integer | Hayır | Sezon yılı (YYYY) |
| `player` | integer | Hayır | Oyuncu ID'si |
| `date` | string | Hayır | Tarih (YYYY-MM-DD) |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/injuries"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Belirli bir maç için
params = {"fixture": 157201}
response = requests.get(url, headers=headers, params=params)

# Takım için
params = {"team": 33}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Kodları
- **200 OK**: Başarılı, sakatlık listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 10. Predictions

#### Endpoint
```
GET /predictions
```

#### Açıklama
Maç tahminlerini getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `fixture` | integer | Evet | Maç ID'si |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/predictions"

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

#### Response Kodları
- **200 OK**: Başarılı, tahmin döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 11. Coaches

#### Endpoint
```
GET /coaches
```

#### Açıklama
Teknik direktör bilgilerini getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `id` | integer | Hayır | Teknik direktör ID'si |
| `team` | integer | Hayır | Takım ID'si |
| `search` | string | Hayır | Arama terimi (min 3 karakter) |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/coaches"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Belirli bir teknik direktör
params = {"id": 1}
response = requests.get(url, headers=headers, params=params)

# Takıma göre
params = {"team": 33}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Kodları
- **200 OK**: Başarılı, teknik direktör bilgisi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 12. Players

#### 12.1. Get Players

##### Endpoint
```
GET /players
```

##### Açıklama
Oyuncu bilgilerini getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `id` | integer | Hayır | Oyuncu ID'si |
| `team` | integer | Hayır | Takım ID'si |
| `league` | integer | Hayır | Lig ID'si |
| `season` | integer | Hayır | Sezon yılı (YYYY) |
| `search` | string | Hayır | Arama terimi (min 3 karakter) |
| `page` | integer | Hayır | Sayfa numarası |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/players"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Belirli bir oyuncu
params = {"id": 276}
response = requests.get(url, headers=headers, params=params)

# Takım ve sezona göre
params = {
    "team": 33,
    "season": 2023
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### 12.2. Get Player Seasons

##### Endpoint
```
GET /players/seasons
```

##### Açıklama
Oyuncunun oynadığı sezonları listeler.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `player` | integer | Evet | Oyuncu ID'si |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/players/seasons"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

params = {
    "player": 276
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### 12.3. Get Player Statistics

##### Endpoint
```
GET /players/topscorers
GET /players/topassists
GET /players/topyellowcards
GET /players/topredcards
```

##### Açıklama
Oyuncu istatistiklerini getirir (en çok gol, asist, sarı kart, kırmızı kart).

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `league` | integer | Evet | Lig ID'si |
| `season` | integer | Evet | Sezon yılı (YYYY) |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/players/topscorers"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

params = {
    "league": 39,
    "season": 2023
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### 12.4. Get Player Squad

##### Endpoint
```
GET /players/squads
```

##### Açıklama
Takım kadrosunu getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `team` | integer | Evet | Takım ID'si |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/players/squads"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

params = {
    "team": 33
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Kodları (Tüm Player Endpoint'leri)
- **200 OK**: Başarılı, veri döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 13. Transfers

#### Endpoint
```
GET /transfers
```

#### Açıklama
Transfer bilgilerini getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `player` | integer | Hayır | Oyuncu ID'si |
| `team` | integer | Hayır | Takım ID'si |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/transfers"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Oyuncu için
params = {"player": 276}
response = requests.get(url, headers=headers, params=params)

# Takım için
params = {"team": 33}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Kodları
- **200 OK**: Başarılı, transfer listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 14. Trophies

#### Endpoint
```
GET /trophies
```

#### Açıklama
Kupa/trofe bilgilerini getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `player` | integer | Hayır | Oyuncu ID'si |
| `coach` | integer | Hayır | Teknik direktör ID'si |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/trophies"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Oyuncu için
params = {"player": 276}
response = requests.get(url, headers=headers, params=params)

# Teknik direktör için
params = {"coach": 1}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Kodları
- **200 OK**: Başarılı, kupa listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 15. Sidelined

#### Endpoint
```
GET /sidelined
```

#### Açıklama
Kadro dışı oyuncu/teknik direktör bilgilerini getirir.

#### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `player` | integer | Hayır | Oyuncu ID'si |
| `coach` | integer | Hayır | Teknik direktör ID'si |
| `team` | integer | Hayır | Takım ID'si |

#### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/sidelined"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Oyuncu için
params = {"player": 276}
response = requests.get(url, headers=headers, params=params)

# Takım için
params = {"team": 33}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### Response Kodları
- **200 OK**: Başarılı, kadro dışı listesi döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

### 16. Odds

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

# Belirli bir maç için
params = {"fixture": 157201}
response = requests.get(url, headers=headers, params=params)

# Lig ve sezona göre
params = {
    "league": 39,
    "season": 2023
}
response = requests.get(url, headers=headers, params=params)

data = response.json()
print(data)
```

#### 16.2. Get Odds (Live)

##### Endpoint
```
GET /odds/live
```

##### Açıklama
Canlı maç bahis oranlarını getirir.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `fixture` | integer | Hayır | Maç ID'si |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/odds/live"

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

#### 16.3. Get Bookmakers

##### Endpoint
```
GET /odds/bookmakers
```

##### Açıklama
Bahis şirketlerini listeler.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `id` | integer | Hayır | Bahis şirketi ID'si |
| `search` | string | Hayır | Arama terimi (min 3 karakter) |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/odds/bookmakers"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

response = requests.get(url, headers=headers)
data = response.json()
print(data)
```

#### 16.4. Get Bets

##### Endpoint
```
GET /odds/bets
```

##### Açıklama
Bahis tiplerini listeler.

##### Parametreler

| Parametre | Tip | Zorunlu | Açıklama |
|-----------|-----|---------|----------|
| `id` | integer | Hayır | Bahis tipi ID'si |
| `search` | string | Hayır | Arama terimi (min 3 karakter) |

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/odds/bets"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

response = requests.get(url, headers=headers)
data = response.json()
print(data)
```

#### 16.5. Get Mapping

##### Endpoint
```
GET /odds/mapping
```

##### Açıklama
Bahis mapping bilgilerini getirir.

##### Parametreler
Bu endpoint parametre almaz.

##### Python Örneği

```python
import requests

url = "https://v3.football.api-sports.io/odds/mapping"

headers = {
    "x-rapidapi-key": "YOUR_API_KEY"
}

response = requests.get(url, headers=headers)
data = response.json()
print(data)
```

#### Response Kodları (Tüm Odds Endpoint'leri)
- **200 OK**: Başarılı, bahis oranları döner
- **204 No Content**: Sonuç bulunamadı
- **499 Time Out**: İstek zaman aşımına uğradı
- **500 Internal Server Error**: Sunucu hatası

---

## Hata Yönetimi

### Hata Response Formatı

```json
{
  "get": "endpoint_name",
  "parameters": {},
  "errors": [
    {
      "type": "error_type",
      "message": "Error message"
    }
  ],
  "results": 0,
  "paging": {
    "current": 1,
    "total": 1
  },
  "response": []
}
```

### Yaygın Hatalar

| HTTP Kod | Açıklama | Çözüm |
|----------|----------|-------|
| **400** | Geçersiz parametre | Parametreleri kontrol edin |
| **401** | Geçersiz API key | API key'inizi kontrol edin |
| **403** | Erişim reddedildi | Abonelik planınızı kontrol edin |
| **404** | Endpoint bulunamadı | Endpoint URL'ini kontrol edin |
| **429** | Rate limit aşıldı | İstek sıklığınızı azaltın |
| **500** | Sunucu hatası | Daha sonra tekrar deneyin |

---

## Python Helper Fonksiyonları

### Örnek API Client

```python
import requests
from typing import Dict, Optional, Any

class APIFootballClient:
    def __init__(self, api_key: str):
        self.base_url = "https://v3.football.api-sports.io"
        self.headers = {
            "x-rapidapi-key": api_key
        }
    
    def _make_request(self, endpoint: str, params: Optional[Dict] = None) -> Dict[str, Any]:
        """
        Genel istek fonksiyonu
        """
        url = f"{self.base_url}{endpoint}"
        response = requests.get(url, headers=self.headers, params=params)
        response.raise_for_status()
        return response.json()
    
    def get_timezones(self):
        """Timezone listesini getirir"""
        return self._make_request("/timezone")
    
    def get_countries(self, name: Optional[str] = None, code: Optional[str] = None):
        """Ülke listesini getirir"""
        params = {}
        if name:
            params["name"] = name
        if code:
            params["code"] = code
        return self._make_request("/countries", params)
    
    def get_leagues(self, **kwargs):
        """Lig listesini getirir"""
        return self._make_request("/leagues", params=kwargs)
    
    def get_teams(self, **kwargs):
        """Takım listesini getirir"""
        return self._make_request("/teams", params=kwargs)
    
    def get_fixtures(self, **kwargs):
        """Maç listesini getirir"""
        return self._make_request("/fixtures", params=kwargs)
    
    def get_standings(self, league: int, season: int, team: Optional[int] = None):
        """Puan durumunu getirir"""
        params = {
            "league": league,
            "season": season
        }
        if team:
            params["team"] = team
        return self._make_request("/standings", params=params)
    
    def get_players(self, **kwargs):
        """Oyuncu listesini getirir"""
        return self._make_request("/players", params=kwargs)

# Kullanım örneği
if __name__ == "__main__":
    client = APIFootballClient("YOUR_API_KEY")
    
    # Timezone listesi
    timezones = client.get_timezones()
    print(timezones)
    
    # Premier League takımları
    teams = client.get_teams(league=39, season=2023)
    print(teams)
    
    # Puan durumu
    standings = client.get_standings(league=39, season=2023)
    print(standings)
```

---

## Önemli Notlar

1. **Rate Limiting**: API kullanımınızı rate limit'lere göre yönetin
2. **Caching**: Mümkün olduğunca verileri cache'leyin
3. **Error Handling**: Tüm hata durumlarını handle edin
4. **API Key Güvenliği**: API key'inizi asla public repository'lerde paylaşmayın
5. **Güncelleme Sıklığı**: Endpoint'lerin güncelleme sıklığını dikkate alın
6. **Pagination**: Büyük veri setleri için pagination kullanın

---

## Kaynaklar

- **Resmi Dokümantasyon**: https://www.api-football.com/documentation-v3
- **Dashboard**: https://dashboard.api-football.com
- **API Base URL**: https://v3.football.api-sports.io

---

## Son Güncelleme

Bu dokümantasyon **API-Football v3.9.3** versiyonuna göre hazırlanmıştır.

**Tarih**: 2024-12-17

---

## Lisans

Bu dokümantasyon API-Football API'sinin resmi dokümantasyonundan derlenmiştir. API kullanımı için API-Football'un kullanım şartlarına uymanız gerekmektedir.

