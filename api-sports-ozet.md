# API-Football v3 - Endpoint Özeti (Mühendislik Görevi İçin)

## 📋 Genel Bilgiler

**Base URL:** `https://v3.football.api-sports.io`  
**Metod:** Tüm endpoint'ler GET  
**Auth:** Header: `x-rapidapi-key: YOUR_API_KEY`  
**Response Format:** `{get, parameters, errors, results, paging, response}`  
**Status Kodları:** 200 (OK), 204 (No Content), 499 (Timeout), 500 (Error)

---

## 🎯 Endpoint Kategorileri ve Kullanım Amaçları

### 1. Temel Referans Verileri (Statik/Seyrek Güncellenen)

#### `/timezone`
- **Ne yapar:** Tüm timezone listesi (425 adet)
- **Parametre:** Yok
- **Kullanım:** Tarih/saat filtreleme için referans
- **Güncelleme:** Statik
- **DB:** `timezones` tablosu (cache)

#### `/countries`
- **Ne yapar:** Ülke listesi (isim, kod, bayrak)
- **Parametreler:** `name`, `code`, `search` (opsiyonel)
- **Kullanım:** Ülke bazlı filtreleme
- **Güncelleme:** Statik
- **DB:** `countries` tablosu (cache)

#### `/leagues/seasons`
- **Ne yapar:** Mevcut sezon yılları listesi
- **Parametre:** Yok
- **Kullanım:** Sezon seçimi için referans
- **Güncelleme:** Yılda 1 kez
- **DB:** `seasons` tablosu (cache)

---

### 2. Lig ve Takım Bilgileri

#### `/leagues`
- **Ne yapar:** Lig/kupa listesi, coverage bilgisi
- **Parametreler:** `id`, `name`, `country`, `code`, `season`, `team`, `type`, `current`, `search`, `last`
- **Kullanım:** Lig seçimi, coverage kontrolü (hangi veriler mevcut)
- **Güncelleme:** Günde birkaç kez
- **Cronjob:** Saatte 1
- **DB:** `leagues` tablosu (league_id PK, coverage JSON)

#### `/teams`
- **Ne yapar:** Takım bilgileri (isim, logo, stadyum)
- **Parametreler:** `id`, `name`, `league`, `season`, `country`, `code`, `venue`, `search`
- **Kullanım:** Takım seçimi, takım bilgisi
- **Güncelleme:** Günde birkaç kez
- **Cronjob:** Saatte 1
- **DB:** `teams` tablosu (team_id PK, venue_id FK)

#### `/teams/statistics`
- **Ne yapar:** Takım istatistikleri (form, gol, asist, vb.)
- **Parametreler:** `league`, `season`, `team`, `date` (opsiyonel)
- **Kullanım:** Takım performans analizi, form snapshot
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `team_statistics` tablosu (team_id, league_id, season, stats JSON)

#### `/teams/seasons`
- **Ne yapar:** Takımın oynadığı sezonlar
- **Parametreler:** `team` (zorunlu)
- **Kullanım:** Takım geçmişi
- **DB:** `team_seasons` tablosu

#### `/teams/countries`
- **Ne yapar:** Ülkeye göre takım listesi
- **Parametreler:** `country` (opsiyonel)
- **Kullanım:** Ülke bazlı takım listesi
- **DB:** `teams` tablosundan filtreleme

#### `/venues`
- **Ne yapar:** Stadyum bilgileri
- **Parametreler:** `id`, `name`, `city`, `country`, `search`
- **Kullanım:** Stadyum bilgisi, kapasite
- **Güncelleme:** Seyrek
- **DB:** `venues` tablosu (venue_id PK)

---

### 3. Puan Durumu ve Sıralamalar

#### `/standings`
- **Ne yapar:** Lig puan durumu
- **Parametreler:** `league` (zorunlu), `season` (zorunlu), `team` (opsiyonel)
- **Kullanım:** Puan tablosu, sıralama, form analizi
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `standings` tablosu (league_id, season, team_id, rank, points, form JSON)

