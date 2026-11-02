# Yapay zekâ destekli geliştirme yöntemi

> Bu belgede geçen dosya yolları (`CLAUDE.md`, `docs/ARCHITECTURE.md`, `ideas/` vb.) projenin özel (private) deposuna aittir.

Bu proje, tek geliştiricinin yapay zekâ ile nasıl **disiplinli ve doğrulanabilir** bir oyun yazabileceğinin bir denemesidir. Kod, **Claude Code** ile yazılır; karar ve ürün sahipliği geliştiricidedir. Oyunun kendisinde bir yapay zekâ modeli çalışmaz (çevrimdışı, kural tabanlı); yapay zekâ **geliştirme sürecinin** parçasıdır.

## İlkeler

1. **Kurallar repoda yaşar.** `CLAUDE.md` ve klasör bazlı kural dosyaları (`systems/`, `ui/`, `models/`, `data/`) yapay zekânın uyacağı mühendislik kurallarını yazılı tutar: statik tipleme, para her zaman `int`, rastgelelik her zaman tohumlu, UI'da oyun kuralı yok.
2. **Karar belgesi tek yerde.** Tüm mimari kararlar `docs/ARCHITECTURE.md` içindedir. Yapay zekâ belgenin tamamını okumaz, görevin dokunduğu bölümü okur. Oyun vizyonu (`docs/VISION.md`) ile çelişki görürse karar vermez, sorar.
3. **Fikir → onay → uygulama ayrı aşamalardır.** Fikirler `ideas/` altında durum klasörleriyle tutulur (aday, onaylı, park, reddedilen). Yalnızca geliştiricinin onayladığı fikir, geliştirici isteyince uygulanır. Yapay zekâ ürün kararını kendi vermez.
4. **Önce kabul ölçütü, sonra kod.** Her kural için, uç durumlar (yeniden yükleme, saat atlaması, reddedilen işlem) dahil test yazılır.
5. **Ölçmeden dengeleme yok.** Ekonomi değişiklikleri botlarla çalışan simülasyonda ölçülür; sonuç ARCHITECTURE belgesine yazılır.
6. **Kayıt uyumluluğu bozulmaz.** Kayıt formatı her değiştiğinde sürüm artar, göç adımı yazılır, gidiş-dönüş ve göç testi eklenir.
7. **"Çalışıyor" demeden önce çalıştır.** Test süiti ve betik denetimi (`check_scripts`) yeşil olmadan iş bitmiş sayılmaz; başarısız ya da atlanan adım raporda açıkça yazılır.

## Roller

| Rol | Görev |
|---|---|
| **Geliştirici (insan)** | Vizyon, ürün kararları, onay, tüm görseller |
| **Ana oturum (Lead Engineer)** | Uygular, test eder, karar verir, raporlar |
| **systems-analyst** (salt okunur ajan) | Para, kayıt, ekonomi gibi riskli değişikliklerden önce gerçek koddaki etkileşimleri ve riskleri çıkarır |
| **gameplay-designer** (salt okunur ajan) | Yeni ekran/akış öncesi oyuncu deneyimi, kural ve arayüz taslağı önerir |
| **creative-ideation** (salt okunur ajan) | Yalnızca istenirse yaratıcı mekanik keşfi |
| **Çapraz model incelemesi** | Para/kayıt/sahiplik gibi riskli değişiklikler sonunda ikinci bir modelin (Codex) bağımsız kod incelemesi |

Ajanlar salt okunurdur; öneri verir, karar vermez. Bulgular körü körüne benimsenmez, gerçek kodla karşılaştırılıp doğrulanır.

## Orkestrasyon: yalnızca değeceği zaman devret

Çoklu ajan kurulumunun en kolay yapılan hatası, her işe ajan çağırmaktır. Burada varsayılan tersidir: **ana oturum (Lead Engineer) işi kendisi yapar.** Şartname, kabul ölçütü, mimari, her dosya değişikliği, entegrasyon, testler ve son karar devredilmez. Kurallar `.claude/skills/orchestration` ve `CLAUDE.md` içinde yazılıdır.

Her ajan soğuk başlar, dosyaları yeniden okur, abonelik kotasından yer ve Lead'in raporu okuyup doğrulama maliyetini getirir. Bu yüzden devir yalnızca şu dört durumdan biri açıkça doğruysa yapılır:

1. **Bağımsız yargı gerekir (kodlamadan önce):** kayıt formatı, para, sahiplik veya ekonomi değişikliği için `systems-analyst`; yeni ya da belirgin değişen bir oyuncu ekranı için `gameplay-designer`.
2. **Ayrı doğrulama (önemli değişiklikten sonra):** ikinci bir modelle tek seferlik Codex incelemesi.
3. **Yalnızca sonucun önemli olduğu geniş arama:** çok dosyalık tarama için salt okunur `Explore`.
4. **Gerçekten bağımsız paralel iş:** ortak dosyası, açık kararı ve sırası olmayan kapsamlar (bu projede nadirdir, çünkü tüm düzenlemeler Lead'de kalır).

Hiçbiri geçerli değilse ajan çağrılmaz. "Kapsamlı", "çok parçalı" ya da "büyük görev" tek başına sebep sayılmaz.

### En küçük etkili ekip

- Karmaşık bir değişiklik en fazla **bir analist (öncesinde)** ve **bir Codex incelemesi (sonrasında)** alır. Ekran işlerinde buna bir kez `gameplay-designer` eklenir.
- Aynı soru iki ajana sorulmaz; aynı dosyaları okuyan iki ajan çalıştırılmaz.
- Ajanlar yalnızca kapsamları birbirinden ayrıksa paralel çalışır; aksi halde sırayla, ikincisi birincinin cevabını alarak.
- Codex **inceleyicidir, uygulayıcı değildir.** Yalnızca geliştirici isterse ya da Lead kendi denemelerinden sonra tıkanırsa uygulama için çağrılır.

### Brifing

Her ajana şunlar verilir: hedef, kesin kapsam (dosyalar, mimari bölümleri, fikir numarası), zaten bilinenler, **somut sorular** ve çıktı biçimi ("en çok 25 satır, en ciddi olan önce, her biri dosya:satır"). Dosyalar yapıştırılmaz, yol gösterilir. Gevşek bir brifing, özenli bir brifingin maliyetinden fazlasını takip sorularında harcatır.

### Modeller ve maliyet

- Ajanların modeli ve çaba seviyesi tanım dosyalarında (`.claude/agents/*.md`) sabittir: analist ve yaratıcı keşif Opus 5.5 (orta çaba), oyun tasarımcısı Sonnet 5.5 (yüksek çaba, daha sade brifingler). Çağrı sırasında değiştirilmez.
- İş türüne göre model seçilir: arayüz ve metin işleri Sonnet, kayıt/para/ekonomi Opus.
- Çok ajanlı bir plandan önce kota durumu bir kez kontrol edilir; sınıra yakınsa plan daraltılır ve sıralı çalışılır.

### Bulguların entegrasyonu

Her bulgu gerçek kodla karşılaştırılır, körü körüne benimsenmez. Ajanlar arasındaki çelişkiyi Lead çözer. **Geliştiricinin kararı olan konular** (açık sorular, çelişen fikirler) sessizce birleştirilmez, geliştiriciye sorulur. Onay iş akışı, testler, kayıt uyumluluğu ve commit yasağı, devredilen işe de Lead'in kendi işi gibi uygulanır.

### Bir aşamanın akışı (`/implement-phase IDEA-xxx`)

Tek komut, onaylı bir fikrin tek aşamasını uçtan uca yürütür:

1. **Kapsam:** fikrin durumu `APPROVED` değilse durur; açık soruları ve aşamayı okur.
2. **Bağlam:** yalnızca ilgili mimari bölümleri ve değişecek kod okunur, belgenin tamamı okunmaz.
3. **Çalışma ağacı:** `git status` ile geliştiricinin kaydedilmemiş değişiklikleri korunur.
4. **Kabul ölçütleri** kodlamadan önce yazılır; gerekirse analist ve/veya tasarımcı bir kez çağrılır.
5. **Belirsizlik:** geliştiricinin kararı gereken konu varsa tek net soru sorulur ve beklenir.
6. **Uygulama ve testler:** en küçük güvenli değişiklik, her kural için test.
7. **Hedefli testler, sonra tam süit (bir kez)**, kayıt değiştiyse göç ve gidiş-dönüş testleri, ekonomi değiştiyse simülasyon ölçümü.
8. **Codex incelemesi (yüksek riskliyse, bir kez)** ve doğrulanan bulguların düzeltilmesi.
9. **Kayıt:** fikrin durumu ve geçmişi güncellenir, mimari belgeye ölçülen sonuç yazılır, kısa rapor verilir ve oturum kapatılır.

Orkestrasyon becerisinin ilk hali [avenoxai/avenoxskills](https://github.com/avenoxai/avenoxskills) projesinden uyarlanmıştır; bu projenin kurallarına göre sadeleştirilmiş ve sıkılaştırılmıştır.

## Güvenlik ve kalite kapıları

- Geliştirici istemedikçe commit, push, reset veya geçmiş yeniden yazımı yapılmaz; kaydedilmemiş çalışma asla ezilmez.
- Para, sahiplik, kayıt ve saat kuralları için **bilerek kural bozma** testi: kuralı kasıtlı kırıp testlerin yakaladığını kanıtlar, sonra dosyalar birebir geri alınır.
- Her aşamanın sonunda tam test süiti, betik uyarı denetimi ve (arayüz değiştiyse) demo ekran görüntüsü kontrolü yapılır.
- Token ve maliyet disiplini: orta/büyük işler tek oturumda tek görev olarak yürütülür; iş türüne göre model seçilir.

## Neden bu yöntem

Yapay zekâ ile hızlı kod üretmek kolaydır; zor olan, üretilen kodun **büyüyen bir sistemde bozulmamasıdır**. Bu projede güven, yazılı kurallardan, deterministik tasarımdan, geniş test kapsamından ve ölçülebilir ekonomi simülasyonundan gelir. Yapay zekâ hız sağlar, kalite kapıları doğruluğu korur, ürün yönünü insan belirler.
