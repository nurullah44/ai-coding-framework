# Yazılım projesi temelleri v0 — seçenekler

## Büyük resim

Amaç, 0–5.000 kullanıcı aralığında büyürken yeniden yazma ihtiyacını azaltan, ajanların çalışmasını kanıtla denetleyen bir temel seçmek. Bu aralık için kapasite veya yeniden yazmama garantisi kaynaklarda yok.

Proje sahibinin seçtiği omurga **spec → ticket → build → review → ship**: ne yapılacağını yaz, doğrulanabilir işe böl, geliştir, incele, yayınla. Matt Pocock ve Alexey Grigorev bu akışa dayanak verir; ayrıntılı sıralamaları aynı değildir. Matt mimari kararları spec öncesinde olgunlaştırır; Alexey ürün spec'inden sonra teknoloji ve mimariyi seçtirir. [Matt][M-flow], [Alexey][A]

Lauren/pstack, uygulamadan önce kullanım örneklerini, veriyi ve parça sınırlarını belirginleştirir. Kuralı tutturma sırası: **mimari → tipler** (verinin izin verilen biçimleri) **→ lint/CI** (otomatik kod kuralları ve her değişiklikte çalışan kontroller) **→ davranış testleri → karar gerektiren konular için belgeler**. Mekanizmalar projeye kurulup çalıştırılmalıdır. Çalışan uygulamada eylem, sonuç ve yan etki gösterilmeden iş bitmiş sayılmaz. [Tasarım][P-arch], [yaptırım][P-correct], [kanıt][P-verify]

```text
İhtiyaç → ürün kapsamı ↔ mimari / sistem tasarımı → uygulamaya hazır spec
                       veri, güvenlik, ağ, kapasite, işletme kararları
spec → ticket → build → review + çalışan uygulamada kanıt → ship
         kabul/test ölçütleri                              ↓
         ←←←←←←←←← izleme, bakım ve öğrenme ←←←←←←←←←←←←←←
```

## Proje aşamaları

Omurga sabit; aşağıdakiler tasarımı ve kararları ne kadar ayrıntılı kaydedeceğimizi seçtirir. Birlikte kullanılabilirler. Sistem tasarımı; uygulama, veri tabanı, dış hizmetler ve aralarındaki veri akışını kararlaştırmaktır. Güvenlik, kapasite ve yayın kararları burada ele alınır; yalnızca son incelemeye bırakılmaz. [Ubl][design-doc], [OWASP][threat]

| Seçenek | Ne demek | Kim öneriyor | Ne zaman uygun |
|---|---|---|---|
| Kısa tasarım notu | Koddan önce amaçları, kapsam dışını, çözümü ve alternatiflerini kısa bir metinde toplarsınız. | [Malte Ubl / Google][design-doc] | Çözüm belirsizse veya önemli bir tercih tartışılacaksa; bariz küçük değişiklikte gereksiz olabilir. |
| Karar kaydı + basit şema | Önemli kararların nedenini ADR adlı kısa kayıtta, sistemin parçalarını C4 adlı çizim yaklaşımıyla gösterirsiniz. | [Michael Nygard][adr], [C4][c4] | Karar gerekçesini sonradan bulmak ve ajanlara anlaşılır sınırlar vermek için; tüm kararlar başlangıçta verilmez. |
| Shape Up'tan hafif planlama | Problemi ve ayrılacak süreyi önceden sınırlar, ayrıntılı işleri yaparken belirlersiniz. | [Ryan Singer / Basecamp][shape-size], [işlerin belirlenmesi][shape-handover] | Küçük ekipte toplantıları azaltmak için; özgün yöntem bütün işleri baştan biletlere bölmez. |

**v0 için kaynakların önerisi:** Ubl belirsizliğe göre belge miktarını ayarlamayı, Basecamp küçük ekipte toplantı ve süreç adımlarını azaltmayı öneriyor. Nygard'ın önemli kararları gerekçesiyle kaydetmesi bu hafif yaklaşıma eklenebilir. [Ubl][design-doc], [Basecamp][shape-size], [Nygard][adr]

