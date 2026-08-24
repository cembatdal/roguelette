# ROGUELETTE — Yazılım Mimarisi ve Kodlama Kuralları

## Belgenin amacı

Bu belge, Roguelette prototipinin kodunu nasıl düzenleyeceğimizi tarif eder. Geliştirme sırasında biz veya yardım aldığımız başka biri mimari bir karar verirken bu kuralları referans almalıdır.

`ROGUELETTE_GAME_DESIGN.md` oyunun ne olduğunu anlatır. Bu belge ise kodun sorumluluklarını ve bu sorumlulukların birbirleriyle nasıl çalışacağını tanımlar. Geliştirme sırası `ROGUELETTE_DEVELOPMENT_ROADMAP.md` içinde tutulur.

Ana hedefimiz şudur:

> Skor kuralları, oyun akışı ve ekranda gösterilenler birbirinden ayrı olsun. Birini değiştirdiğimizde ilgisiz parçalar bozulmasın.

Bu kurallar gereksiz altyapı kurmak için kullanılmamalıdır. Prototip küçük, anlaşılır ve kolay değiştirilebilir kalmalıdır.

---

## 1. Oyunu üç ana sorumluluğa ayır

Kodun üç temel bölümü olmalıdır:

1. **Oyun kuralları:** Rulet ilişkilerini, modifier etkilerini ve skoru hesaplar.
2. **Oyun akışı:** Bahisleri, spinleri, roundları, hedefleri ve modifier seçimini yönetir.
3. **Arayüz:** Tahtayı, çarkı, önizlemeyi, animasyonları, sesleri ve menüleri gösterir.

Bu bölümler haberleşebilir fakat birbirlerinin işini yapmamalıdır.

Örnek:

```text
Oyuncu 34'e tıklar
→ UI bu isteği oyun akışına bildirir
→ Oyun akışı bahsin geçerli olup olmadığını kontrol eder ve kaydeder
→ Skor sistemi, 32 gelirse ne olacağını hesaplar
→ UI kendisine verilen sonucu gösterir
```

---

## 2. Skoru hesaplayan tek bir sistem olsun

Bütün skor hesapları aynı merkezi parçadan geçmelidir. Bu parçanın geçici adı `ScoreResolver` olabilir.

Girdi olarak şunları alır:

- yerleştirilmiş bahisler,
- gelen veya önizlenen sayı,
- aktif modifierlar,
- hesap için gereken round bilgileri.

Çıktı olarak yalnızca toplam puanı değil, puanın nasıl oluştuğunu da verir.

Örnek:

```text
Bahis: 34
Sonuç: 32

Aynı düzine:       +2
İkisi de çift:     +1
Aynı yarı:         +1
Modifier etkisi:   +3
Toplam:              7
```

UI hiçbir koşulda kendi skor hesabını yazmamalıdır.

Hover önizlemesi ile gerçek spin aynı `ScoreResolver`ı kullanmalıdır. Aynı girdiler verildiğinde ikisi de aynı sonucu üretmelidir.

---

## 3. Her sonuç açıklanabilir olsun

Skor sistemi yalnızca `7` gibi bir sayı döndürmemelidir. Bu sayının neden oluştuğunu anlatan bir sonuç raporu da üretmelidir. Bu yapının geçici adı `ResolutionReport` olabilir.

Bu raporu şunlar kullanabilir:

- hover sonuç önizlemesi,
- spin sonrası skor animasyonu,
- hata ayıklama,
- otomatik testler,
- denge simülasyonları,
- ileride eklenebilecek run geçmişi.

UI açıklamayı sonradan yeniden oluşturmaya çalışmamalıdır. Açıklama, hesap sonucunun doğrudan bir parçasıdır.

---

## 4. Oyun akışı yalnızca koordinasyon yapsın

`RunSession` veya `RunController` gibi bir parça mevcut run'ı yönetebilir:

```text
Bahisleri bekle
→ Çark sonucunu iste
→ Sonucu skor sistemine hesaplat
→ Kazanılan skoru ekle
→ Round hedefini kontrol et
→ Devam et veya modifier seçimine geç
```

