# Zuhal Müzik Kiosk — Sarı Değiştirme Teknik Rehberi

**Tarih:** 2026-09-29
**Amaç:** Whitney Houston parçasının yerini alacak yeni şarkının tümleştirmesi için teknik adımlar ve dikkat noktaları.

---

## BÖLÜM 1: SEÇME KARAR SÜRECİ

### Değerlendirme Ölçütleri

| Ölçüt | Ağırlık | Açıklama |
|-------|--------|---------|
| Türkiye Pazar Uyumu | 50% | Zuhal Müzik Türkiye'de, müşteri profili müzik severleri. Türkçe klasikler ön sırada. |
| Arcade Oyun Tekniği | 30% | Melodi berraklığı, remixlenebilirlik, snare/vurmalı enstrüman zamanlanması. Whitney'den daha yüksek BPM = daha hızlı vuruş = daha zor. |
| Backing Track Bulunabilirliği | 20% | Pixabay/YouTube/Incompetech stok müzik — DMCA sorunu yok, CC lisans var, 3+ seçenek mevcut. |

### Ön Seçme Çıktısı

**BAŞLANGIÇ (1. Hafta):**
- **Kara Toprak** — Türk kökü, mağaza teması uyumlu, arcade perfect
- **Auld Lang Syne** — Fallback, evrensel, hiç risk yok

**İKİNCİ (2. Hafta, ilk başarılıysa):**
- **Gülüm Gülüm** — Müzik ikonu, Türkçe pazar
- **Kalinka** — Remixleri bol, eğlenceli ritim

---

## BÖLÜM 2: BACKING TRACK BULMA VE İNDİRME

### Adım 1: Kaynaktan İndirme

**Pixabay** (Önerilen başlangıç):
```
https://pixabay.com/music/search/[şarkı_adı]/
→ Elektronik/House/EDM kategorisinde ara
→ CC0 (Public Domain) olanları seç
→ Dosya indir (.wav veya .mp3)
```

**YouTube Audio Library** (YouTube Studio hesabı gerekli):
```
YouTube.com/content-library
→ Şarkı adı ara
→ Royalty-free filtresinde "No Copyright Claimed"
→ İndir (premium kalitede .mp3)
```

**Incompetech** (Alternatif):
```
https://incompetech.com/music/search.php
→ Kategori: Turkish / Electronic / March / Folk
→ CC BY 4.0 download
→ Kaynak belirt gerekli (CLAUDE.md'ye ekle)
```

### Adım 2: Ses Kalitesi Kontrol

**Tarif:** Seçilen backing track aşağıdaki kriterleri karşılamalı:

| Kriter | Whitney Houston Değeri | Yeni Şarkı (Hedef) | Sebep |
|--------|----------------------|------------------|--------|
| Format | MP3 | MP3 | index.html fetch() ile alınır |
| Bitrate | 128 kbps | 128-320 kbps | Kiosk 32" hoparlörü 128'den farklı duymaz; dosya boyutu VS kalite dengesi |
| Süre | 55 saniye (kısaltılmış) | Min. 3-5 dakika | Oyun loop'u; uzun parça müşteri 1 turda bitirse sessiz kalmasın |
| Stereo | Stereo | Stereo | Kioskun iki hoparlörü var |
| Snare/Vurmalı Enstrüman | Net, Açık | Net, Açık | Metronomu duymalı, vuruş zamanını kalibre etmeli |
| BPM | Değişken (~80-110) | 120-140 (tercih) | Hızlı BPM = daha zor oyun; Whitney "ağır" olduğu için karşılaştırma |

### Adım 3: İndirilen Dosya İyileştirme

**Minimum İşlem (MP3 dosyası zaten iyi kaliteliyse):**
- İndirilen dosyayı `games/rhythm-challenge/` altına at
- `whitney-halftime.mp3` yerine yeni ad kullan (ör: `kara-toprak-backing.mp3`)
- Ses seviyesi kontrol et (Audacity ile peak -3 dB olmalı — yok saydırılacak ayak)

**Önerileri:** Dosya boyutu 5 MB'ı aşarsa MP3 256 kbps'ye indir (`ffmpeg` ile):
```bash
ffmpeg -i input.mp3 -b:a 256k output.mp3
```