**Ajan bunu nasıl tutturur:** pstack'in plan kontrolü kanıt adımlarının yazıldığını denetler; bu adımların gerçekten çalışması ayrıca canlı uygulama kanıtıyla gösterilir. [Plan][P-plan], [kanıt][P-verify]

## Mimari stili

Buradaki seçim, kaç ayrı uygulama yayınlayacağımızı ve iç sınırları belirler. “Tek uygulama” ile “modüllü uygulama” birbirini dışlamaz; modül, belli bir işin bilgisini ve davranışını toplayan parçadır.

| Seçenek | Ne demek | Kim öneriyor | Ne zaman uygun |
|---|---|---|---|
| Tek uygulama: monolit | Ürünün işlevlerini birlikte geliştirilen ve yayınlanan tek uygulamada tutarsınız. | [DHH][dhh-monolith], [Martin Fowler][monolith-first] | Yeni ve basit üründe; ayrı yayın süreçlerinin yükünü taşımadan başlamak için. |
| Sınırları belirgin tek uygulama: modüler monolit | Tek uygulamanın içinde iş alanlarına göre sınırlar ve açık erişim noktaları kurarsınız. | [Shopify][shopify] | Parçaların birbirine karışması değişiklikleri zorlaştırdığında; bilinmeyen ürün için haftalarca sınır tasarlamadan. |
| Bağımsız hizmetler: mikroservisler | Ürünü ayrı geliştirilen ve ayrı yayınlanan uygulamalara bölersiniz. | [Stefan Tilkov][tilkov] | İş alanı iyi biliniyorsa ve bağımsız ekiplerin paralel çalışması gerçek ihtiyaçsa; ek ağ ve işletme yükü getirir. |

**v0 için kaynakların önerisi:** DHH çoğu web ürünü için monoliti, Shopify başlangıçta sınırlı mimari yatırımı öneriyor; sınırlar ihtiyaç belirginleştikçe güçlenebilir. Tilkov'un farklı önerisi, bağımsız geliştirme ve bilinen iş alanı koşullarına bağlı. [DHH][dhh-monolith], [Shopify][shopify], [Tilkov][tilkov]

**Ajan bunu nasıl tutturur:** pstack önce kullanım örneği ve tiplerle sınırı kurar, başka modülün iç parçalarına yanlış erişimi derlemeyi durduran kurallarla engellemeyi ister. [Mimari][P-arch], [yaptırım][P-correct]

## Teknoloji seçimi

Bu karar dil ve araç sayısından çok, ekibin hangi araçları anlayıp işletebildiğiyle ilgilidir. Yeni aracın maliyeti yalnızca kurulum değildir; hata ayıklama, izleme ve bakım da eklenir. [McKinley][boring]

| Seçenek | Ne demek | Kim öneriyor | Ne zaman uygun |
|---|---|---|---|
| Az sayıda, olgun ve bilinen araç | Mevcut, davranışını ve arızalarını bildiğiniz araçlarla ürünü kurarsınız. | [Dan McKinley][boring] | Teknolojinin yeniliği ürünün asıl değeri değilse ve mevcut araçlar işi karşılıyorsa. |
| Gerekçeli yeni veya özel araç | Mevcut araçların neden yetersiz kaldığını gösterip belirli bir ihtiyaca yeni araç eklersiniz. | [Dan McKinley][boring] | Kazancı öğrenme ve işletme yükünü karşılıyorsa; yalnızca popüler olması yeterli gerekçe değildir. |

**v0 için kaynakların önerisi:** McKinley olgun araçlara ve az eklemeye öncelik veriyor; eski olmak tek başına iyi olmak demek değil. Kesin bir dil, uygulama çatısı, ürün listesi veya araç sayısı: **Kaynaklarda yok**. [McKinley][boring]

**Ajan bunu nasıl tutturur:** pstack'in tek desteklenen yol ilkesi ve yerelde/CI'da aynı kontroller seçilen kullanım biçimini korur; teknoloji seçimini kendiliğinden yapmaz. [Yaptırım][P-correct]

## Veri ve ölçek hazırlığı