---

### 4. Maç Bilgileri (Fixtures)

#### `/fixtures`
- **Ne yapar:** Maç listesi, skorlar, durumlar
- **Parametreler:** `id`, `live`, `date`, `league`, `season`, `team`, `last`, `next`, `from`, `to`, `round`, `status`, `venue`, `timezone`
- **Kullanım:** Fikstür, geçmiş maçlar, gelecek maçlar, canlı maçlar
- **Güncelleme:** Canlı maçlar için sürekli, diğerleri günlük
- **Cronjob:** Canlı maçlar için 30 saniyede 1, diğerleri günde 1
- **DB:** `fixtures` tablosu (fixture_id PK, home_team_id, away_team_id, date, status, score JSON)

#### `/fixtures/rounds`
- **Ne yapar:** Lig turları/haftaları
- **Parametreler:** `league` (zorunlu), `season` (zorunlu), `current` (opsiyonel)
- **Kullanım:** Hafta bazlı filtreleme
- **Güncelleme:** Sezon başında
- **DB:** `rounds` tablosu (league_id, season, round_name)

#### `/fixtures/headtohead`
- **Ne yapar:** İki takım arası geçmiş maçlar
- **Parametreler:** `h2h` (zorunlu, format: "team1_id-team2_id"), `date`, `league`, `season`, `last`
- **Kullanım:** Takımlar arası geçmiş analizi
- **Güncelleme:** Yeni maç sonrası
- **DB:** `fixtures` tablosundan JOIN ile

#### `/fixtures/statistics`
- **Ne yapar:** Maç istatistikleri (şut, pas, top kontrolü)
- **Parametreler:** `fixture` (zorunlu), `team` (opsiyonel), `type` (opsiyonel)
- **Kullanım:** Maç analizi, performans metrikleri
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `fixture_statistics` tablosu (fixture_id, team_id, stats JSON)

#### `/fixtures/events`
- **Ne yapar:** Maç olayları (gol, kart, değişiklik, VAR)
- **Parametreler:** `fixture` (zorunlu), `team`, `player`, `type`
- **Kullanım:** Maç detayları, olay zaman çizelgesi
- **Güncelleme:** Canlı maçlarda sürekli
- **Cronjob:** Canlı maçlar için 15 saniyede 1
- **DB:** `fixture_events` tablosu (fixture_id, minute, type, player_id, team_id)