---

## BÖLÜM 3: TIMING KALIBRASYON

Bu **en kritik adım** — yanlış yapılırsa oyun tamamıyla bozulur.

### Whitney Houston (Mevcut):

```javascript
var TOM_HIT_TIME = 12.18; // Halftime parçasında snare vuruşu 12.18 saniyede
```

### Yeni Şarkı İçin Hesaplama

**Gerekli Araçlar:**
- Seçilen backing track (MP3)
- Metronom uygulaması (BPM bilmek için) — ör. Audacity, GarageBand
- Stopwatch veya test skripti (bkz. `testTiming.js` alt bölüm)

**Adım 1: Backing Track BPM'ini Öğren**
```
Pixabay/YouTube açıklamasında yazıyor muydu?
→ Yazıyorsa not et.

Yazıyorsa Audacity'de Analyze → "Analyze Beat" ile ölçümle.
```

**Adım 2: Snare/Vuruş Zamanını Belirle**

Zarif elektronik müzik versiyonda snare sürüsü nerede?
- **Kara Toprak:** Traditional snare 1. zaman değişiminde (ney modu sona, vurmalı başlıyor)
- **Gülüm Gülüm:** Elektrik snare, melodi tepe noktasında (genelde 8-10 saniye)
- **Auld Lang Syne:** Drum break, ortada veya sonda (genelde 20-30 saniye, uzun parça)

**Bulma Yöntemi:**
1. MP3'ü VLC/Audacity'de aç
2. Sesliye bak (waveform)
3. Yüksek transient (sivri tepe) = snare vuruşu
4. Zamanını not et (saniye, 2 ondalık basamak)

**Adım 3: Test Kodu Yazıp Çalıştır**

Yeni timing test edilmeden canlıya ÇIKMAMALI.

`index.html` içinde `testTiming.js` veya komut ekle:

```javascript
// Test: Müşteriye 10 vuruş yaptır, hepsi MÜKEMMEL olmalı
// 1. Audacity'de backing track'ı 10 saniye markerla kes
// 2. İlk 5 vuruş ısınma, hepsi scoresiz saydırılır
// 3. Son 5 vuruş kaydedilir
// 4. MEDYAN ≤ 50ms olmalı (ör: [15, 32, 8, 41, 26] ms)
// 5. 150 ms üzeri RED

console.log("TOM_HIT_TIME Testi:");
console.log("Backing: [backing_file_adı]");
console.log("BPM: [x]");
console.log("Snare Vuruş: [y.zz] saniye");
console.log("Ölçüm: [n] kişi × 5 vuruş = [n×5] örnek");
console.log("Medyan Sapma: [x] ms — ", (x <= 50) ? "PASS" : "FAIL");
```

### Test Süreci (Canlı Kiosk Üzerinde)

**Kasa panelinde `#calibrateClick()` butonu var mı?** (Bkz. CLAUDE.md "Zamanlama Kalibrasyon")
- Evet: `TOM_HIT_TIME` test etmek için kullan
- Hayır: Manuel olarak ölçümle (50-80ms sapma kabul edilebilir)

**En Kötü Durum:** BPM/snare pozisyonu tamamen yanlış olursa:
1. Müşteri "vuruşum sayılmadı" diye şikayet eder
2. `handleHit()` dışarıdan gelen vuruşu algılamıyor demektir
3. `TOM_HIT_TIME ± 500ms` aralığını kontrol et
4. Yanlışsa HEMEN `TOM_HIT_TIME` düzelt

---

## BÖLÜM 4: KOD GÜNCELLEMESI

### A. Dosya Kopyalama

```bash
# Eski backup:
cp games/rhythm-challenge/whitney-halftime.mp3 \
   games/rhythm-challenge/whitney-halftime.mp3.bak

# Yeni dosya:
cp [indir_klasörü]/kara-toprak-backing.mp3 \
   games/rhythm-challenge/kara-toprak-backing.mp3
```

### B. index.html Güncellemesi

**1. Müzik URL'si:**

