---
title: REA Kullanım Rehberi
aliases: [rea, rea cli, reverse engineer anything]
tags: [rea, tersine-muhendislik, cli, rehber]
versiyon: 6.5.0
olusturma: 2026-10-15
---

# REA Kullanım Rehberi

> [!abstract] REA nedir?
> **REA** (`rea@6.5.0`), terminalden veya bir yapay zekâ ajanından kullanılan bir tersine mühendislik aracıdır. Uygulamaları (native, Android, Apple, Electron/JS, .NET, WASM, web, firmware vb.) analiz eder; sonuçları **Evidence** (kanıt) adı verilen doğrulanabilir JSON paketleri olarak üretir.

> [!warning] Yasal ve etik uyarı
> Yalnızca **sahibi olduğunuz** veya **analiz izniniz olan** yazılım/cihazlar üzerinde kullanın. `debug-with-*`, `instrument-with-frida`, `install-adb-package`, `uninstall-adb-package`, `observe-native-calls` gibi komutlar hedefi çalıştırır veya cihazı değiştirir.

> [!info] Bu rehber hakkında
> Komut listesi `rea --help` çıktısına dayanır. Her komutun **tam parametreleri** için her zaman `rea <komut> --help` çalıştırın; bu rehberde parametre uydurulmamıştır.

---

## 1. Hızlı başlangıç

```bash
rea --version          # sürümü göster
rea doctor             # REA kullanıma hazır mı? kontrol et
rea setup              # ajan entegrasyonlarını ve analiz sağlayıcılarını yapılandır
rea providers          # yapılandırılmış analiz sağlayıcılarını listele
rea capabilities       # sağlayıcı yetenekleri ve yan etkileri
rea update             # REA'yı güncelle
```

**İlk kullanım sırası:**

1. `rea doctor` → eksikleri gör (Rizin, GDB, adb, tshark, Frida vb. gerekebilir).
2. `rea setup` → entegrasyonları ve sağlayıcıları kur.
3. `rea providers` ve `rea capabilities` → neyin kullanılabilir olduğunu doğrula.
4. `rea analyze ...` ile ilk analizi yap.

---

## 2. Genel (global) seçenekler

Her komutla birlikte kullanılabilir.

| Seçenek | Açıklama |
|---|---|
| `--format <toon\|json\|yaml\|md\|jsonl>` | Çıktı formatı. Obsidian'a not almak için `md` uygundur. |
| `--filter-output <anahtarlar>` | Çıktıyı anahtar yollarıyla filtrele (`foo,bar.baz,a[0,3]`) |
| `--full-output` | Tam çıktı zarfını göster |
| `--schema` | Komutun JSON Şeması'nı göster |
| `--token-count` | Çıktının token sayısını yazdır (çıktıyı değil) |
| `--token-limit <n>` | Çıktıyı n token ile sınırla |
| `--token-offset <n>` | İlk n token'ı atla (sayfalama gibi) |
| `--llms`, `--llms-full` | LLM tarafından okunabilir manifest |
| `--mcp` | MCP stdio sunucusu olarak başlat |
| `--update` | En son sürüme güncelle |
| `--version`, `--help` | Sürüm / yardım |

### Faydalı örnek kalıplar

```bash
rea <komut> --help                       # parametreleri öğren
rea <komut> --schema                     # girdi/çıktı şemasını gör
rea <komut> ... --format md > not.md     # Obsidian notu olarak kaydet
rea <komut> ... --format json --filter-output anahtar1,anahtar2
rea <komut> ... --token-limit 2000       # büyük çıktıyı kısıt
```

> [!tip] Büyük çıktılar
> Çıktı çok büyükse `--token-limit` ve `--token-offset` ile parça parça okuyun; `--token-count` ile önce boyutu öğrenin.

---

## 3. Temel kavramlar