Bu parça mevcut skor, kalan spinler, bahisler ve aktif modifierlar gibi run bilgilerini tutabilir. Fakat özel işleri ilgili parçalara bırakmalıdır:

- skor hesabını `ScoreResolver`a,
- rastgele sonucu RNG sistemine,
- kayıt işlemini kayıt sistemine,
- görsel davranışları UI'ya.

Her şeyi bilen ve yapan dev bir `GameManager` oluşturulmamalıdır.

---

## 5. Değişebilen her bilginin tek sahibi olsun

Oyunun mevcut durumunu belirleyen her bilginin yalnızca bir asıl sahibi olmalıdır.

Örnek:

- Güncel skor ve kalan spin sayısı `RunSession` tarafından tutulur.
- Yerleştirilmiş bahisler bahis durumunu yöneten parçada tutulur.
- Bir modifierın kalan kullanımı, o modifierın run içindeki örneğinde tutulur.
- UI bu değerleri gösterir fakat ikinci bir asıl kopya tutmaz.

Bu kural, iki farklı parçanın oyunun mevcut durumu hakkında farklı şeyler söylemesini engeller.

---

## 6. Modifierlar küçük ve bağımsız kurallar olsun

Her modifier anlaşılır bir şart ve etki tanımlamalıdır.

Örnek:

```text
Düzine eşleşmeleri iki kez skor üretir.
Tek sayı geldiğinde komşuluk bonusu iki katına çıkar.
17'ye konan chip, 0'a da konmuş gibi davranır.
```

Bir modifier doğrudan UI'yı, animasyonu veya ilgisiz başka bir modifierı değiştirmemelidir. Gerçekleşmesini istediği etkiyi skor sistemine bildirmeli; etkiyi skor sistemi uygulamalıdır.

Önerilen akış:

```text
Oyunda bir durum oluşur
→ Modifier kendi şartının sağlandığını görür
→ Uygulanacak etkiyi üretir
→ Skor sistemi etkiyi uygular ve sonuç raporuna yazar
```

Modifierların hangi sırayla çözüleceği açık ve sabit olmalıdır. Tekrar tetiklemelerin sonsuz döngü üretmesini engelleyen bir sınır bulunmalıdır.

Ortak şartlar ve etkiler ortaya çıktıkça küçük, tekrar kullanılabilir parçalara ayrılabilir. Fakat birkaç gerçek modifier yazılmadan bütün olasılıkları çözmeye çalışan genel bir modifier dili tasarlanmamalıdır.

---

## 7. Modifier tanımı ile run içindeki durumunu ayır

Bir modifierın değişmeyen içeriği Godot `Resource` dosyasında tutulabilir:

- ID,
- isim,
- açıklama,
- ikon,
- denge değerleri,
- davranış türü.

Run sırasında değişen bilgiler ayrı bir modifier örneğinde tutulmalıdır:

- stack sayısı,
- kalan kullanım,
- kaç kez tetiklendiği,
- geçici geliştirmeler.

Paylaşılan `Resource` dosyaları run durumu taşımamalıdır. Aksi hâlde bir yerde yapılan değişiklik başka modifier örneklerini yanlışlıkla etkileyebilir.

---

## 8. Oyun kuralları sahne ağacına bağlı olmasın

Skor ve oyun kuralı sınıfları normalde sade GDScript nesneleri olmalıdır. Gerçekten Godot yaşam döngüsüne veya sahne ağacına ihtiyaç duymuyorlarsa `Node` yapılmamalıdırlar.

Oyun kuralı kodu şunları yapmamalıdır:

- sahne ağacında bağımlılık aramak,
- label veya sprite güncellemek,
- animasyon veya ses oynatmak,
- doğrudan oyuncu input'u okumak,
- belirli bir ekranın açık olmasına güvenmek.

Böylece kurallar, UI açılmadan da çalıştırılabilir ve test edilebilir.

---

## 9. UI yalnızca oyuncunun isteğini iletsin ve sonucu göstersin

UI şu tür istekler gönderebilir:

```text
bet_requested(34)
preview_requested(32)
spin_requested()
modifier_selected(modifier_id)
```