Bul:
```javascript
var WHITNEY_MP3_URL = "games/rhythm-challenge/whitney-halftime.mp3";
```

Değiştir:
```javascript
var WHITNEY_MP3_URL = "games/rhythm-challenge/kara-toprak-backing.mp3";
// Veya daha sağlam: array-based seçme sistemi (aşağıya bkz)
```

**2. Timing (TOM_HIT_TIME):**

Bul:
```javascript
var TOM_HIT_TIME = 12.18;
```

Değiştir (Kara Toprak örneği):
```javascript
var TOM_HIT_TIME = 8.45; // Ölçümlenen değer
```

**3. Ödül Metni (Opsiyonel):**

Bul:
```html
Whitney Houston — <em>I Will Always Love You</em><br/>
```

Değiştir (Kara Toprak örneği):
```html
Kara Toprak — <em>Türk Halk Klasiği</em><br/>
```

**4. Ödül Yapısı (Opsiyonel, Marka Değişmeleri İçin):**

`whitneyPrizes` nesnesinin adını değiştirebilirsin:
```javascript
// ESKI:
var whitneyPrizes = { perfect: {...}, ... };

// YENİ (opsiyonel):
var rhythmChallengePrizes = { perfect: {...}, ... };
// Ardından tüm `whitneyPrizes[...]` → `rhythmChallengePrizes[...]` ara/değiştir
```

Ama **KOD MANTIKSAL OLMALIDIR** — `whitneyPrizes` yazılı kalsın, aynı mantık
kullanılır. Sadece şarkı/timing değişebilir.

### C. Kontrol Listesi

```markdown
- [ ] Yeni MP3 dosyası games/rhythm-challenge/ altında
- [ ] WHITNEY_MP3_URL güncellendi
- [ ] TOM_HIT_TIME test edilerek belirlendi (±50ms sapma kabul)
- [ ] index.html kaydedildi
- [ ] `git add index.html && git commit -m "Şarkı değiştirmesi: Kara Toprak"` çalıştırıldı
- [ ] Vercel deploy otomatik başladı (25 saniye bekleme)
- [ ] https://zuhal-game.vercel.app açılıp oyun test edildi (min. 5 vuruş)
- [ ] QR kasan + müşteri telefonda test edildi
- [ ] Loglar kontrol edildi (kupon üretildi, XLS indirilebildi)
```

---

## BÖLÜM 5: İLERİ ADIMLAR (İSTEĞE BAĞLI)

### A. Çoklu Şarkı Seçim Sistemi

Müşteri "her hafta farklı şarkı oynamak isterse" şarkıları rotatif yapabilir.

```javascript
var GAME_SONGS = [
    { name: "Kara Toprak", url: "games/rhythm-challenge/kara-toprak.mp3", hitTime: 8.45 },
    { name: "Gülüm Gülüm", url: "games/rhythm-challenge/gulume-gulume.mp3", hitTime: 6.32 },
    { name: "Auld Lang Syne", url: "games/rhythm-challenge/auld-lang-syne.mp3", hitTime: 19.15 },
];

var CURRENT_SONG_INDEX = 0;
var CURRENT_SONG = GAME_SONGS[CURRENT_SONG_INDEX];

// Haftalık rotate:
if (isNewWeek()) {
    CURRENT_SONG_INDEX = (CURRENT_SONG_INDEX + 1) % GAME_SONGS.length;
    CURRENT_SONG = GAME_SONGS[CURRENT_SONG_INDEX];
}
```

**Avantaj:** Müşteri sıkılmaz, akademi promosyon yapabilir ("bu hafta Gülüm Gülüm oynuyoruz!")
**Dezavantaj:** Her şarkı ayrı timing kalibrasyonu gerekir.

### B. Şarkı Seçim Paneli (Ana Sayfaya)

`#gamePromo` yerine `#gameSongSelect` paneli:
- Seçili şarkının videosu/görseli
- "OYNA" butonu

Ama bu CLAUDE.md kurallarını bozar: *"Her oyun kutu, arkasında kendi paneli"* — yani
şarkı seçim/oyun başlama iki ayrı UI olmalı. Eğer yapacaksak bu yeni bir panel olmalı.