İlk iki seçenek birlikte kullanılabilir; üçüncüsü belirli bir soruna eklenir. Kullanıcı sayısı tek başına kapasite ölçüsü değildir: eşzamanlı işler, veri hacmi ve yapılan işlemler önemlidir. Kaynaklar bu ürünler için ölçülmüş bir kapasite sunmuyor. [AWS][db-scale], [pstack ölçüm][P-benchmark]

| Seçenek | Ne demek | Kim öneriyor | Ne zaman uygun |
|---|---|---|---|
| İlişkisel veri tabanı, önce mevcut yapıyı iyileştirme | Veriyi bağlantılı tablolarda tutar, yük arttığında sorguları ve veri tabanı sunucusunun kapasitesini iyileştirirsiniz. | [AWS][db-scale], [ortak veri tabanı örneği: Shopify][shopify] | İlişkisel model ürüne uyuyorsa ve tek veri tabanının kaynakları yeterliyse; kapasite artışı kesinti gerektirebilir. |
| Kalıcı durumu uygulama dışında tutma | Kalıcı veriyi uygulamanın geçici belleğine veya yerel dosyalarına bağlamazsınız. | [Adam Wiggins / Twelve-Factor][processes] | Yeniden başlatma ve gerektiğinde ek uygulama kopyalarıyla büyüme için; baştan çok sunucu çalıştırmak gerekmez. |
| Gerektiğinde arka plan işi veya önbellek | Uzun işleri kullanıcı isteğinden ayırır veya sık okunan verinin kopyasını tutarak tekrar işi azaltırsınız. | [Heroku][jobs], [Amazon Builders' Library][cache] | Uzun süren işler ya da ölçülmüş okuma yükü varsa; kuyruk ve önbellek farklı sorunları çözer, bakım yükleri vardır. |

**v0 için kaynakların önerisi:** AWS mümkünse yönetilen veri tabanını ve mevcut kapasiteyi büyütmeyi, Twelve-Factor kalıcı durumu uygulama dışında tutmayı destekliyor. Belirli kullanıcı sayısında ikinci veri tabanı, kuyruk veya önbellek ekleme zorunluluğu: **Kaynaklarda yok**. [AWS][aws-startup], [büyütme][db-scale], [Twelve-Factor][processes]

**Ajan bunu nasıl tutturur:** pstack veri sahipliğini ve erişim biçimini önce tasarlar, tekrar çalıştırmanın güvenli olmasını ister; hız iddiasını gerçekçi veri ve eşzamanlı yükle ölçtürür. [Sahiplik][P-red], [veri][P-domain], [tekrar güvenliği][P-idempotent], [ölçüm][P-benchmark]

## Güvenlik temeli

OWASP Top 10 riskleri tanıtır; ASVS, doğrulanabilir güvenlik gereksinimleri listesidir. Düzeyi kullanıcı sayısıyla değil, tutulan verinin hassasiyeti ve olası zararla seçeriz. [OWASP][top10], [ASVS][asvs]

| Seçenek | Ne demek | Kim öneriyor | Ne zaman uygun |
|---|---|---|---|
| ASVS Düzey 1 ile başlangıç | Uygulamaya uyan temel güvenlik kontrollerini uygular ve doğrularsınız. | [OWASP ASVS][asvs] | Sınırlı hassas veri toplayan erken girişim örneğinde; küçük ürün olmak tek başına yeterli gerekçe değildir. |
| ASVS Düzey 2 hedefi | Temel kontrollerin üzerine daha geniş güvenlik gereksinimlerini eklersiniz. | [OWASP ASVS][asvs] | OWASP'ın çoğu uygulama için önerdiği hedefte; veri ve risk daha güçlü güvence gerektiriyorsa. |

Her iki seçimde tasarım sırasında “Ne ters gidebilir, nasıl önleriz, önlem çalıştı mı?” soruları sorulur. Kimlik doğrulama, kişinin kim olduğunu; yetkilendirme, hangi veriye ve işleme erişebildiğini belirler. Bakımı yapılan bir giriş kütüphanesi kullanmak ile dış kimlik sağlayıcısına bağlanmak ayrı tercihlerdir; OWASP ikinci yol için OIDC adlı giriş standardını önerir. [Tehdit değerlendirmesi][threat], [kütüphaneler][security-library], [giriş][auth]

Parola ve erişim anahtarları koda veya kayıtlara girmemeli; erişimleri, yenilenmeleri ve iptalleri ele alınmalı. Twelve-Factor ortam değişkenlerini önerirken OWASP paketlenmiş uygulamalarda bunların sızma riskini belirtir; dolayısıyla tek saklama yöntemi üzerinde ortak öneri yok. Kullanılan yazılım bağımlılıklarının açıkları da izlenmeli. [Yapılandırma][config], [gizli bilgiler][secrets], [bağımlılıklar][supply-chain]

**v0 için kaynakların önerisi:** OWASP çoğu uygulamada Düzey 2'yi hedeflemeyi öneriyor; Düzey 1 sınırlı hassas verili başlangıç için düşünülebilir. Ürünün veri ve risk bilgisi olmadan kesin düzey seçimi: **Kaynaklarda yok**. [ASVS][asvs]

**Ajan bunu nasıl tutturur:** pstack dış veriyi girişte doğrular; bağımsız inceleme giriş/yetki açıklarını, tehlikeli girdileri ve anahtar sızıntılarını arar, fakat ASVS'nin tamamını otomatik doğrulamaz. [Sınırlar][P-boundary], [inceleme][P-rubric]

## Ağ ve yayın

Bu seçim, uygulamayı nerede çalıştıracağımızı ve sunucu bakımını kimin üstleneceğini belirler. Yönetilen hizmet kullanmak, tüm güvenlik ve kurtarma ayarlarının hazır olduğu anlamına gelmez.

| Seçenek | Ne demek | Kim öneriyor | Ne zaman uygun |
|---|---|---|---|
| Yönetilen platform ve veri tabanı | Yayın, dış erişim ve sertifika gibi işleri sağlayıcının sunduğu hizmetlerle yürütürsünüz. | [Render örneği][render-web], [Kamal'ın yönlendirmesi][kamal] | Linux ve Docker (uygulamayı bağımlılıklarıyla paketleme aracı) bilgisi sınırlıysa; plan kapsamı kontrol edilir. |
| Tek sanal sunucu + yayın aracı | Kiralık sunucunun bakımını üstlenir, uygulamayı Kamal gibi bir araçla yayınlarsınız. | [Kamal][kamal] | Sunucu işletme bilgisi ve taşınabilirlik isteği varsa; güncelleme, veri tabanı ve yedek sorumluluğu sürer. |
| Kubernetes | Birden fazla sunucudaki paketlenmiş uygulamaların yerleşimini ve çalışmasını ortak sistemle yönetirsiniz. | [Kubernetes belgeleri][kubernetes] | Dağıtık iş yükü ve otomatik toparlanma ihtiyacı varsa; kümenin güvenlik ve bakım maliyeti karşılanabiliyorsa. |

HTTPS, yani şifreli web bağlantısı, veri tabanına yalnızca gereken makinelerin erişmesi ve en az yetki temel konulardır. Örneğin Render'da dış veri tabanı erişimi varsayılan olarak açık olabilir; iç bağlantıda tam sertifika doğrulama sınırı da OWASP önerisinden ayrılır. Sağlayıcının somut ayarları incelenmelidir. [OWASP ağ][db-security], [şifreli bağlantı][tls], [Render ayarları][render-db]

**v0 için kaynakların önerisi:** Kamal'ın kendi önerisi, Linux/Docker zor geliyorsa tam yönetilen hizmeti seçmek. Bu kullanıcı aralığında herkes için aynı sağlayıcı veya Kubernetes gerektiren sayısal eşik: **Kaynaklarda yok**. [Kamal][kamal], [Kubernetes][kubernetes-prod]

**Ajan bunu nasıl tutturur:** pstack çalışan uygulama kanıtını ister ve yayında insan kontrolü öngörür; genel ağ kurallarını kurup denetleyen hazır mekanizma: **Kaynaklarda yok**. [Kanıt][P-verify], [yayın sınırı][P-mode]

## İşletme/gözlem

İkinci seçenek ilkine eklenebilir; yedekleme her ikisiyle ayrıca ele alınır. Amaç, kullanıcı sorunu yaşarken bunu fark etmek ve kaybedilen veriyi kabul edilen sürede geri getirebilmektir.

| Seçenek | Ne demek | Kim öneriyor | Ne zaman uygun |
|---|---|---|---|
| Temel kayıtlar, hata bildirimi ve ölçümler | Olayları kaydeder; gecikme, trafik, hata ve kaynak doluluğunu izlersiniz. | [Twelve-Factor][logs], [Google SRE][monitoring] | İlk görünürlük için; sunucuya hiç ulaşmayan kullanıcı hatalarını ayrıca gözlemek gerekir. |
| Hizmet hedefi ve hata bütçesi | Kullanıcının önemli bir işi için başarı hedefi ve kabul edilen başarısızlık payı belirlersiniz. | [Google SRE][slo] | Güvenilirlik ile yeni özellik arasında karar vermek için; düşük trafikte tek hata oranı çok oynatabilir. |

Yedekte iki yol var: belirli anın veri kopyası veya işlem kayıtlarıyla seçilen ana dönme. İkincisi daha ayrıntılı kurtarma sağlar, daha çok kurulum ya da uygun yönetilen hizmet planı ister. Kabul edilebilir veri kaybını ve kurtarma süresini seçip veriyi ayrı yere geri yükleyerek içeriğini kontrol etmek gerekir; “yedek alındı” bildirimi kanıt değildir. [PostgreSQL kopya][dump], [zamana dönüş][pitr], [AWS kurtarma kanıtı][restore]

Arızada karar verecek kişi ve yapılanların kaydı da belirlenir; Google küçük olaylarda rollerin birleşebilmesine izin verir. [Olay yönetimi][incident]

**v0 için kaynakların önerisi:** Google ucuz ölçümlerle hizmet hedeflerine başlamayı, AWS kurtarma ihtiyacına göre yedekleri gerçekten geri yükleyerek sınamayı öneriyor. Evrensel hedef yüzdesi veya yedekleme takvimi: **Kaynaklarda yok**. [Google][slo], [AWS][restore]

**Ajan bunu nasıl tutturur:** pstack canlı kullanıcı akışlarını kanıtlar; üretim izlemesini, alarmları ve yedeklemeyi işleten hazır mekanizma: **Kaynaklarda yok**. [Canlı kanıtın kapsamı][P-verify]

## Test stratejisi

İki yaklaşım testlerin ağırlığını farklı dağıtır. Birim testi küçük bir davranışı, bütünleşme testi parçaların birlikte çalışmasını, uçtan uca test kullanıcının baştan sona işini sınar.

| Seçenek | Ne demek | Kim öneriyor | Ne zaman uygun |
|---|---|---|---|
| Test piramidi | Çok sayıda küçük test, arayüzün altındaki davranış testleri ve daha az uçtan uca test kullanırsınız. | [Mike Cohn][cohn], [Martin Fowler][pyramid] | Ekran üzerinden bütün akışı sınayan testler yavaş, kırılgan veya pahalıysa; küçük testler hızlı geri bildirim sağlıyorsa. |
| Testing Trophy: bütünleşme ağırlığı | Otomatik kod kontrolleriyle birlikte, parçaları kullanım biçimine yakın şekilde sınayan testlere ağırlık verirsiniz. | [Kent C. Dodds][trophy] | Özellikle JavaScript/arayüz çalışmaları ve kapsamındaki arka uç monolitleri için; öneri mikroservisleri kapsamıyor. |

Her iki yolda iş kuralları ve veri tabanı/dış hizmet bağlantıları sınanır; temel kullanıcı işleri için seçilmiş uçtan uca senaryolar eklenir. Güvenlik ve geri yükleme kanıtları da ayrıca gerekir. [Pratik test yaklaşımı][practical-tests], [ASVS][asvs], [kurtarma][restore]

**v0 için kaynakların önerisi:** Bu kullanıcı aralığı için tek kazanan, sabit test oranı veya kapsama yüzdesi: **Kaynaklarda yok**. Ortak yön, gözlenebilir davranışı sınamak ve önemli kullanıcı işlerini çalışan sistemde doğrulamaktır. [Fowler][pyramid], [Dodds][trophy], [pstack][P-testing]

**Ajan bunu nasıl tutturur:** pstack testin davranış bozulunca başarısız olmasını, canlı tarifin çalıştırılmasını ve birleştirmeden önce güncel değişikliği kapsayan bağımsız kullanıcı akışı kanıtını ister. [Test][P-testing], [canlı tarif][P-verify], [birleştirme][P-ship]

## Kaynaklarda olmayanlar

Buradaki eksikler genel yöntemin değil, bu ürüne verilecek somut yanıtların eksikliğidir; dayanak [A notu](<../research_notes/Yazılım projesi temelleri v0/A_classic_engineering.md>) ve [B notu](<../research_notes/Yazılım projesi temelleri v0/B_lauren_pstack_matt_design.md>).

- **Ürün yükü:** Eşzamanlı kullanıcı, veri hacmi, en yoğun işler ve kabul edilen yanıt süreleri: **Kaynaklarda yok**; sahibi hedefleri seçmeli.
- **Somut teknoloji ve bütçe:** Dil, uygulama çatısı, veri tabanı ürünü, sağlayıcı ve aylık bütçe: **Kaynaklarda yok**; ekip bilgisiyle seçilmeli.
- **Ürüne özel güvenlik:** Roller, müşteri verilerinin ayrılması, hassas veri kapsamı, giriş yöntemi ve buna uygun kesin ASVS düzeyi: **Kaynaklarda yok**.
- **Kurtarma ve hizmet hedefleri:** Kabul edilen veri kaybı, kesinti, yedek sıklığı, saklama süresi ve alarm sorumlusu: **Kaynaklarda yok**.
- **İnsan onayı ve yayın:** Bu projede mimari kararları kimin onaylayacağı, yayın yetkisi ve başarısız yayını geri alma tarifi: **Kaynaklarda yok**; pstack'in yayın duruşu bu tarifi sağlamaz.

## Önerilen v0 paketi

**Bu tablo kaynaklardan türetilmiş öneridir; proje sahibinin onayına veya değişikliğine sunulur.** Kaynakların ortak ve zorunlu tek paketi değildir. Güvenlik ve yayın seçimleri belirtilen koşullarla geçerlidir.

| Alan | Önerilen seçenek | Neden |
|---|---|---|
| Proje aşamaları | Sabit omurga + kısa tasarım notu + önemli karar kayıtları | Belirsizliği koddan önce azaltır, küçük ekibin belge yükünü sınırlar. [Ubl][design-doc], [Nygard][adr] |
| Mimari stili | Tek uygulama; ihtiyaç kadar belirgin modül sınırları | Yayın yükünü düşük tutarken parçaların birbirine karışmasını sınırlar. [DHH][dhh-monolith], [Shopify][shopify] |
| Teknoloji seçimi | Ekibin bildiği az sayıda olgun araç | Yeni araçların öğrenme ve işletme maliyetini azaltır. [McKinley][boring] |
| Veri ve ölçek hazırlığı | Uygunsa tek ilişkisel veri tabanı; kalıcı durum uygulama dışında | Mevcut kapasiteyi büyütme ve gerektiğinde uygulama kopyası ekleme yolunu açık tutar. [AWS][db-scale], [Twelve-Factor][processes] |
| Güvenlik temeli | Düzey 2 hedefi; sınırlı hassas veride Düzey 1 başlangıcını değerlendirme | Çoğu uygulama için OWASP yönünü izler; başlangıç düzeyini gerçek riske bağlar. [ASVS][asvs] |
| Ağ ve yayın | İşletme bilgisi sınırlıysa yönetilen platform ve veri tabanı | Sunucu bakım yükünü azaltır; ağ ve kurtarma ayarları yine kontrol edilir. [Kamal][kamal], [OWASP][db-security] |
| İşletme/gözlem | Temel ölçümler + basit hizmet hedefi + sınanmış geri yükleme | Sorunu görmeyi ve kurtarmanın gerçekten çalıştığını göstermeyi sağlar. [Google][slo], [AWS][restore] |
| Test stratejisi | Davranış ve bütünleşme testleri + seçilmiş canlı kullanıcı işleri | İş kurallarını ve kullanıcı sonucunu birlikte sınar; test dağılımı ürüne göre seçilir. [Vocke][practical-tests], [pstack][P-verify] |

[M-flow]: https://github.com/mattpocock/skills/blob/49dd158d1076134a641b33efb035946536778336/skills/engineering/ask-matt/SKILL.md
[A]: https://aishippingblog.com/p/ai-native-development-specifications
[P-arch]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/architect/SKILL.md
[P-correct]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/correct/SKILL.md
[P-verify]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/create-verification-skill/SKILL.md
[P-plan]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/multi-phase-plan.md
[P-domain]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-model-the-domain/SKILL.md
[P-red]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/architect/references/design-red-flags.md
[P-idempotent]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-make-operations-idempotent/SKILL.md
[P-benchmark]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/benchmark-checklist/SKILL.md
[P-boundary]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-boundary-discipline/SKILL.md
[P-rubric]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/interrogate/references/rubric.md
[P-mode]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/SKILL.md
[P-testing]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/principle-test-behavior-not-implementation/SKILL.md
[P-ship]: https://github.com/cursor/plugins/blob/f713f14203cd6420cec13cd7c55c32d598dbab68/pstack/skills/poteto-mode/playbooks/shipping.md
[design-doc]: https://www.industrialempathy.com/posts/design-docs-at-google/
[adr]: https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions
[c4]: https://c4model.com/diagrams
[shape-size]: https://basecamp.com/shapeup/4.1-appendix-02
[shape-handover]: https://basecamp.com/shapeup/3.1-chapter-10
[dhh-monolith]: https://signalvnoise.com/svn3/the-majestic-monolith-can-become-the-citadel/
[monolith-first]: https://martinfowler.com/bliki/MonolithFirst.html
[shopify]: https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity
[tilkov]: https://martinfowler.com/articles/dont-start-monolith.html
[boring]: https://mcfunley.com/choose-boring-technology
[db-scale]: https://aws.amazon.com/blogs/database/scale-your-relational-database-for-saas-part-1-common-scaling-patterns/
[aws-startup]: https://aws.amazon.com/blogs/startups/how-to-scale-to-10k-users-with-aws/
[processes]: https://12factor.net/processes
[jobs]: https://devcenter.heroku.com/articles/background-jobs-queueing
[cache]: https://aws.amazon.com/builders-library/caching-challenges-and-strategies/
[top10]: https://owasp.org/projects/Top-Ten
[asvs]: https://github.com/OWASP/ASVS/blob/5cf9b032440be53ce345ab3c130fda46ba1ce7a2/5.0/en/0x03-What-is-the-ASVS.md
[threat]: https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
[security-library]: https://top10proactive.owasp.org/archive/2018/c2-leverage-security-frameworks-libraries/
[auth]: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
[config]: https://12factor.net/config
[secrets]: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
[supply-chain]: https://top10.owasp.org/2025/A03_2025-Software_Supply_Chain_Failures/
[render-web]: https://render.com/docs/web-services
[kamal]: https://kamal-deploy.org/
[kubernetes]: https://kubernetes.io/docs/concepts/overview/
[kubernetes-prod]: https://kubernetes.io/docs/setup/production-environment/
[db-security]: https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html
[tls]: https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Security_Cheat_Sheet.html
[render-db]: https://render.com/docs/postgresql-creating-connecting
[logs]: https://12factor.net/logs
[monitoring]: https://sre.google/sre-book/monitoring-distributed-systems/
[slo]: https://sre.google/workbook/implementing-slos/
[dump]: https://www.postgresql.org/docs/18/backup-dump.html
[pitr]: https://www.postgresql.org/docs/18/continuous-archiving.html
[restore]: https://docs.aws.amazon.com/wellarchitected/latest/framework/rel_backing_up_data_periodic_recovery_testing_data.html
[incident]: https://sre.google/workbook/incident-response/
[cohn]: https://www.mountaingoatsoftware.com/agile/the-forgotten-layer-of-the-test-automation-pyramid
[pyramid]: https://martinfowler.com/bliki/TestPyramid.html
[trophy]: https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
[practical-tests]: https://martinfowler.com/articles/practical-test-pyramid.html
