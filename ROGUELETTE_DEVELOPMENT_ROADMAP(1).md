# ROGUELETTE — Geliştirme Yol Haritası

## Belgenin amacı

Bu belge, Roguelette prototipinin hangi sırayla geliştirileceğini ve her aşamanın neyi kanıtlaması gerektiğini tarif eder.

Bu bir takvim değildir. Aşamalar, süre tahminlerinden çok doğrulanabilir çıktılara göre düzenlenmiştir. Amaç mümkün olduğunca erken oynanabilir bir sürüme ulaşmak, oyunun temel fikrini test etmek ve henüz ihtiyaç duyulmayan sistemlere yatırım yapmamaktır.

`ROGUELETTE_GAME_DESIGN.md` oyunun ne olduğunu, `ROGUELETTE_SOFTWARE_ARCHITECTURE.md` ise kodun nasıl düzenleneceğini tanımlar. Bu belge yalnızca geliştirme sırasının ve kilometre taşlarının doğruluk kaynağıdır.

---

## Genel yaklaşım

- Önce rulet ve skor kuralları doğrulanır, ardından arayüz bu kuralların üzerine kurulur.
- Her aşama mümkün olduğunca küçük, gösterilebilir ve test edilebilir bir çıktı üretir.
- Çarpanlar, hedef skorları ve modifier değerleri başlangıçta geçici kabul edilir.
- Karmaşıklık içerik miktarından değil, kuralların birbiriyle etkileşiminden doğmalıdır.
- Görsel cila, temel oyun döngüsünün eğlenceli olduğu doğrulanmadan önce önceliklendirilmez.
- Prototipin cevaplamadığı gelecekteki ihtiyaçlar için altyapı kurulmaz.

---

## Aşama 0 — Proje temeli

### Amaç

Godot projesini, oyun kuralları ile arayüzün birbirinden bağımsız gelişebileceği sade bir temel üzerinde başlatmak.

### Kapsam

- Temel klasör ve sahne yapısının kurulması
- Oyun kuralları, oyun akışı ve arayüz sınırlarının belirlenmesi
- Otomatik testlerin çalıştırılabileceği en küçük altyapının hazırlanması
- Kontrol edilebilir ve seed ile tekrar üretilebilir rastgelelik yaklaşımının kurulması
- Geçici görsellerle açılan sade bir başlangıç sahnesi

### Tamamlanma ölçütü

Proje sorunsuz açılır, temel testler çalışır ve ileride eklenecek oyun kuralları herhangi bir arayüze ihtiyaç duymadan test edilebilir.

---

## Aşama 1 — Rulet çekirdeği

### Amaç

Avrupa rulet çarkını ve sayılar arasındaki temel ilişkileri oyunun geri kalanından bağımsız olarak modellemek.

### Kapsam

- Geçerli rulet sayıları
- Çark üzerindeki gerçek fiziksel sıra ve komşuluklar
- Düzine, tek/çift ve yarı ilişkileri
- 0'ın temel sistem içindeki davranışı
- Bu kuralları doğrulayan otomatik testler

### Tamamlanma ölçütü

Herhangi iki sayı arasındaki tüm temel ilişkiler güvenilir ve açıklanabilir biçimde hesaplanabilir. Fiziksel komşuluk dahil olmak üzere rulet bilgileri tek bir doğruluk kaynağından gelir.

---

## Aşama 2 — Skor ve sonuç açıklaması

### Amaç

Bir bahis ile olası veya gerçek bir sonuç arasındaki skoru tek merkezden hesaplamak.

### Kapsam

- `ScoreResolver`
- `ResolutionReport`
- Beş temel skor ilişkisi
- Üst üste binen eşleşmeler
- Kolay değiştirilebilir prototip çarpanları
- Önizleme, gerçek spin ve ilerideki simülasyonlar için ortak hesaplama yolu

### Tamamlanma ölçütü