#### `/fixtures/lineups`
- **Ne yapar:** Maç kadroları (11'ler, yedekler, teknik direktör)
- **Parametreler:** `fixture` (zorunlu), `team` (opsiyonel)
- **Kullanım:** Kadro analizi, formasyon
- **Güncelleme:** Maç öncesi ve sırasında
- **Cronjob:** Maç öncesi 1 saat, maç sırasında 15 dakikada 1
- **DB:** `fixture_lineups` tablosu (fixture_id, team_id, formation, players JSON)

#### `/fixtures/players`
- **Ne yapar:** Maçtaki oyuncu istatistikleri
- **Parametreler:** `fixture` (zorunlu), `team`, `player`
- **Kullanım:** Oyuncu performans analizi
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `fixture_player_stats` tablosu (fixture_id, player_id, team_id, stats JSON)

---

### 5. Oyuncu Bilgileri

#### `/players`
- **Ne yapar:** Oyuncu listesi ve bilgileri
- **Parametreler:** `id`, `team`, `league`, `season`, `search`, `page`
- **Kullanım:** Oyuncu arama, oyuncu bilgisi
- **Güncelleme:** Günlük
- **Cronjob:** Günde 1
- **DB:** `players` tablosu (player_id PK, team_id FK)

#### `/players/seasons`
- **Ne yapar:** Oyuncunun oynadığı sezonlar
- **Parametreler:** `player` (zorunlu)
- **Kullanım:** Oyuncu kariyer geçmişi
- **DB:** `player_seasons` tablosu

#### `/players/profiles`
- **Ne yapar:** Oyuncu profilleri (biyografi, fiziksel özellikler)
- **Parametreler:** `id`, `team`, `search`, `page`
- **Kullanım:** Oyuncu detay bilgisi
- **Güncelleme:** Haftada birkaç kez
- **Cronjob:** Haftada 1
- **DB:** `player_profiles` tablosu (player_id PK, profile JSON)

#### `/players/statistics`
- **Ne yapar:** Oyuncu istatistikleri (gol, asist, rating)
- **Parametreler:** `id`, `team`, `league`, `season`, `search`, `page`
- **Kullanım:** Oyuncu performans analizi
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `player_statistics` tablosu (player_id, team_id, league_id, season, stats JSON)

#### `/players/squads`
- **Ne yapar:** Takım kadrosu
- **Parametreler:** `team` (zorunlu)
- **Kullanım:** Takım kadro listesi
- **Güncelleme:** Transfer dönemlerinde
- **Cronjob:** Haftada 1
- **DB:** `team_squads` tablosu (team_id, player_id, position)

#### `/players/teams`
- **Ne yapar:** Oyuncunun kariyerindeki takımlar
- **Parametreler:** `player` (zorunlu)
- **Kullanım:** Transfer geçmişi
- **DB:** `player_teams` tablosu

#### `/players/topscorers`
- **Ne yapar:** En çok gol atan oyuncular (top 20)
- **Parametreler:** `league` (zorunlu), `season` (zorunlu)
- **Kullanım:** Gol krallığı listesi
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `top_scorers` view veya cache tablosu

#### `/players/topassists`
- **Ne yapar:** En çok asist yapan oyuncular (top 20)
- **Parametreler:** `league` (zorunlu), `season` (zorunlu)
- **Kullanım:** Asist liderliği
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `top_assists` view veya cache tablosu

#### `/players/topyellowcards`
- **Ne yapar:** En çok sarı kart alan oyuncular
- **Parametreler:** `league` (zorunlu), `season` (zorunlu)
- **Kullanım:** Disiplin analizi
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `top_yellow_cards` view

#### `/players/topredcards`
- **Ne yapar:** En çok kırmızı kart alan oyuncular
- **Parametreler:** `league` (zorunlu), `season` (zorunlu)
- **Kullanım:** Disiplin analizi
- **Güncelleme:** Maç sonrası
- **Cronjob:** Maç bitişinden sonra
- **DB:** `top_red_cards` view

---

### 6. Sakatlık ve Durum Bilgileri

#### `/injuries`
- **Ne yapar:** Sakatlık ve ceza bilgileri
- **Parametreler:** `fixture`, `team`, `league`, `season`, `player`, `date`
- **Kullanım:** Oyuncu durumu, kadro tahmini
- **Güncelleme:** 4 saatte bir
- **Cronjob:** 4 saatte 1
- **DB:** `injuries` tablosu (player_id, fixture_id, status, reason)

#### `/sidelined`
- **Ne yapar:** Kadro dışı oyuncular/teknik direktörler
- **Parametreler:** `player`, `coach`, `team`
- **Kullanım:** Eksik oyuncu takibi
- **Güncelleme:** Günlük
- **Cronjob:** Günde 1
- **DB:** `sidelined` tablosu (player_id/coach_id, reason, start_date, end_date)

---

### 7. Tahmin ve Analiz

#### `/predictions`
- **Ne yapar:** Maç tahminleri (sonuç, gol, tavsiye)
- **Parametreler:** `fixture` (zorunlu)
- **Kullanım:** AI tahmin desteği
- **Güncelleme:** Saatlik
- **Cronjob:** Saatte 1 (maç öncesi)
- **DB:** `predictions` tablosu (fixture_id, prediction JSON)

---

### 8. Teknik Direktör ve Transfer

#### `/coaches`
- **Ne yapar:** Teknik direktör bilgileri
- **Parametreler:** `id`, `team`, `search`
- **Kullanım:** Teknik direktör bilgisi
- **Güncelleme:** Seyrek
- **DB:** `coaches` tablosu (coach_id PK, team_id FK)

#### `/transfers`
- **Ne yapar:** Transfer bilgileri
- **Parametreler:** `player`, `team`
- **Kullanım:** Transfer geçmişi
- **Güncelleme:** Transfer dönemlerinde
- **Cronjob:** Transfer dönemlerinde günlük
- **DB:** `transfers` tablosu (player_id, from_team_id, to_team_id, date, type)

#### `/trophies`
- **Ne yapar:** Kupa/trofe bilgileri
- **Parametreler:** `player`, `coach`
- **Kullanım:** Başarı geçmişi
- **Güncelleme:** Seyrek
- **DB:** `trophies` tablosu (player_id/coach_id, trophy JSON)

---

### 9. Bahis Oranları (Odds)

#### `/odds`
- **Ne yapar:** Maç öncesi bahis oranları
- **Parametreler:** `fixture`, `league`, `season`, `date`, `timezone`, `page`, `bookmaker`, `bet`
- **Kullanım:** Bahis analizi, oran takibi
- **Güncelleme:** 3 saatte bir
- **Cronjob:** 3 saatte 1
- **DB:** `odds_prematch` tablosu (fixture_id, bookmaker_id, bet_type_id, odds JSON, timestamp)

#### `/odds/live`
- **Ne yapar:** Canlı maç bahis oranları
- **Parametreler:** `fixture` (opsiyonel)
- **Kullanım:** Canlı bahis analizi
- **Güncelleme:** 5-60 saniye arası
- **Cronjob:** Canlı maçlar için 30 saniyede 1
- **DB:** `odds_live` tablosu (fixture_id, bookmaker_id, bet_type_id, odds JSON, timestamp)

#### `/odds/live/bets`
- **Ne yapar:** Canlı bahis türleri listesi
- **Parametre:** Yok
- **Kullanım:** Canlı bahis türü referansı
- **Güncelleme:** 60 saniyede bir
- **DB:** `bet_types_live` tablosu (cache)

#### `/odds/bets`
- **Ne yapar:** Maç öncesi bahis türleri listesi
- **Parametreler:** `id`, `search`
- **Kullanım:** Bahis türü referansı
- **Güncelleme:** Haftada birkaç kez
- **Cronjob:** Günde 1
- **DB:** `bet_types_prematch` tablosu (cache)

#### `/odds/bookmakers`
- **Ne yapar:** Bahis şirketleri listesi
- **Parametreler:** `id`, `search`
- **Kullanım:** Bahis şirketi referansı
- **Güncelleme:** Birkaç günde bir
- **Cronjob:** Günde 1
- **DB:** `bookmakers` tablosu (cache)

#### `/odds/mapping`
- **Ne yapar:** Oranlar için kullanılabilir maç ID'leri
- **Parametreler:** `page` (opsiyonel)
- **Kullanım:** Oran sorguları için fixture ID bulma
- **Güncelleme:** Günlük
- **Cronjob:** Günde 1
- **DB:** `odds_mapping` tablosu (fixture_id, league_id, date)

---

## 🗄️ Önerilen DB Yapısı (Özet)

### Ana Tablolar

1. **`fixtures`** - Maçlar (fixture_id PK, home_team_id, away_team_id, league_id, date, status, score)
2. **`teams`** - Takımlar (team_id PK, name, country, venue_id)
3. **`players`** - Oyuncular (player_id PK, name, team_id)
4. **`leagues`** - Ligler (league_id PK, name, country, coverage JSON)
5. **`standings`** - Puan durumu (league_id, season, team_id, rank, points)
6. **`odds_prematch`** - Ön maç oranları (fixture_id, bookmaker_id, bet_type_id, odds JSON, timestamp)
7. **`odds_live`** - Canlı oranlar (fixture_id, bookmaker_id, bet_type_id, odds JSON, timestamp)
8. **`fixture_events`** - Maç olayları (fixture_id, minute, type, player_id)
9. **`fixture_statistics`** - Maç istatistikleri (fixture_id, team_id, stats JSON)
10. **`player_statistics`** - Oyuncu istatistikleri (player_id, team_id, league_id, season, stats JSON)
11. **`injuries`** - Sakatlıklar (player_id, fixture_id, status)
12. **`predictions`** - Tahminler (fixture_id, prediction JSON)

### Cache Tabloları (Seyrek Güncellenen)

- `countries`, `timezones`, `seasons`, `bookmakers`, `bet_types_prematch`, `bet_types_live`, `venues`, `coaches`

---

## ⏰ Cronjob Önerileri

### Yüksek Frekans (Canlı Maçlar)
- **30 saniyede 1:** `/fixtures` (live), `/odds/live`, `/fixtures/events` (canlı maçlar)
- **15 dakikada 1:** `/fixtures/lineups` (maç sırasında)

### Orta Frekans
- **Saatte 1:** `/leagues`, `/teams`, `/fixtures` (geçmiş/gelecek)
- **3 saatte 1:** `/odds` (prematch)
- **4 saatte 1:** `/injuries`

### Düşük Frekans
- **Günde 1:** `/standings`, `/players`, `/odds/bookmakers`, `/odds/bets`, `/odds/mapping`, `/sidelined`
- **Haftada 1:** `/players/profiles`, `/players/squads`
- **Yılda 1:** `/leagues/seasons`

### Event-Based (Maç Sonrası)
- Maç bitişinden sonra: `/fixtures/statistics`, `/fixtures/players`, `/players/statistics`, `/standings`, `/players/topscorers`, `/players/topassists`

---

## 🎯 Endpoint Kullanım Senaryoları

### Senaryo 1: Günlük Fikstür ve Oranlar
1. `/fixtures` (date=today) → Bugünün maçları
2. `/odds` (fixture=...) → Her maç için oranlar
3. `/predictions` (fixture=...) → Tahminler
4. `/injuries` (fixture=...) → Sakatlıklar

### Senaryo 2: Canlı Maç Takibi
1. `/fixtures` (live=all) → Canlı maçlar
2. `/odds/live` (fixture=...) → Canlı oranlar
3. `/fixtures/events` (fixture=...) → Olaylar
4. `/fixtures/statistics` (fixture=...) → İstatistikler

### Senaryo 3: Takım Analizi
1. `/teams` (id=...) → Takım bilgisi
2. `/teams/statistics` (team=...) → İstatistikler
3. `/standings` (team=...) → Puan durumu
4. `/fixtures/headtohead` (h2h=...) → Geçmiş maçlar

### Senaryo 4: Oyuncu Analizi
1. `/players` (id=...) → Oyuncu bilgisi
2. `/players/statistics` (player=...) → İstatistikler
3. `/injuries` (player=...) → Sakatlık durumu
4. `/transfers` (player=...) → Transfer geçmişi

---

## 📊 Önemli Notlar

- **Rate Limiting:** API key planına göre günlük/dakikalık limitler var
- **Pagination:** Bazı endpoint'lerde `page` parametresi gerekli
- **Batch Limits:** `/fixtures` endpoint'i maksimum 20 fixture ID kabul eder
- **Coverage:** `/leagues` endpoint'inden hangi verilerin mevcut olduğunu kontrol edin
- **Timezone:** Tarih filtreleri için `/timezone` kullanın
- **Cache Stratejisi:** Statik veriler (countries, timezones, bookmakers) cache'lenmeli

---

**Toplam Endpoint:** 36  
**Kritik Endpoint'ler:** `/fixtures`, `/odds`, `/standings`, `/players/statistics`, `/teams/statistics`  
**Yüksek Frekans Endpoint'ler:** `/fixtures` (live), `/odds/live`, `/fixtures/events`