- **Evidence (kanıt):** Analiz sonucunun doğrulanabilir, kanonik JSON paketi. `compare`, `evidence-export`, `evidence-import`, `inspect-evidence-bundle` bununla çalışır.
- **Provider (sağlayıcı):** Arka planda çalışan analiz motorları (Rizin, Cutter, JEB, Hopper, GDB, Frida, WABT, TShark, sigrok, apktool vb.). `rea providers` ile listelenir.
- **Application Graph:** Bir uygulamanın doğrulanmış envanter/kanıtlarından çıkarılan ilişki grafiği (Android, Apple, Harmony, Managed).
- **Pasif / aktif ayrımı:** `inspect-*` komutları genelde hedefi **çalıştırmadan** inceler. `observe-*`, `capture-*`, `debug-*`, `instrument-*` komutları hedefi çalıştırabilir veya etkileşime girebilir.

---

## 4. Komut referansı

### 4.1 Kurulum ve yönetim

| Komut | Ne işe yarar |
|---|---|
| `doctor` | REA'nın hazır olup olmadığını kontrol eder |
| `setup` | Ajan entegrasyonlarını ve isteğe bağlı analiz sağlayıcılarını yapılandırır |
| `update` | REA kurulumunu günceller ve sürümü doğrular |
| `uninstall` | REA'nın eklediği ajan yapılandırmasını ve skill dosyalarını kaldırır |
| `providers` | Yapılandırılmış analiz sağlayıcılarını listeler |
| `capabilities` | Sağlayıcı yeteneklerini ve yan etkilerini listeler |
| `completions` | Kabuk (shell) otomatik tamamlama betiği üretir |
| `mcp` | REA'yı MCP sunucusu olarak kaydeder (`add`, `doctor`) |

### 4.2 Genel analiz (başlangıç noktası)

| Komut | Ne işe yarar |
|---|---|
| `analyze` | Uygulamaya genel bakış |
| `inspect` | Kanıtlarla birlikte uygulama özeti |
| `inspect-artifact` | Tek bir dosyayı inceler; tüm gözlemleri ve sonraki önerilen adımları verir |
| `decompile` | Uygulamanın bir bölümünü kod olarak okur |
| `function` | Tek bir fonksiyonu kanıtlarıyla eksiksiz analiz eder |
| `instructions` | Decompile etmeden ham makine talimatlarını okur |
| `search` | Analiz edilmiş tüm stringlerde veya prosedür adlarında arar |
| `trace` | Bir literal özelliği referanslar üzerinden izler |
| `xrefs` | Bir adrese yapılan sınırlı sayıda referansı listeler |
| `read-bytes` | Belirli analiz adresindeki baytları okur |
| `search-bytes` | Maskeli bayt kalıplarını bellekte arar |
| `address-to-file-offset` | Analiz adresini dosyadaki bayt ofsetine çevirir |
| `extract-artifact` | Arşiv içeriğini güvenle çıkarır |
| `inspect-analysis-view` | Tamamlanmış analiz Evidence'ından seçili bir görünüm üretir |

> [!example] Tipik akış
> `inspect-artifact` → `analyze` → `search` (ilginç string) → `xrefs` → `function` / `decompile`

### 4.3 Native (Mach-O, ELF, PE) ve disassembly