Aynı girdiler her zaman aynı toplam skoru ve aynı gerekçe dökümünü üretir. Skorun yalnızca kaç olduğu değil, neden oluştuğu da otomatik testlerle doğrulanır.

---

## Aşama 3 — İlk oynanabilir spin

### Amaç

Oyunun en küçük etkileşim döngüsünü oynanabilir hâle getirmek.

### Kapsam

- Rulet sayılarının sade bir arayüzde gösterilmesi
- Oyuncunun üç sabit chip yerleştirmesi
- Geçersiz bahis girişlerinin engellenmesi
- Olası sonuçlar için hover önizlemesi
- Çark sonucunun üretilmesi
- Spin sonucunun ve skor gerekçelerinin gösterilmesi

### Tamamlanma ölçütü

Oyuncu üç bahis koyabilir, olası sonuçları inceleyebilir, spin başlatabilir ve kazandığı puanın nedenlerini görebilir. Önizleme ile gerçek sonuç aynı skor sistemini kullanır.

---

## Aşama 4 — Round ve run akışı

### Amaç

Tek bir spini, başı ve sonu olan oynanabilir bir round yapısına dönüştürmek.

### Kapsam

- Bahis, spin, çözümleme ve sonuç aşamalarının yönetilmesi
- Round başına beş spin
- Toplam skor ve kalan spinlerin takibi
- Round hedefinin kontrol edilmesi
- Başarı, başarısızlık ve yeniden başlatma akışları
- Aynı seed ve oyuncu kararlarıyla mekanik sonuçların yeniden üretilebilmesi

### Tamamlanma ölçütü

Oyuncu bir roundu baştan sona oynayabilir. Sistem hedefe ulaşılıp ulaşılmadığını doğru biçimde belirler ve oyun durumları arasında geçersiz hareketlere izin vermez.

---

## Aşama 5 — Modifierlar ve ilerleme

### Amaç

Oyunun roguelike kimliğini oluşturan ilk build kararlarını temel döngüye eklemek.

### Kapsam

- Run başında modifier seçimi
- Başarılı round sonrasında modifier ödülü
- Farklı bahis yerleşimlerini teşvik eden küçük bir başlangıç modifier havuzu
- Modifierların merkezi skor sistemi üzerinden çözümlenmesi
- Sabit çözülme sırası ve tekrar tetikleme sınırı
- Modifier etkilerinin sonuç raporunda açıklanması

### Tamamlanma ölçütü

Oyuncunun modifier seçimi, chiplerini nereye koymak istediğini anlamlı biçimde değiştirir. Birden fazla modifier birlikte çalışabilir ve ortaya çıkan bütün skor etkileri açıklanabilir.

---

## Aşama 6 — Minimum oynanabilir prototip

### Amaç

Roguelette'in bütün temel vaadini baştan sona test edilebilen tek bir build içinde birleştirmek.

### Kapsam

- Başlangıç modifierı seçimi
- Birden fazla round boyunca yükselen hedefler
- Roundlar arasında modifier kazanımı
- Run'ın kazanma veya kaybetme ile sona ermesi
- Temel yönlendirme ve anlaşılır geçişler
- Hızlı yeniden başlatma

### Tamamlanma ölçütü

Yeni bir oyuncu, geliştirici müdahalesi olmadan bir run başlatabilir, bahislerini değerlendirebilir, modifier build'i oluşturabilir ve run'ı doğal sonucuna kadar oynayabilir.

Bu aşamanın sonunda prototip şu ana soruyu cevaplayabilecek durumda olmalıdır:

> Üç bahis yerleştirmek, sonucu okumak ve modifierlarla rulet sistemini giderek kendi lehine dönüştürmek eğlenceli mi?

---

## Aşama 7 — Oyun testi ve dengeleme

### Amaç

Yeni sistem eklemek yerine mevcut temel döngünün karar kalitesini, temposunu ve tekrar oynanabilirliğini ölçmek.

