# API-Football v3 Endpoint Özeti

Bu dosya, `cursor-api.md` dokümantasyonundaki tüm API endpoint'lerinin işlevsel özetlerini ve açıklamalarını içerir.

## 1. Sistem ve Konfigürasyon Verileri
*   **`/timezone`**: API'de kullanılabilir olan tüm zaman dilimlerini (timezone) listeler.
*   **`/countries`**: Maçların ve liglerin ait olduğu ülkeleri listeler. İsim veya kod ile filtreme yapılabilir.
*   **`/leagues`**: Mevcut tüm lig ve kupa bilgilerini getirir (ID, logo, tip).
*   **`/leagues/seasons`**: API'de verisi bulunan tüm sezon yıllarını listeler.

## 2. Takım ve Mekan Bilgileri
*   **`/teams`**: Takımların temel bilgilerini (kuruluş yılı, ülke, logo vb.) getirir.
*   **`/venues`**: Maçların oynandığı stadyum ve mekan detaylarını (kapasite, yüzey, şehir vb.) içerir.

## 3. Puan Durumu ve Sıralama
*   **`/standings`**: Liglerin canlı veya sezonluk puan durumlarını, takım formlarını ve istatistiklerini getirir.

## 4. Maç Verileri (Fixtures)
*   **`/fixtures`**: Maç listelerini, skorları ve maç durumlarını (başlamadı, canlı, bitti) getirir.
*   **`/fixtures/rounds`**: Belirli bir ligdeki hafta veya tur (round) bilgilerini döndürür.
*   **`/fixtures/headtohead`**: İki takım arasındaki geçmiş karşılaştırmaları ve ikili rekabet istatistiklerini getirir.
*   **`/fixtures/statistics`**: Maç bazlı detaylı istatistikler (topla oynama, şutlar, kornerler vb.) sağlar.
*   **`/fixtures/events`**: Maç içindeki anlık olayları (goller, kartlar, oyuncu değişiklikleri, VAR) listeler.
*   **`/fixtures/lineups`**: Maç kadrolarını, dizilişleri ve yedek oyuncuları gösterir.
*   **`/fixtures/players`**: Maçta oynayan oyuncuların performans istatistiklerini bireysel bazda getirir.

## 5. Sakatlık ve Tahmin Verileri
*   **`/injuries`**: Maçlardaki sakatlık ve eksik oyuncu bilgilerini tarih bazlı listeler.
*   **`/predictions`**: Maçlar için istatistiksel algoritmalarla (Poisson vb.) üretilmiş kazanma ihtimalleri ve tahmin önerileri sunar.

## 6. Teknik Ekip ve Oyuncu Detayları
*   **`/coaches`**: Teknik direktörlerin kariyer geçmişi ve güncel bilgilerini sağlar.
*   **`/players`**: Oyuncuların genel profillerini ve istatistiklerini getirir.
*   **`/players/seasons`**: Bir oyuncunun kariyeri boyunca verisi bulunan sezonları listeler.
*   **`/players/squads`**: Bir takımın güncel oyuncu kadrosunu listeler.
*   **`/players/topscorers`**: Belirli bir lig/sezon için en çok gol atan oyuncuları listeler.
*   **`/players/topassists`**: Belirli bir lig/sezon için en çok asist yapan oyuncuları listeler.
*   **`/players/topyellowcards`**: En çok sarı kart gören oyuncuları listeler.
*   **`/players/topredcards`**: En çok kırmızı kart gören oyuncuları listeler.

## 7. Transfer ve Başarı Verileri
*   **`/transfers`**: Oyuncu transfer hareketlerini ve tarihçesini listeler.
*   **`/trophies`**: Oyuncu veya teknik direktörlerin kazandığı kupaları ve başarıları listeler.
*   **`/sidelined`**: Disiplin veya diğer nedenlerle kadro dışı kalan oyuncuları listeler.

## 8. Bahis Oranları (Odds)
*   **`/odds`**: Maç öncesi (pre-match) tüm bahis oranlarını ve market bilgilerini getirir.
*   **`/odds/live`**: Canlı devam eden maçlar için anlık değişen bahis oranlarını sağlar.
*   **`/odds/bookmakers`**: Desteklenen bahis şirketlerini (bookmakers) listeler.
*   **`/odds/bets`**: Geçerli olan bahis tiplerini (1X2, Alt/Üst vb.) listeler.
*   **`/odds/mapping`**: Oran sorgulamaları için gerekli olan fixture ID eşleştirmelerini toplu olarak sunar.

---
*Bu özet, API-Football v3 yapısına göre AI modellerine veri beslemek ve mimari tasarlamak için optimize edilmiştir.*
