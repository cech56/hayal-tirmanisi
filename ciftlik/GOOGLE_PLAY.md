# Bizim Yayla'yı Google Play'e Yükleme Rehberi

Oyun bir web uygulaması (PWA). Google Play'e **TWA (Trusted Web Activity)** olarak yüklenir: Android uygulaması, oyunun web adresini tam ekran açan ince bir kabuktur. Kod değiştirmeden güncellenir: siteyi güncellediğinde oyun da güncellenir.

## 1. Oyunu HTTPS bir adreste yayınla

En kolayı GitHub Pages:

1. GitHub'da depo → **Settings → Pages**.
2. Branch olarak oyunun olduğu dalı seç, klasör `/ (root)`.
3. Oyun şu adreste açılır: `https://<kullanıcı-adı>.github.io/hayal-tirmanisi/ciftlik/`

`ciftlik/` klasöründe şunlar hazır:
- `manifest.json`: uygulama adı, simgeler, tam ekran ve dikey ekran ayarı.
- `sw.js`: service worker. Oyun ilk açılıştan sonra internetsiz de çalışır.
- `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`: simgeler.

Kontrol: Chrome'da adresi aç → F12 → **Lighthouse → PWA**. "Installable" görünmeli.

## 2. Android paketi (AAB) üret: Bubblewrap

Bilgisayarda Node.js ve Java (JDK 17) kurulu olmalı.

```bash
npm i -g @bubblewrap/cli
mkdir bizim-yayla-android && cd bizim-yayla-android
bubblewrap init --manifest https://<kullanıcı-adı>.github.io/hayal-tirmanisi/ciftlik/manifest.json
```

Sorulara örnek cevaplar:

| Soru | Cevap |
|---|---|
| Application ID | `com.bizimyayla.oyun` |
| App name | `Bizim Yayla` |
| Display mode | `fullscreen` |
| Orientation | `portrait` |
| Play Billing | **Evet** (uygulama içi satın alma için) |
| Signing key | Yeni anahtar oluştur. **Şifreyi ve `.keystore` dosyasını kaybetme!** |

Sonra paketi üret:

```bash
bubblewrap build
```

Çıkan `app-release-bundle.aab` dosyası Play Console'a yüklenecek dosyadır.

