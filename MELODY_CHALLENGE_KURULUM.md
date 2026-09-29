# Melody Challenge - Eye of the Tiger

## Kurulum Tamamlandı

**Tarih:** 2026-09-29
**Commit:** 8b8842c
**Durum:** ✓ HAZIR (Video dosyası bekleniyor)

### Neler Eklendi

#### 1. HTML Panelleri (index.html, lines 323-449)
- `#gamePromoMelody` - Tanıtım (Eye of the Tiger video + akademi şeritleri)
- `#gameStartMelody` - Ad girişi + ödül listesi
- `#gamePlayingMelody` - 8 nota padi (4x2 grid)
- `#gameWinMelody` - Sonuç + kupon + QR kod

Tüm paneller Rhythm Challenge ile özdeş mimaride yazılmış.

#### 2. CSS Stiller (style tag sonunda, ~15 satır)
```css
.melody-pad { /* Nota düğmesi: daire, sarı, aktif/hata animasyonları */ }
@keyframes melody-scale-pulse { /* Tıklamada 1.0x → 1.2x → 1.15x */ }
@keyframes melody-shake { /* Hatada -2px ↔ +2px sallama */ }
```

#### 3. JavaScript (script tag sonunda, ~270 satır)

**Ödül Sistemi:**
```javascript
melodyPrizes = {
  perfect: "AKD1AY" (Akademi 1 ay 4 ders)
  great:   "AKDRS"  (Ücretsiz ders)
  good:    "AKD50"  (%50 indirim)
  ok:      "CANTA"  (Bez çanta)
  fail:    "" (Hediye yok)
}
```

**Oyun Mantığı:**
- `handleMelodyInput(noteNum)` - Nota girdisi (dokunmatik)
- `playMelodyNote(noteNum)` - Web Audio API piyano synth
- `finishMelodyGame()` - Doğruluk yüzdesine göre ödül tayin ve kupon üretimi
- `makeCode()` ve `qrRender()` - Mevcut fonksiyonlar kullanılıyor

**Entegrasyon:**
- `wShowPanel()` - Melody panellerine uyumlu
- `closeGame()` - Melody videosunu pause ediyor
- `#cardMelodyChallenge` - Hub'da zaten bulunan button

#### 4. Dosya Yapısı
```
games/melody-challenge/
  eye-of-the-tiger.mp4  ← GEREKLI (henüz eklenmemiş)
  (ileride: müzik dosyası vs)
```

### Test Kontrol Listesi

- [ ] **Video Ekleme**
  - Eye of the Tiger (Survivor) video dosyası `games/melody-challenge/eye-of-the-tiger.mp4` konumuna koyun
  - Boyut: 1080×1920 (dikey), 10-15 sn, 5-15 MB
  - Format: MP4 (H.264)

- [ ] **Oyun Akışı**
  - Ana sayfa → Melodi Challenge kartına tıkla
  - Video oynatılıyor mu? (video kodu hazır, dosya gerekli)
  - "← OYUNLAR" butonu hub'a dönüyor mu?
  - "▶ OYNA" butonu start ekranına gidiyor mu?

- [ ] **Ad Girişi**
  - Adın Soyadın girişi yapılıyor mu?
  - Boş ad "Müşteri" oluyor mu?
  - Ödül listesi gösteriliyor mu?

- [ ] **8 Nota Padi**
  - 4×2 grid doğru yerleşiyor mu?
  - Notalar tıklanabiliyor mu (dokunmatik)?
  - Tıklamada nota sesi çalıyor mu? (Web Audio API)
  - Pad aktif (yeşil) / hata (kırmızı) animasyonları çalışıyor mu?

- [ ] **Doğruluk Hesaplaması**
  - Hedef sıra: Do Do Sol Mi Do Do Sol Mi (1 1 5 3 1 1 5 3)
  - 8/8 = %100 = MÜKEMMEL (AKD1AY)
  - 7/8 = %87 = HARİKA (AKDRS)
  - 6/8 = %75 = İYİ (AKD50)
  - 4/8 = %50 = İDARE EDER (CANTA)
  - 3/8 = %37 < ÇALIŞMAYA DEVAM (hediye yok)

