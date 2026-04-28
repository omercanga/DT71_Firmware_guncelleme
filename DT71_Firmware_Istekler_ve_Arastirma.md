# DT71 Mini Digital Tweezers — Firmware Analizi ve Hedef Değerlendirmesi

> **Firmware:** APP V1.15 / DFU V3.55  
> **Analiz kapsamı:** Kullanıcı kılavuzu V1.2, CAL.INI, APP115.bin (55 KB, STM32 Cortex-M, 0x08010000)

---

## Hedef 1: Auto Mod (Identify)

### Durum: Zaten mevcut — ek değişiklik gerekmez

Firmware'de dört mod bulunuyor:

| İndeks | Mod adı    | Açıklama                        |
|--------|------------|---------------------------------|
| 0      | Measure    | Manuel ölçüm (**varsayılan açılış**) |
| 1      | Identify   | Otomatik eleman tanıma (Auto)   |
| 2      | Calibration| Kalibrasyon                     |
| 3      | Signal Gen | Sinyal jeneratörü               |

**Identify moduna geçiş:** Dokunmatik tuşa **uzun basış** → mod sırası Measure → Identify → Signal Gen → Calibration → tekrar başa döner.

Identify modundayken:
- Ekranın sol alt köşesinde **"A"** göstergesi belirir.
- Otomatik olarak tanınan eleman türleri: **L (indüktans), C (kapasitans), R (direnç), D (diyot)**
- Ekranda ana parametre + yardımcı parametre birlikte görünür (örn. `10uH  1Ω`)

**Cihaz son modu NV'ye kaydeder:** APP113 sürüm notlarında "NV gear configuration" değişikliği yer alıyor. Bu, cihazın son kullanılan modu non-volatile bellekte sakladığı anlamına gelir. Identify moduna bir kez geçilip cihaz kapatılırsa, büyük olasılıkla bir sonraki açılışta Identify modunda başlar. **Bunu doğrulamak için test edin.**

---

## Hedef 2: Varsayılan Açılış Modu

### Durum: CAL.INI yoluyla değiştirilemez — firmware binary patch gerektirir

**CAL.INI'nın desteklediği parametreler** (kılavuz s.13, kesin liste):

| Parametre          | Varsayılan | Açıklama                          | Aralık |
|--------------------|-----------|-----------------------------------|--------|
| `SLEEP_TIME`       | 60        | Uyku süresi (saniye)              | 30–999 |
| `DISPLAY_DIRECTION`| 4         | 0=sağ el, 3=sol el, 4=otomatik   | 0/3/4  |
| `OLED_BRIGHTNESS`  | 2         | Ekran parlaklığı                  | 0–10   |
| `TSC_SEN`          | 1         | Dokunmatik hassasiyeti            | 0/1/2  |
| `SINE_FREQ_OPT`    | 0         | Sinüs frekansı                    | 0–5    |
| `NOISE_FREQ_OPT`   | 0         | Gürültü frekansı (yalnızca 100KHz)| 0      |
| `USER_FREQ_OPT`    | 0         | Kullanıcı dalga frekansı          | 0–5    |
| `PULSE_FREQ_OPT`   | 0         | Darbe frekansı                    | 0–8    |
| `USER_WAVEFORM`    | sinüs     | 128 noktalı özel dalga tablosu    | 0x000–0xFFF |
| `CALRB_K0/K1`      | —         | Kalibrasyon katsayıları           | —      |

`DEFAULT_MODE`, `STARTUP_MODE` gibi bir parametre **yoktur** — ne CAL.INI'da ne de DFU bootloader'da (binary analizi ile doğrulandı).

### Binary patch yolu (riskli, yedek alarak deneyin)

Firmware binary analizi:
- APP binary: STM32, 0x08010000 baz adresi
- DFU bootloader: CAL.INI'yı parse eder, sonucu NV'ye yazar — DFU binary'si elimizde yok
- APP binary'sindeki mod tablosu: `0x08015A34` (Measure / Identify / Calibration / Signal Gen)

Eğer cihaz NV'ye son modu kaydedip geri okuyorsa (Hedef 1'de belirtilen test), binary patch'e gerek kalmaz. Eğer kaydedip okumuyorsa, Ghidra veya Radare2 ile APP binary'si tam disassemble edilmeli ve mode init adresindeki `0x00` baytı `0x01` yapılmalıdır. Bu işlem için:

```bash
# Ghidra açık kaynaklı, ücretsiz
# Dosyayı Ghidra'ya yükle: DT71APP115.bin, ARM Cortex-M Little Endian
# Base address: 0x08010000
# Reset handler: 0x0801D698
# 'Identify' string: 0x08015A44
# Mode table: 0x08015A34
```

---

## Hedef 3: Kısa Devre / Süreklilik (Continuity) + Beep

### Durum: Donanım ve firmware desteklemiyor — mümkün değil

**Firmware string analizi (55 KB binary taraması):**
- `beep`, `buzz`, `tone`, `cont`, `short` → **hiçbir string yok**
- Donanım şemasında buzzer/piezo bileşeni görünmüyor
- DT71 bir LCR metre — multimetre değil; sesli devamlılık testi donanımsal olarak tasarlanmamış

**Pratik alternatif:** Rx modunda 0Ω'a yakın okumaları kendiniz yorumlayın. Kısa devre eşiği donanıma bağlı, firmware konfigürasyon eşiği yok.

---

## CAL.INI Değişiklikleri (yapılan güncelleme)

`TSC_SEN=1` parametresi eklendi (kılavuzda belgelenmiş ama orijinal CAL.INI'da eksikti). Bu parametre dokunmatik tuş hassasiyetini ayarlar.

**Kalibrasyon katsayılarına (CALRB_K0/K1) kesinlikle dokunmayın** — değiştirirseniz ölçüm doğruluğu bozulur ve geri almak için yeniden kalibrasyon gerekir.

---

## Özet: Ne Yapılabilir, Ne Yapılamaz

| Hedef | CAL.INI | Binary patch | Durum |
|-------|---------|-------------|-------|
| Auto/Identify modu | — | — | **Zaten var (uzun basış)** |
| Açılışta Identify başlasın | Hayır | Evet (riskli) | Önce NV kalıcılığını test edin |
| Kısa devre + beep | Hayır | Hayır | Donanım yok |

---

## Tuş Kullanımı (hızlı başvuru)

- **Uzun basış**: Mod geçişi (Measure → Identify → Signal Gen → Calibration → …)
- **Kısa basış**: Seçili mod içinde alt-menü/ölçüm tipi değiştirme

Ölçüm alt-tipleri (Measure modunda kısa basış sırası): `Rx → Dx → Cx → Lx → Fx → Vx → Rx →…`