Alternatif: Bubblewrap kurmak istemezsen [PWABuilder](https://www.pwabuilder.com) sitesine adresi yaz, **Package for stores → Android** seç. Aynı dosyaları indirirsin.

## 3. Digital Asset Links (adres çubuğunu gizlemek için şart)

Bubblewrap bir SHA-256 parmak izi verir (`bubblewrap fingerprint` ile de görülür). Şu dosyayı oluştur:

`https://<kullanıcı-adı>.github.io/.well-known/assetlinks.json`

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.bizimyayla.oyun",
    "sha256_cert_fingerprints": ["AA:BB:...:FF"]
  }
}]
```

Notlar:
- Bu dosya alan adının **kökünde** olmalı. Proje sitesinde (`/hayal-tirmanisi/`) çalışmaz.
- Bunun için ayrı bir `<kullanıcı-adı>.github.io` deposu aç ya da kendi alan adını bağla.
- Play App Signing kullanırsan Play Console → **Kurulum → Uygulama imzalama** sayfasındaki SHA-256'yı da listeye ekle.

## 4. Play Console

1. [play.google.com/console](https://play.google.com/console) → geliştirici hesabı aç (tek seferlik 25 $).
2. **Uygulama oluştur** → Oyun → Ücretsiz.
3. Mağaza girişi:
   - **Kısa açıklama:** "Köyünü kur, hayvanlarına bak, yaylaya çık!"
   - **Uzun açıklama:** oyunun özellikleri (hayvanlar, köy, sohbet, traktör, panayır, mevsimler…).
   - **Ekran görüntüleri:** en az 2 dikey telefon görüntüsü.
   - **Simge:** 512×512 (`icon-512.png`).
   - **Öne çıkan görsel:** 1024×500.
4. **İçerik derecelendirmesi** anketi: şiddet yok, kumar yok. Tavla sadece oyun içi parayla oynanır.
5. **Gizlilik politikası** adresi gerekiyor (aşağıda örnek var).
6. **Veri güvenliği formu:** "Veri toplanmıyor". Kayıt sadece cihazda (localStorage) tutuluyor.
7. **Test → Dahili test** kanalına AAB'yi yükle, kendin dene, sonra **Üretim**'e gönder.

## 5. Uygulama içi satın almalar (Play Billing)

Oyundaki mağaza (Telefon → 💎 Mağaza) Google Play'in **Digital Goods API**'sini kullanır. TWA içinde açıldığında gerçek Google Play ödeme ekranı çıkar. Normal tarayıcıda sadece bilgi gösterir.

Play Console → **Para kazanma → Ürünler → Uygulama içi ürünler**. Kimlikleri **aynen** şu şekilde oluştur:

| Ürün kimliği | Ad | Tür | Önerilen fiyat |
|---|---|---|---|
| `baslangic` | Başlangıç paketi (50.000₺ + 1 inek + 10 tavuk + bakıcı) | Tek seferlik | 49,99 TL |
| `kese_kucuk` | Küçük kese (25.000₺ oyun parası) | Tüketilebilir | 29,99 TL |
| `kese_buyuk` | Büyük kese (150.000₺ oyun parası) | Tüketilebilir | 129,99 TL |
| `kese_dev` | Ağa sandığı (1.000.000₺ oyun parası) | Tüketilebilir | 349,99 TL |
| `usta_paket` | Usta Çiftçi paketi (satışlar kalıcı %20 fazla) | Tek seferlik | 89,99 TL |
| `altin_traktor` | Altın traktör + 2 tohum paketi | Tek seferlik | 109,99 TL |
| `araba_simsek` | Kırmızı Şimşek GT (süper spor araba) | Tek seferlik | 199,99 TL |
| `araba_yildirim` | Sarı Yıldırım (süper spor araba) | Tek seferlik | 179,99 TL |
| `araba_altin` | Altın Lüks Sedan | Tek seferlik | 149,99 TL |

Bubblewrap'te Play Billing'i açtıysan ayrıca kod gerekmez.

Ürünleri test etmek için: Play Console → **Ayarlar → Lisans testi** bölümüne kendi Gmail adresini ekle. Test alımları ücretsiz olur.

## 6. Örnek gizlilik politikası

Bunu `ciftlik/gizlilik.html` olarak yayınlayabilirsin:

> **Bizim Yayla Gizlilik Politikası**
> Bizim Yayla kişisel veri toplamaz. Oyun kaydı yalnızca cihazınızda saklanır, hiçbir sunucuya gönderilmez. Uygulama içi satın almalar Google Play tarafından işlenir; ödeme bilgilerinize erişimimiz yoktur. Sorularınız için: <iletişim e-postası>

## 7. Güncelleme

- Oyun dosyalarını (`ciftlik/index.html` vb.) güncelleyip yayınla. Kullanıcılar yeni sürümü otomatik alır.
- Önbelleği yenilemek için `sw.js` içindeki `bizim-yayla-v1` sürümünü `v2` yap.
- Yeni AAB yüklemek sadece simge, ad veya paket ayarı değişince gerekir. O zaman `twa-manifest.json`'da `appVersionCode` değerini artırıp `bubblewrap build` çalıştır.

## 8. Google Drive bulut kaydı (isteğe bağlı)

Oyunda Google Drive'a otomatik kayıt kodu hazır. Bağlanan oyuncunun çiftliği 5 dakikada bir kendi Drive'ının gizli uygulama klasörüne yazılır. Telefon değişince "Buluttan yükle" ile geri gelir.

Açmak için bir **OAuth istemci kimliği** gerekiyor. Kimlik olmadan bu bölüm oyunda hiç görünmez.

1. [console.cloud.google.com](https://console.cloud.google.com) → yeni proje oluştur.
2. **API'ler ve Hizmetler → Kitaplık** → **Google Drive API**'yi etkinleştir.
3. **OAuth izin ekranı:**
   - Tür: Harici.
   - Uygulama adı: Bizim Yayla.
   - Kapsam olarak `.../auth/drive.appdata` ekle. Bu kapsam yalnızca oyunun kendi gizli klasörünü görür, kişinin dosyalarını görmez.
4. **Kimlik bilgileri → OAuth istemci kimliği oluştur:**
   - Tür: Web uygulaması.
   - Yetkili JavaScript kaynakları: `https://<kullanıcı-adı>.github.io` (ya da kendi alan adın).
5. Çıkan kimliği (`...apps.googleusercontent.com`) oyunun kaynağında şu satıra yaz:
   ```js
   const GCLIENT='BURAYA_KIMLIK';
   ```
   Ardından `index.html`'i yeniden yayınla.
6. Oyunda **Telefon → ⚙️ → ☁️ Google Drive → Google ile bağlan**.

İzin ekranı "Test" modundayken sadece test kullanıcısı olarak eklediğin Gmail adresleri bağlanabilir. Herkese açmak için Google'a doğrulamaya gönder. `drive.appdata` hassas olmayan bir kapsam olduğu için genelde hızlı onaylanır.

## Kontrol listesi

- [ ] Oyun HTTPS adreste açılıyor, Lighthouse "Installable" diyor
- [ ] `assetlinks.json` alan adının kökünde, SHA-256 doğru
- [ ] AAB imzalandı, keystore yedeklendi
- [ ] Mağaza girişi, ekran görüntüleri, simge, öne çıkan görsel
- [ ] İçerik derecelendirmesi ve veri güvenliği formu dolduruldu
- [ ] Gizlilik politikası adresi eklendi
- [ ] 9 uygulama içi ürün aynı kimliklerle oluşturuldu
- [ ] Dahili testte satın alma denendi