- [ ] **Kupon Üretimi**
  - Kod format: `HT50-XXX-DDAA-XXXX` ✓ (makeCode() kullanılıyor)
  - QR kod gösteriliyor mu?
  - Geçerlilik tarihi: +1 ay doğru hesaplanıyor mu? ✓ (addOneMonth() kullanılıyor)

- [ ] **Sosyal Kutu**
  - "@zuhalmuzik ve @zuhalakademi etiketle" metni gösteriliyor mu?

- [ ] **PROMO'YA DÖN / TAMAM Butonları**
  - "◀ PROMO'YA DÖN" → gamePromoMelody'ye dönüyor mu?
  - "✓ TAMAM" → hub'a (banner) dönüyor mu?

- [ ] **Kasa Paneli**
  - İstatistik > İstatistik sekmesi → oyuncu listelenyor mu?
  - Kazananlar sekmesi → kupon kodu bulunabiliyor mu?
  - "HT50-AKD1AY-..." formatı doğru mu?

### MIDI Desteği (İsteğe Bağlı)

Rhythm Challenge'daki MIDI dinleyicisini Melody Challenge'a eklemek için:

1. Rhythm Challenge'daki `handleHit()` fonksiyonunun MIDI kısmını kopyala
2. Melody Challenge'da yeni bir `handleMelodyMIDI()` fonksiyonu yaz
3. `"note on"` event'ini `handleMelodyInput(noteNum)` çağrısına bağla

Şimdi yapılmadı çünkü dokunmatik pad yeterli ama gerekirse 30 dakika işi.

### Dosya Boyutu

```
index.html:
  + HTML panelleri: ~7 KB
  + CSS: ~0.5 KB
  + JavaScript: ~9 KB
  ───────────────────────
  Toplam artış: ~16.5 KB

games/melody-challenge/:
  + eye-of-the-tiger.mp4: ~10-15 MB (GEREKLI - henüz bekleniyor)
```

### Bilinen Sınırlamalar

1. **Video dosyası gerekli** - Kod hazır ama `.mp4` dosyası bekleniyor
2. **MIDI henüz eklenmedi** - Dokunmatik pad hazır, MIDI pad için ek kod gerekli
3. **Şarkı süresi** - Eye of the Tiger ~6 dakika; oyun her 8 notayı ~3 saniyede bitiriyor
4. **Arkasından müzik yok** - Web Audio API sadece piyano synth, araç sesi yok

### Nerelerden Test Edilebilir

**Canlı sunucu:**
```
https://zuhal-game.vercel.app
```

**Lokal test:**
```bash
python3 -m http.server 8000
# Tarayıcıda: http://localhost:8000
```

### Sorun Giderme

| Sorun | Çözüm |
|-------|-------|
| "Video oynatılmıyor" | eye-of-the-tiger.mp4 dosyasını games/melody-challenge/ konumuna koyun |
| "Nota sesi yok" | AudioContext izni verin (tarayıcı isteyin) |
| "Kupon kodu boş" | makeCode() ve prizeCodeMelody elementini kontrol edin |
| "QR kod görinmiyor" | QR veri 216 baytı geçerse qrRender() null döner - kodu kısaltın |
| "Pad tıklaması bağlı değil" | document.querySelectorAll(".melody-pad") addEventListener kontrol edin |

### Sonraki Adımlar

1. Eye of the Tiger MP4 dosyası sağla
2. Kioskta test et (1080×1920 dokunmatik ekran)
3. MIDI pad (Roland SPD::One) entegrasyonu ekle (isteğe bağlı)
4. Vercel'e push et - canlıya otomatik yayınlanır

---

**Hazırlayan:** Claude Code
**Tarih:** 2026-09-29
**Durum:** ✓ TAMAMLANMIŞ (Video bekleniyor)