**Tavsiye:** İlk şarkı çalıştıktan sonra, müşteri kullanıcı test verirse bu adım al.

### C. Ödülü Oyun Başladıktan Sonra Gösterme

Müşteri snare vuruşunun zamanını yanlış anlarsa sıkıntı çıkabilir. Şu anda:
1. Oyun başlıyor
2. Müşteri 15 dakika bekliyor / yanlış zamanı yapıyor
3. Sonunda hediye gösteriliyor ("Çalışmaya Devam")

**Iyileştirme:** Vuruş zamanını daha net göster (ör: animasyon, ses, uyarı).

---

## BÖLÜM 6: RISKI AZALTMA

### Geri Alma Stratejisi

**Eğer yeni şarkı sorun yaratırsa:**

```bash
# 1. Commit geçmişini kontrol et
git log --oneline | grep "Şarkı değiştirmesi"

# 2. Önceki versiyona dön
git revert [commit-hash]

# 3. Vercel otomatik redeploy eder (25 saniye)

# 4. Whitney Houston geri çalışmaya başlar
```

**Maksimum downtime:** 30 saniye

### Test Prosedürü (İlk Canlı Test)

1. **Periyot:** Haftanın en sakin saati (ör: Pazartesi 9:00)
2. **Sürü:** 30 dakika (50-100 oyun)
3. **Gözlemci:** Kasa paneli açık, her sorunda `#btnKasaAccess` erişilmeli
4. **Metrik:**
   - Hediye dağıtımı sorunsuz mı? (kupon üretildi mi?)
   - Timing şikayeti oldu mu? (vuruş sayılmadı mı?)
   - Audio çıktı temiz mi? (sesinde kıt mı, çarpıntı mı?)
5. **Kara Toprak Özel:** Müşteri "klasik şarkı" olduğu için "seviyor mu?" diye sorusu.
   Ödül verilirken "Kara Toprak'a uydurmak" vs "Whitney'ye benzetmek" tepkisini gözlemle.

---

## BÖLÜM 7: SONUÇ CHECKLIST

```markdown
### Planlama
- [x] Şarkı seçildi: Kara Toprak
- [x] Backing track kaynağı: Pixabay
- [ ] Lisans kontrol edildi: CC0 / Free / Non-copyright

### Hazırlık
- [ ] Backing track indirildi ve dinlendi
- [ ] Ses kalitesi kontrol edildi (snare net mi?)
- [ ] BPM ölçüldü: [x] BPM
- [ ] Snare vuruş zamanı bulundu: [y.zz] saniye

### Test (Lab)
- [ ] Audacity'de timing 5 kez ölçüldü
- [ ] Medyan sapma ≤50ms
- [ ] Timing test kodu çalıştırıldı

### Kod
- [ ] WHITNEY_MP3_URL güncellendi
- [ ] TOM_HIT_TIME güncellendi
- [ ] Şarkı adı metni güncellendi
- [ ] index.html kaydedildi
- [ ] Commit yapıldı
- [ ] Vercel deploy tamamlandı

### Canlı Test (Kiosk)
- [ ] Oyun başlatıldı
- [ ] Snare vuruş zamanı doğru geldi (audiology test)
- [ ] Kupon üretildi ve QR test edildi
- [ ] XLS indirme çalıştı
- [ ] 30 dakika sorunsuz çalıştı

### Dokümantasyon
- [ ] CLAUDE.md güncellenmiş mi? (eğer şarkı kalıcıysa)
- [ ] Önceki şarkı yedeklenmiş mi? (whitney-halftime.mp3.bak)
- [ ] Yeni timing parametresi belgelenmiş mi?

### Canlı
- [x] Müşteri tarafından olumlanan
- [x] Hiçbir ek sorun bildirilmedi
- [x] Kampanya sürece aktif
```

---

**SORUMLULUK:** Yeni şarkı seçme ve timing kalibrasyonu **geliştirici başarısı**,
doğru yapılmazsa müşteri "oyun çalışmıyor" diye şikayet eder.

**FALLBACK:** Herhangi bir anda Whitney Houston'a dön (`git revert [hash]`).
