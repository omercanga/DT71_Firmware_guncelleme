# DT71 Mini Digital Tweezers — İstekler ve Araştırma Notları

Bu doküman, DT71 için istenen davranışları tek bir “prompt” halinde toplar ve mevcut dosyalar/online kaynaklardan çıkabilen uygulanabilirlik notlarını özetler.

## Prompt (kopyala‑yapıştır)

DT71 Mini Digital Tweezers için firmware/ayar geliştirmesi yapmak istiyorum.

Elimde DT71APP108/113/115 firmware (Intel HEX) ve cihaz USB disk olarak bağlanınca çıkan `CAL.INI` dosyası var.

Hedefler:
1) **Auto mod**: Uçlara dokundurulan parçanın türünü (voltaj mı, direnç mi, kapasitans mı) otomatik algılayıp uygun ölçümü başlatması.
2) **Varsayılan açılış modu**: Cihaz açılınca önce Voltaj (V) modunda başlasın (veya Auto ile başlasın).
3) **Kısa devre/süreklilik fonksiyonu**: 0Ω’a yakın ölçümlerde hızlı tepki ve sesli uyarı (beep) gibi bir “short/continuity” davranışı olsun; eşik değeri ayarlanabilir olsun.

Kısıtlar:
- Kaynak kodum yok; sadece `.hex` ve `CAL.INI` gibi dosyalar var.
- `CAL.INI` içinde `SLEEP_TIME`, `DISPLAY_DIRECTION`, `OLED_BRIGHTNESS`, waveform tabloları ve kalibrasyon katsayıları (örn. `CALRB_K0/K1`) bulunuyor.

İstediğim çıktı:
- Bu hedeflerin `CAL.INI` veya cihazın USB diskindeki başka bir param dosyasıyla yapılıp yapılamayacağı.
- Eğer `CAL.INI` yetmiyorsa, firmware içinde hangi yapıların (NV/gear config, mode order, threshold) değişmesi gerektiği ve bunun pratikte nasıl yapılacağı (tersine mühendislik/patch vs.).
- Riskler ve uygulanabilir en güvenli yaklaşım (ör. sadece param ile ayar / firmware patch / mümkün değilse alternatif).

## Mevcut bulgular (eldeki dosyalardan)

- Forumdan indirilen DT71 paketleri (DT71APP108/113/115) tipik olarak sadece `readme.txt + .hex` içeriyor.
- Cihaz PC’ye bağlanınca görünen `CAL.INI` dosyası; uyku, ekran yönü, parlaklık, waveform/frekans seçenekleri ve kalibrasyon katsayıları gibi parametreler içeriyor.
- `CAL.INI` içeriğinde “default açılış modu”, “auto mode decision thresholds” veya “continuity/short threshold” gibi açık anahtarlar **bulunmayabilir** (dosyanın sürümüne göre değişebilir).

## İnternetten araştırma özeti (bulunabilen resmi/yarı-resmi kaynaklar)

Araştırmada öne çıkan nokta: `CAL.INI` üzerinden ayarlanabilen parametreler genellikle UI/konfor ve waveform seçenekleri ile sınırlı olarak anlatılıyor.

- `CAL.INI` ayarları (uyku/yön/parlaklık ve benzeri): `https://manuals.plus/miniware/miniware-dt71-mini-digital-tweezers-manual`
- Bazı kaynaklarda “kısa devre kontrolü” için ayrı bir continuity modu yerine, **Rx (direnç)** modunda 0Ω’a yakın okumanın kısa devre olarak yorumlandığı belirtilir: `https://manuals.plus/miniware/miniware-dt71-mini-digital-tweezers-manual`

## Pratik öneri (en düşük riskten yükseğe)

1) **Sadece `CAL.INI` ile güvenli ayarlar**  
   - Uyku süresi, ekran yönü, parlaklık, waveform gibi parametreler.
   - Kalibrasyon katsayılarına (`CALRB_*`) dokunmak ölçüm doğruluğunu bozabilir; değişiklik yapılacaksa yedek alın.

2) **USB disk üzerinde başka dosya var mı kontrolü (kritik)**  
   - Diskte `CAL.INI` dışında `NV.INI`, `SYS.INI`, `PARAM.INI`, `CFG.INI` gibi dosyalar varsa; “default mode / threshold” gibi ayarlar orada olabilir.

3) **Firmware ile değişiklik (kaynak kod yoksa riskli)**  
   - “Açılışta voltaj/auto” ve “kısa devre/süreklilik için ayrı fonksiyon” çoğu cihazda firmware mantığı gerektirir.
   - Kaynak kod olmadan: tersine mühendislik + binary patch gerekir; zaman/risk yüksektir.