Oyun akışı bu istekleri işler ve UI'ya güncel durum veya sonuç raporu verir.

UI bir şeyin nasıl göründüğüne karar verebilir fakat mekanik olarak ne anlama geldiğine karar vermemelidir.

Özellikle:

- Çark animasyonu sonucu seçmemelidir.
- Sonuç önce belirlenmeli, animasyon daha sonra o sonuca gitmelidir.
- Skor animasyonu hesabı yeniden yapmamalı, verilen raporu göstermelidir.

---

## 10. Bağlantılar görünür olsun

Ana sahne gerekli parçaları oluşturmalı ve birbirlerine açıkça bağlamalıdır. Bir sınıf ihtiyaç duyduğu parçayı global bir yerden aramak yerine dışarıdan almalıdır.

Sırf kolay erişmek için oyun durumu ve kural sistemleri global autoload yapılmamalıdır.

UI'nın oyuncu hareketlerini bildirmesinde signal kullanmak uygundur. Temel skor hesabının içinde ise doğrudan metot çağrıları genellikle daha kolay takip edilir.

Global bir `EventBus` kullanılmamalıdır. Böyle bir yapı, hangi parçanın neyi gönderdiğini ve kimin oyun durumunu değiştirdiğini görünmez hâle getirebilir.

---

## 11. Rastgelelik kontrol edilebilir olsun

Çark sonucunu üreten RNG sistemi UI'dan ve skor hesabından ayrı olmalıdır.

Bilinen bir seed ile çalıştırılabilmelidir. Böylece bir run veya hata aynı şartlarla yeniden üretilebilir. Kullanılan seed ve önemli sonuçlar kaydedilebilir olmalıdır.

Aynı seed ve aynı oyuncu kararları, oyun sürümü ve kural sırası değişmediği sürece aynı mekanik sonuçları üretmelidir.

Bir modifier gizlice kendi bağımsız rastgele sayı üreticisini oluşturmamalıdır. Rastgelelik, sisteme açıkça görülen girdiler üzerinden girmelidir.

---

## 12. Rulet bilgileri tek yerde tanımlansın

Rulet çarkıyla ilgili bilgiler `WheelModel` veya `RouletteRules` gibi tek bir yerde tutulmalıdır:

- geçerli sayılar,
- çark üzerindeki fiziksel komşular,
- düzine üyeliği,
- tek/çift bilgisi,
- yarı bilgisi,
- 0'ın özel davranışı.

Fiziksel komşuluk, sayısal yakınlığa göre değil gerçek Avrupa ruleti çark sırasına göre hesaplanmalıdır.

Bu bilgiler hem UI hem skor kodu içinde ayrı ayrı kopyalanmamalıdır.

---

## 13. Oyunun mevcut aşaması açıkça belli olsun

Oyun hangi aşamada olduğunu açık bir değerle tutmalıdır. Örneğin:

```text
BETTING
SPINNING
RESOLVING
SHOWING_RESULT
CHOOSING_MODIFIER
RUN_ENDED
```

Oyuncu hareketleri bu aşamaya göre kabul veya reddedilmelidir. Geçersiz bir hareketi engelleyen tek şey animasyon süresi veya gizlenmiş bir buton olmamalıdır.

---

## 14. Test ve simülasyona uygun tasarla

Skor sistemi herhangi bir sahne çizmeden çalıştırılabilmelidir.

En azından şu konular test edilmelidir:

- beş temel eşleşme türü,
- fiziksel komşuluk haritası,
- 0'ın davranışı,
- üst üste binen eşleşmeler,
- modifier etkileşimleri,
- modifier çözülme sırası,
- tekrar tetikleme sınırı,
- önizleme ile gerçek hesabın eşitliği,
- bilinen seed ile aynı sonucun üretilebilmesi.

İleride headless bir simülasyon aracı, oyunun gerçek kural kodunu kullanarak çok sayıda spin veya run çalıştırabilmeli ve denge analizi için çıktı üretebilmelidir.

Simülasyon için kurallar başka bir dilde yeniden yazılmamalıdır. Aynı kuralın iki farklı uygulaması zamanla birbirinden sapabilir.

---