### İncelenecek konular

- Temel skor çarpanları
- Exact eşleşmenin diğer ilişkilerle birlikte skor üretmesi
- 0'ın davranışı
- Spin başına chip ve round başına spin sayısı
- Hedef skor eğrisi
- Başlangıç modifierının gücü
- Modifierların birbirleriyle etkileşimleri
- Baskın veya otomatikleşen bahis stratejileri
- Hover önizlemesinin anlaşılırlığı
- Spin ve sonuç gösteriminin temposu

Gerekirse aynı oyun kuralı kodunu kullanan headless simülasyonlardan denge analizi için yararlanılabilir. Simülasyon sonuçları, gerçek oyuncu testlerinin yerine değil, onları yönlendirmek için kullanılmalıdır.

### Tamamlanma ölçütü

Oyuncular bahis yerleşimleri arasında anlamlı seçimler yapabildiğini hisseder, sonuçların nedenlerini anlayabilir ve modifierlar sayesinde run boyunca stratejisinin değiştiğini görebilir. Temel döngüyü gölgeleyen belirgin bir baskın strateji veya ciddi tempo sorunu kalmaz.

---

## Aşama 8 — Sunum ve cila

### Amaç

Doğrulanmış oyun döngüsünü daha okunabilir, tatmin edici ve karakter sahibi hâle getirmek.

### Kapsam

- Çark ve chip animasyonları
- Skor geri bildirimi
- Ses ve müzik
- Modifier tetiklenme efektleri
- Arayüz hiyerarşisi ve okunabilirlik
- Geçişlerin ve bekleme sürelerinin iyileştirilmesi
- Gerekli erişilebilirlik seçenekleri

### Tamamlanma ölçütü

Sunum, mekanik bilgiyi saklamak yerine güçlendirir. Oyuncu bir spin sırasında ne olduğunu kolayca takip eder ve tekrarlanan aksiyonlar gereksiz bekleme yaratmaz.

---

## Kilometre taşları

### M1 — Kurallar doğrulandı

Aşama 0–2 tamamlanmıştır. Rulet ve skor sistemi arayüzden bağımsız, deterministik ve test edilebilirdir.

### M2 — Temel döngü oynanabilir

Aşama 3–4 tamamlanmıştır. Oyuncu bahis koyabilir, spin atabilir ve bir roundu bitirebilir.

### M3 — Roguelette prototipi tamamlandı

Aşama 5–6 tamamlanmıştır. Modifier seçimi ve yükselen hedeflerle baştan sona bir run oynanabilir.

### M4 — Oyun fikri doğrulandı

Aşama 7 tamamlanmıştır. Temel döngünün güçlü ve zayıf tarafları gerçek oyun testleriyle anlaşılmıştır.

### M5 — Sunulabilir build

Aşama 8 tamamlanmıştır. Doğrulanmış prototip, dışarıdan oyunculara rahatça gösterilebilecek seviyededir.

---

## Kapsam yönetimi

Prototipe dahil olan ve kapsam dışında bırakılan sistemler yalnızca `ROGUELETTE_GAME_DESIGN.md` içindeki **Scope Rules** bölümünde tanımlanır. Bu yol haritası kapsamı yeniden tanımlamaz; yalnızca onaylanmış kapsamın hangi sırayla geliştirileceğini gösterir.

---

## İlk geliştirme hedefi

İlk somut hedef **M1 — Kurallar doğrulandı** kilometre taşıdır.

Bu hedefe ulaşıldığında elimizde henüz görsel olarak gelişmiş bir oyun bulunmayacaktır. Buna karşılık rulet ilişkilerini doğru bilen, bütün skorları tek merkezden hesaplayan, sonuçlarını açıklayan ve ilerideki arayüz ile modifier sisteminin güvenle üzerine kurulabileceği sağlam bir mekanik çekirdek bulunacaktır.