| Komut | Ne işe yarar |
|---|---|
| `inspect-binary-layout` | Dosya/bağlı yerleşimi, hedefi çalıştırmadan inceler |
| `inspect-macho` | Mach-O incelemesi (Hopper'sız) |
| `inspect-signature` | İmza bilgisi (Hopper'sız) |
| `inspect-plist` | Uygulama plist metadata'sı |
| `list-architectures` | Desteklenen mimariler (Hopper'sız) |
| `inspect-native-api` | Bir fonksiyonun API sınırını kanıtla çıkarır |
| `inspect-native-instruction` | Tek talimatı, çözülmüş operand bilgisiyle inceler |
| `inspect-native-data-type` | Kurtarılmış veri tipi veya tipli veri nesnesi |
| `inspect-native-dispatch-metadata` | Objective-C ve Swift dispatch metadata'sı |
| `inspect-native-load-image` | Yüklenen baytları, kaynak eşlemelerini, relocation ve entry'yi doğrular |
| `resolve-native-call-targets` | Tek bir statik çağrı noktasını çözer |
| `trace-native-values` | Sınırlı def-use ve çağrı bağımlılıklarını izler |
| `trace-native-ui-action` | Derlenmiş UI action selector'ünü native handler'a kadar izler |
| `trace-dylib-resolution` | dyld yükleme yolu çözümlemesini izler |
| `annotate-native-function` | Fonksiyon açıklamalarını düzenler, yenilenmiş analizi döner |
| `demangle-swift` | Swift sembollerini demangle eder |
| `inspect-keyed-archive` | Foundation keyed archive'ı nesne-referans grafiği olarak inceler |
| `inspect-asset-catalog` | Derlenmiş Apple asset catalog metadata'sı |
| `decode-interface-builder` | Storyboard/nib arşivlerini çözer |
| `inspect-go-binary` | Gömülü Go derleyici, modül ve build ayarları |
| `inspect-rust-binary` | Gömülü cargo-auditable Rust bağımlılık listesi |
| `inspect-recorded-crash` | Core register/sinyal ve debugger eşleme kanıtı |
| `inspect-with-objdump` | GNU objdump ile inceler, ham çıktıyı Evidence olarak saklar |
| `list-objdump-architectures` | objdump'ın bildiği BFD hedefleri |

### 4.4 Rizin, Cutter, GDB, Frida (aktif/dinamik)

> [!danger] Dikkat
> Bu komutlar hedef süreci çalıştırabilir, duraklatabilir veya değiştirebilir.

| Komut | Ne işe yarar |
|---|---|
| `execute-rizin-command` | Yerel dosya üzerinde tek bir Rizin komutu çalıştırır |
| `debug-with-rizin` | Geçici oturumda tek bir Rizin debugger komutu |
| `debug-with-gdb` | Geçici MI oturumunda tek bir GDB konsol komutu (canlı hedefi güvenle ayırmak için inferior TTY verin) |
| `cutter-command` | Canlı Cutter örneğinde tek Rizin komutu |
| `list-cutter-sessions` | REA Python eklentisine bağlı canlı Cutter örnekleri |
| `observe-native-calls` | Mach-O'yu LLDB altında başlatıp seçili fonksiyon/metod girişlerini kaydeder |
| `observe-native-ui` | Çalışan bir uygulama penceresini PID + window ID ile gözlemler |
| `capture-native-ui-scenario` | Tek bir pencerede bildirimsel AX eylemleri/beklemeleri çalıştırır |
| `capture-process` | Seçilen tek bir süreç senaryosunu yakalar |
| `compare-process-captures` | İki süreç yakalama Evidence JSON'unu karşılaştırır |
| `list-frida-devices` | Yerel Frida cihazları (+ isteğe bağlı uzak uç nokta) |
| `list-frida-processes` | Frida cihazındaki süreçler |
| `instrument-with-frida` | Seçilen süreçte bir Frida betiği çalıştırır ve ayrılır |

### 4.5 Android (APK)

**Statik analiz:**

| Komut | Ne işe yarar |
|---|---|
| `inspect-android-package` | APK bildirimlerini ve çözülmüş manifest'i, çalıştırmadan inceler |
| `decode-android-resources` | Manifest, metadata ve string kaynaklarını apktool ile çözer |
| `inspect-apktool-client` | Seçili apktool başlatıcısını kontrol eder |
| `search-android-classes` | Sınıf adlarında arar (boş sorgu = tüm sınıflar) |
| `inspect-android-class` | Tek sınıfın üye envanteri |
| `inspect-android-method` | Tek metodu decompile eder (overload seçimiyle) |
| `trace-android-references` | Sınıf/metoda gelen kullanım düğümlerini izler |
| `project-android-application-graph` | APK envanter Evidence'ından Android uygulama grafiği |
| `identify-flutter-build` | Flutter build'inin Dart snapshot hash'i ve engine bilgisi |
| `inspect-dart-aot` | Bir ABI'nin Dart AOT snapshot'ı: bölümler, başlıklar, string havuzu |

**ADB (cihaz) — okuma/inceleme:**

| Komut | Ne işe yarar |
|---|---|
| `inspect-adb-client` | adb istemcisini, cihaza dokunmadan inceler |
| `list-adb-devices` | Bağlı cihazlar (serial, durum, transport, tür) |
| `inspect-adb-device` | Sabit getprop listesiyle cihaz build kimliği |
| `inspect-adb-display` | Ekran boyutu ve yoğunluğu |
| `inspect-adb-window` | Pencere yöneticisinin güncel odak bildirimleri |
| `inspect-adb-package` | Yüklü paketin sürümü, kurucu ve bayrakları |
| `list-adb-packages` | Yüklü paketler ve base APK yolları |
| `resolve-adb-packages` | Paket adını (alt dize) arar |
| `list-adb-processes` | `ps -A` çıktısı kadar süreç listesi |
| `list-adb-services` | Binder servisleri ve arayüzleri |
| `list-adb-features` | Cihazın donanım/yazılım özellikleri |
| `list-adb-directory` | Cihaz dizin listesi (tür, izin, boyut) |
| `read-adb-logcat` | Tek buffer için sınırlı logcat dökümü |
| `read-adb-setting` | `system`, `secure` veya `global` ayarı oku |
| `capture-adb-screen` | Tek ekran karesini PNG olarak yakalar |
| `collect-adb-bugreport` | Cihaz bugreport arşivini toplar ve özetler |

**ADB — dosya aktarımı ve değişiklik yapan komutlar:**

| Komut | Ne işe yarar | Risk |
|---|---|---|
| `pull-adb-file` | Cihazdan dosya çek (SHA-256 ile) | Düşük |
| `pull-adb-package` | Paketin tüm APK setini çek | Düşük |
| `push-adb-file` | Cihaza dosya gönder (`--overwrite` olmadan mevcut hedefi reddeder) | Orta |
| `start-adb-app` | Launcher activity ile uygulamayı başlat | Orta |
| `start-adb-activity` | `am start` ile intent başlat | Orta |
| `stop-adb-app` | Paketin tüm süreçlerini zorla durdur | Orta |
| `install-adb-package` | Yerel APK kur (`--replace` ile veriyi koruyarak yeniden kur) | **Yüksek** |
| `uninstall-adb-package` | Paketi **verisiyle birlikte** kaldır | **Yüksek** |

### 4.6 Apple / HarmonyOS uygulama grafikleri

| Komut | Ne işe yarar |
|---|---|
| `project-apple-application-graph` | IPA veya macOS app envanter Evidence'ından Apple grafiği |
| `project-harmony-application-graph` | HAP, HSP veya App Pack envanterinden HarmonyOS grafiği |
| `project-managed-application-graph` | Yönetilen (managed) statik Evidence'tan grafik |
| `trace-application-feature` | Tipli bir tohumu uygulama Evidence'ı boyunca izler |
| `compare-application-versions` | İki JavaScript Application Graph sürümünü karşılaştırır |

### 4.7 JavaScript / Electron

| Komut | Ne işe yarar |
|---|---|
| `analyze-javascript-application` | Yerel JS/Electron uygulamasını statik olarak yeniden kurar |
| `recover-javascript-sources` | Okunabilir JS modüllerini kaynak izlenebilirliğiyle kurtarır |
| `trace-javascript-semantics` | Sınırlı statik anlamsal ilişkileri izler |
| `compare-javascript-export-shapes` | Statik export dönüş şekillerini çalıştırmadan karşılaştırır |
| `inspect-electron-fuses` | Paketlenmiş Electron fuse baytlarını, başlatmadan inceler |
| `inspect-electron-page` | Tek bir Electron dosya sayfasını pasif inceler |
| `list-electron-targets` | CDP uç noktasının açtığı yerel dosya sayfaları |
| `capture-electron-scenario` | Sahip olunan bir Electron senaryosunu iptal edilebilir eylemlerle çalıştırır |
| `list-javascript-runtime-targets` | Node/Electron V8 Inspector hedefleri |
| `observe-javascript-runtime` | Tek Inspector hedefini pasif gözlemler |
| `reconcile-javascript-runtime` | Statik uygulama ve pasif çalışma zamanı Evidence'ını uzlaştırır |
| `compare-source-to-bundle` | Tarihsel kaynağı, doğrulanmış uygulama Evidence'ıyla karşılaştırır |

### 4.8 Web

| Komut | Ne işe yarar |
|---|---|
| `analyze-web-bundle` | Sayfa bundle'ını yakalayıp statik analiz eder (varsayılan: hedef origin) |
| `inspect-web-page` | CDP ile sayfayı pasif inceler |
| `list-browser-targets` | Loopback CDP tarayıcısındaki sayfa hedefleri |
| `capture-browser-scenario` | Sınırlı bir Playwright senaryosu çalıştırır, Evidence döner |
| `capture-web-screenshot` | Origin kapsamında tek görünür viewport ekran görüntüsü |
| `compare-web-screenshots` | İki PNG artifact JSON'unu karşılaştırır |
| `compare-web-captures` | İki pasif yakalama / kayıtlı senaryoyu karşılaştırır |
| `export-web-scripts` | Saklanan site betiklerini boş bir dizine dışa aktarır |
| `observe-web-session` | Seçilen pencerede kullanıcı gezintisini gözlemler |
| `observe-web-execution` | Hassas sayfa yürütme gözlemi (sayaçları sıfırlar, optimizasyonu geçici kapatır) |
| `inspect-web-event-listeners` | İlk CSS eşleşmesindeki native listener kaynak konumları |
| `inspect-web-network-capture` | HAR, mitmproxy, Fiddler SAZ veya qlog kayıtlarını (URL'leri çekmeden) inceler |
| `trace-web-module-imports` | Yakalanan betiğin import'larını URL'lere ve kaynak adaylarına bağlar |
| `trace-web-source-location` | Betik konumunu yerel source map ile izler |
| `discover-webmcp-tools` | Sayfanın bildirdiği WebMCP araçlarını pasif keşfeder |

### 4.9 .NET (Managed / PE-CLI)

| Komut | Ne işe yarar |
|---|---|
| `inspect-managed-artifact` | PE/CLI kimliği, hedef kodu yüklemeden |
| `inspect-managed-members` | Üyeler, imzalar, CIL çapaları, `ldstr` string'leri |
| `inspect-managed-native-boundaries` | PInvoke ve native implementasyon sınırları |
| `verify-managed-native-boundaries` | P/Invoke bildirimlerini native Evidence'a karşı doğrular |
| `compare-managed-members` | Tam tip + ad + imza ile üye karşılaştırması (yalnız ad eşleşmesi sayılmaz) |
| `import-managed-reconstruction` | Decompiler'ın ürettiği yeniden yapımı statik kanıtlara karşı içe aktarır |
| `inspect-pe-resources` | PE kaynak kimlikleri, payload aralıkları, hash'ler, ikon grupları |

### 4.10 WASM, EVM, firmware, veri tabanı

| Komut | Ne işe yarar |
|---|---|
| `inspect-wasm-artifact` | Yerel WASM'ı WABT ile inceler |
| `compare-wasm-interfaces` | İki WASM'ın çekirdek arayüzlerini karşılaştırır |
| `inspect-evm-interface` | EVM bytecode'dan arayüz adayları çıkarır |
| `inspect-firmware-regions` | Firmware imza adaylarını çıkarmadan inceler |
| `extract-firmware` | Seçili firmware baytlarını çıkarır; doğrulanmış dosya ve köken döner |
| `inspect-sqlite-database` | SQLite şeması ve (isteğe bağlı) tablo satırları, yerel anlık görüntüden |

### 4.11 Ağ ve sinyal

| Komut | Ne işe yarar |
|---|---|
| `inspect-pcap-client` | tshark/capinfos'u, capture okumadan kontrol eder |
| `inspect-pcap-capture` | pcap/pcapng: uç noktalar, DNS, TLS SNI, protokol dağılımı |
| `inspect-packet-capture` | Mevcut capture'ı TShark ile inceler |
| `inspect-packet-fields` | TShark protokol/alan tanımlarında arar |
| `decode-signal-recording` | Mevcut sinyal kaydını sigrok ile çözer |
| `inspect-signal-decoders` | Kurulu sigrok decoder'larını arar |

### 4.12 JEB

| Komut | Ne işe yarar |
|---|---|
| `inspect-jeb-client` | Çalışan JEB MCP istemcisini, hedef açmadan inceler |
| `open-jeb-project` | Artifact veya `.jdb2` veritabanından JEB projesi açar/oluşturur |
| `list-jeb-units` | JEB proje birimlerini yol ve tipe göre listeler |
| `decompile-jeb-item` | Bir JEB tipini/metodunu sözde koda çevirir |

### 4.13 Evidence, karşılaştırma ve yeniden yapım (reconstruction)

| Komut | Ne işe yarar |
|---|---|
| `compare` | İki kanonik Evidence paketini karşılaştırır |
| `evidence-export` | Kanonik Evidence JSON'unu doğrulayıp atomik olarak dışa aktarır |
| `evidence-import` | Sınırlı yerel Evidence paketini doğrulayıp içe aktarır |
| `inspect-evidence-bundle` | Evidence paketinin keşif metadata'sını (tam veya seçili) inceler |
| `import-reference-source` | Kaynak ağacını yalnızca **tarihsel referans** olarak içe aktarır |
| `build-reconstruction-obligation-ledger` | Deterministik, kanıta dayalı yeniden yapım yükümlülük defteri sayfası |
| `evaluate-reconstruction-coverage` | Satır içi, fail-closed yeniden yapım kapsamını değerlendirir |

---

## 5. Hazır iş akışları (tarifler)

> [!note] Not
> Aşağıdaki adımlar komut sırasını gösterir. Her adımın argümanları için `rea <komut> --help` kullanın.

### 5.1 Bilinmeyen bir dosyaya ilk bakış

1. `rea inspect-artifact` → dosya türü + önerilen sonraki adımlar
2. `rea analyze` → genel bakış
3. `rea search` → ilginç string/URL/anahtar kelime
4. `rea xrefs` → o string'in nerede kullanıldığı
5. `rea function` veya `rea decompile` → ilgili kodu oku

### 5.2 Android APK analizi

1. `rea inspect-android-package` → manifest, izinler, bileşenler
2. `rea decode-android-resources` → string ve kaynaklar
3. `rea search-android-classes` → ilgili sınıf adı
4. `rea inspect-android-class` → üyeler
5. `rea inspect-android-method` → metodu decompile et
6. `rea trace-android-references` → kim çağırıyor?
7. (Flutter ise) `rea identify-flutter-build` → `rea inspect-dart-aot`

### 5.3 Cihaz üzerinde Android paketi alma

1. `rea list-adb-devices`
2. `rea resolve-adb-packages` → paket adını bul
3. `rea inspect-adb-package` → sürüm/kurucu
4. `rea pull-adb-package` → APK setini SHA-256 ile çek
5. Yukarıdaki APK analizine geç

### 5.4 Electron / JavaScript uygulaması

1. `rea inspect-electron-fuses` → güvenlik fuse'ları
2. `rea analyze-javascript-application` → statik yeniden kurulum
3. `rea recover-javascript-sources` → okunabilir modüller
4. `rea trace-javascript-semantics` → ilişkiler
5. `rea compare-application-versions` → sürümler arası fark

### 5.5 Web sitesi / bundle analizi

1. `rea analyze-web-bundle`
2. `rea export-web-scripts` → betikleri diske al
3. `rea trace-web-module-imports` ve `rea trace-web-source-location`
4. `rea inspect-web-network-capture` → HAR vb. kayıtları

### 5.6 .NET uygulaması

1. `rea inspect-managed-artifact` → kimlik
2. `rea inspect-managed-members` → üyeler ve string'ler
3. `rea inspect-managed-native-boundaries` → P/Invoke
4. `rea verify-managed-native-boundaries` → native taraf ile doğrula

### 5.7 İki sürümü karşılaştırma

1. Her sürüm için analiz çıktısını `evidence-export` ile kaydet
2. `rea compare` ile iki Evidence'ı karşılaştır
3. Uygulama grafikleri için `rea compare-application-versions`

---

## 6. Obsidian'da verimli kullanım

> [!tip] Çıktıyı doğrudan nota çevirin
> ```bash
> rea analyze <hedef> --format md > "Analizler/hedef-analiz.md"
> ```
> Sonra nota frontmatter ekleyin:
> ```yaml
> ---
> hedef: uygulama-adı
> tarih: 2026-10-15
> tags: [rea, analiz]
> ---
> ```

**Öneriler:**

- Her hedef için bir klasör açın: `Analizler/<hedef>/` içinde `00-özet.md`, `01-string-bulguları.md`, `02-fonksiyonlar.md`.
- Bulguları `[[wikilink]]` ile birbirine bağlayın (ör. `[[Hedef - Şifreleme Fonksiyonu]]`).
- Etiket önerileri: `#rea/android`, `#rea/electron`, `#rea/native`, `#rea/web`, `#rea/dotnet`.
- Uzun çıktılar için `--token-limit` kullanın ve parçaları ayrı notlara bölün.
- Çıktıdaki adresleri/hash'leri inline code ile yazın: `` `0x100003f40` ``.
- Evidence JSON'larını nota yapıştırmak yerine `evidence-export` ile dosyaya kaydedip nottan bağlantı verin.

**Örnek bulgu şablonu:**

```markdown
## Bulgu: <başlık>

- **Hedef:** [[Hedef Adı]]
- **Komut:** `rea xrefs ...`
- **Adres/Konum:** `0x...`
- **Kanıt dosyası:** [[evidence/xxx.json]]
- **Yorum:** ...
- **Sonraki adım:** ...
```

---

## 7. Yapay zekâ ajanıyla kullanım

- `rea mcp add` → REA'yı MCP sunucusu olarak kaydeder; `rea mcp doctor` ile doğrulayın.
- `rea --mcp` → doğrudan MCP stdio sunucusu başlatır.
- `rea --llms` / `rea --llms-full` → ajanlar için komut manifesti üretir.
- `rea setup` → ajan entegrasyonlarını ve skill dosyalarını kurar; `rea uninstall` ile geri alınır.

---

## 8. Sorun giderme

| Sorun | Çözüm |
|---|---|
| Komut çalışmıyor / sağlayıcı yok | `rea doctor`, sonra `rea providers` ve `rea capabilities` |
| Parametreyi bilmiyorum | `rea <komut> --help` veya `--schema` |
| Çıktı çok büyük | `--token-limit`, `--token-offset`, `--filter-output` |
| Eski sürüm | `rea update` |
| adb cihaz görünmüyor | `rea inspect-adb-client`, ardından `rea list-adb-devices` |
| Cutter komutu çalışmıyor | `rea list-cutter-sessions` ile canlı oturum var mı bak |
| JEB komutları başarısız | `rea inspect-jeb-client` ile MCP istemcisini kontrol et |

---

## 9. Komut seçme rehberi (hızlı arama)

- **"Bu dosya nedir?"** → `inspect-artifact`
- **"Uygulamaya genel bakış"** → `analyze` / `inspect`
- **"Bu string nerede kullanılıyor?"** → `search` → `xrefs`
- **"Bu fonksiyon ne yapıyor?"** → `function` / `decompile`
- **"APK içinde şu sınıf var mı?"** → `search-android-classes`
- **"Cihazdaki uygulamayı al"** → `pull-adb-package`
- **"Ağ trafiği ne konuşuyor?"** → `inspect-pcap-capture` / `inspect-web-network-capture`
- **"İki sürüm arasındaki fark"** → `compare` / `compare-application-versions`
- **"Canlı süreci izle"** → `instrument-with-frida` / `observe-native-calls`

---

#rea #rehber #tersine-muhendislik