## 15. Önerilen klasör yapısı

```text
src/
  domain/
    roulette/       # Çark bilgileri ve eşleşme hesapları
    scoring/        # Skor girdisi, sonuç raporu ve ScoreResolver
    modifiers/      # Modifier kuralları ve etkileri
    run/            # Run ve round durumu

  application/      # Bahis koyma, önizleme, spin ve modifier seçme işlemleri
  presentation/     # Godot sahneleri, UI, animasyon ve ses
  infrastructure/   # RNG, kayıt sistemi ve dışarıya veri aktarma
  content/          # Modifier Resource dosyaları ve denge değerleri

tests/              # Otomatik kural testleri
tools/simulation/   # Headless denge simülasyonu
```

Klasör isimleri geliştirme sırasında değişebilir. Önemli olan sorumlulukların birbirinden ayrı kalmasıdır.

---

## 16. Kaçınılacak yapılar

Şunları oluşturmaktan kaçın:

- bütün oyunu yöneten tek bir script,
- UI içinde tekrar yazılmış skor hesabı,
- birçok dosyaya dağılmış modifier kontrolleri,
- sahneleri doğrudan değiştiren oyun kuralı nesneleri,
- paylaşılan `Resource` içinde tutulan run durumu,
- global singletonlardan gizlice alınan bağımlılıklar,
- sınırsız kullanılan global signal bus,
- mekanik sonucu belirleyen animasyon,
- önizleme, gerçek skor ve simülasyon için ayrı kural uygulamaları,
- mevcut prototipte bulunmayan hayalî sistemler için kurulmuş altyapı.

Loose coupling, her fonksiyonu ayrı bir sınıfa bölmek anlamına gelmez. Bir parça; farklı bir görevi varsa, farklı bir nedenle değişecekse veya bağımsız test edilmesi anlamlıysa ayrılmalıdır.

---

## 17. Yeni bir değişiklik öncesinde sorulacak sorular

Yeni bir sistem veya özellik eklemeden önce şunları sor:

1. Bu bilginin sahibi hangi parça?
2. Bu bir oyun kuralı mı, oyun akışı mı, yoksa UI işi mi?
3. Mevcut `ScoreResolver` ve `ResolutionReport` kullanılabilir mi?
4. Bu değişiklik aynı bilgi için ikinci bir doğruluk kaynağı oluşturuyor mu?
5. UI açılmadan test edilebilir mi?
6. Bunu değiştirmek ilgisiz sistemleri de değiştirmemizi gerektiriyor mu?
7. Bu gerçekten mevcut prototip için gerekli mi?

Bilginin sahibi veya görevin ait olduğu bölüm net değilse kodlamaya başlamadan önce tasarım netleştirilmelidir.

---

## Yardım alınırken kullanılacak kısa talimat

Bir geliştiriciden veya yapay zekâdan kod, mimari ya da refactor yardımı alınırken şu talimat verilebilir:

> Önerini `ROGUELETTE_GAME_DESIGN.md` ve `ROGUELETTE_SOFTWARE_ARCHITECTURE.md` ile uyumlu hazırla. Skor hesabını UI'dan ayır; önizleme, gerçek spin ve simülasyonda aynı kural kodunu kullan. Oyun durumunun tek sahibi olsun. Modifierlar ilgisiz sistemleri doğrudan değiştirmesin. Global GameManager veya EventBus oluşturma. Mevcut prototipin gerektirmediği altyapıyı ekleme. Önerin bu kurallardan biriyle çelişiyorsa bunu açıkça belirt.

---

## Kısa özet

Roguelette tek, deterministik ve açıklanabilir bir skor sistemine sahip olmalıdır. Oyun akışı uzmanlaşmış parçaları koordine etmeli, bütün işleri kendisi yapmamalıdır. UI yalnızca oyuncu hareketlerini iletmeli ve sonuçları göstermelidir. Modifierlar küçük ve bağımsız kurallar olmalı, etkileri merkezi olarak uygulanmalıdır. Değişen her bilginin tek sahibi bulunmalı; aynı oyun kuralı önizleme, gerçek spin, test ve simülasyonda yeniden kullanılmalıdır.
