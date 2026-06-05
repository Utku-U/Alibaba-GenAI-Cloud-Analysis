# Alibaba-GenAI-Cloud-Analysis

### Veri Setine Dair detaylı bilgi:
#### **GENTD26:** 

Alibaba cloud, içerisinde Database(vertabanları), Storage, Compute(Hesaplama, işlem güçleri), Networking ... barındıran cloud yapısıdır. GENTD26 verisi ise Alibaba cloud bünyesinde yalnızca AI modelleri üzerinde gerçekleştirilmek istenen GenAI odaklı işlemleri izlemek adına toplanmış veri setidir. Elimizde yüksek çözünürlükte görüntü oluşturmaya yarayan uçtan uca Stable Diffusion mimarisi var. Bu mimari, işlem esnasında Alibaba cloud içerisindeki GPU kullanır. GENTD26, 3 katman ile, uçtan uca mimarinin(fabrikanın tüm verimini, arızalarını ve yakıt tüketimini) kaydedilmiş bir **KARA KUTU / GÖZETLEME sisteminin** veri setidir. Verileri toplayan bu sistem Stable Diffusion GenAI kullanımına odaklanır. Bu veri seti, büyük ölçekli bir üretken yapay zeka sunucu sisteminin GENTD26 kapsamlı bir yukarıdan aşağıya görünümünü sunarak, üç mimari katmandaki performans verilerini yakalar: 

**<span style="color:red">Dikkat !!</span>** Yani Alibaba.com da alışveriş yapan bir kişi,sistem veritabanı izlenmez.

1. `Uygulama Katmanı(kullanıcı istekleri & uçtan uca gecikme):` Bu katman, sistemin "insanlarla temas eden" yüzüdür. GENTD burada şu soruyu sorar: "Kullanıcı ne kadar bekledi?

2. `Ara Katman(ağ geçidi kuyrukları & zamanlayıcılar & işlem hattı yöntemi):` Resim oluşturma emri GPU'ya gitmeden önce bu "ara durakta" bekler. GENTD burada sistemin yönetim verimliliğini ölçer.

3. `Altyapı Katmanı(Fiziksel Güç & Kaynaklar):` Burası "makine dairesidir". GENTD burada donanımın limitlerini takip eder.


#### **Peki bu veriye neden ihtiyaç duydu?**
(GENTD neden tüm verileri çekiyor? Eğer sadece Altyapı katmanını izleseydi, GPU nun %100 çalıştığı görünürdü. Ama kullanıcının neden 30 sn beklediğni anlayamazdık.)

**<span style="color:lightblue">Bu veri seti: </span>** GENTD26 verisinde, AliBaba cloud kümelerinde çalışan Stable Diffusion mimarisi kullanılan görüntü üretme GenAI işlemini içerir. GENTD26 verisi özellikle, Stable Diffusion mimarisi kullanımı ile elde edilen Görüntü üretme süreci verilerinin bulunması sebebi, GPU kullanımını en yoğun şekilde kullanan işlemler bunlardır. Henüz tam olarak keşfedilmeyen birçok iç görüler sunan bir veri setidir. (Gizliliği korumak amacıyla, orjinal veriler zaman damgası kaydırma, metrik ölçeklendirme ve tanımlayıcı karma işlemi gibi tekniklerle anonimleştirilirken, araştırma için gerekli olan dağıtım özellikleri ve korelasyonlar dikkatlice korunmuştur.) 


#### **Mimari**

Stable Diffusion (Görselden ve metinden görüntü oluşturmak için kullanılan en popüler yöntemlerden biridir) Mimarinin temel amacı: Görselleri ve Promptları matematiksel bir gürültü giderme(denoise) süreci ile yüksek çözünürlüklü ve anlamlı görsel haline getirmek.

- `Giriş(Noise & VAE):` Sistem tamamen rasgele bir gürültüyle başlar. VAE(Encoder) bu gürültüyü "Latent Space" (Gizli Uzay) denilen, hesaplaması daha kolay, sıkıştırılmış bir alana taşır.
- `Yönlendirme:` **CLIP** senin yazdığın yazıyı bilgisayarın anlayacağı matematiksel vektörlere çevirir. **ControlNet** ise görselin taslağını (poz, mimik vb.) belirler. 
- `İşleme(Base Model &LORA):` BaseModel(UNet), gürültünün içine bakıp burada bir insan yüzü olmalı diyen zekadır. LoRA ise bu zekaya özel bir stil (anime, gerçekçilik, yağlı boya vb.) katar.
- `Döngü (Repeated Iteration & Sampler):` Görseldeki geri dönüş oku kritiktir. Sampler, her adımda gürültüyü biraz daha temizler. Bu döngü 20-50 kez tekrarlanır.
- `Çıkış (Image):` Gürültü tamamen temizlendiğinde, VAE (Decoder) latent veriyi tekrar piksellere dönüştürerek final resmini oluşturur.

<img width="401" height="111" alt="MLoRA-Pipeline" src="https://github.com/user-attachments/assets/2bad09ca-4f2f-4d4e-a522-316fabdd89f7" />



#### **<span style="color:lightblue">Kullanıcı İstek çeşitleri:</span>**
Cloud sistemleri, farklı kullanıcı tiplerini tek bir altyapıda yönetmeyi amaçlar özünde. Bu kullanıcı tipleri:
- `End-Users:` Standart kullanıcılar(Arayüz kullanıcılar), doğrudan bulut altyapısı kiralamazlar. Araya bir SaaS katmanı girer. Baskın predict_type; Büyük çoğunluklar `TXT_2_IMG`, eğlence, merak veya basit tasarım içindir. Kaynak tahisisleri; bu hizmeti sunan AliBaba Cloud sisteminde çalışan arayüzler (ConfyUI vb.), temel ve tahmin edilebilir kullanıcı yükünü karşılamak için Rezerv ya da Serverless sunucular kullanır. Görüntü üretimi başarısız olmasın diye bu grupta spot kullanılmaz. 

Web sitesindeki butona basar $\rightarrow$ Gizli API $\rightarrow$ Gateway

- `Yazılımcılar ve Uygulama Geliştiriciler(Developers):` Bu kullanıcılar stable diffusion mimarisini kendileri eğlenmek için kullanmazlar. Amaçları kendi yaptıkları uygulamalara veya web sitelerine entegre etmektir. Bu kullanıcılar web arayüzü kullanmazlar. AliBaba Cloud'un AI stüdyosuna (Model Studio) bağlanıp API anahtarı(şifre gibi bir kod) alırlar. Kendi yazdıkları uygulamanın kodları arasına bu şifreyi gömerler. Baskın predict_type; `TXT_2_IMG`, `IMG_2_IMG`. Kaynak tahsisleri; Spot sunucular, bulut sağlayıcısının o an boşta duran kapasitesini %70-80-90'a varan indirimlerle sattığı "kesilebilir" (preemptible) sunuculardır. Modeller API ile yüksek hacimli çağırılır.

Python scripti çalıştırır $\rightarrow$ Doğrudan API $\rightarrow$ Gateway

- `Bulut ve Sistem Mühendisleri(DevOps/Cloud Architects):` AliBabanın analiz raporunu yazan ve sistemin çökmesini engelleyen asıl sistemdeki kişilerdir. Görsel üretmek ya da kod yazmakla ilgilenmezler. Tek amaçları sunucuların nasıl ayakta kalacaklar sorunusuna cevap ararlar. AliBaba Cloud içerisinde ECS(Sanal Sunucular), Serverless Kubernetes(ASK) ve GPU havuzlarını yönetirler. Sisteme aniden binlerce istek geldiğinde(Çoğunlukla End-Users kullanıcılar) bu mühendislerin kurdukları otomasyonlar devreye girerek boşta duran ekran kartlarında Stable diffusion mimarisini bağlarlar. Yoğunluk birince de o sunucuları kapatıp şirketin fazla para ödemesini engellerler. Sistemi denetlemek ve dolaylı yoldan GPU tüketimine neden olabilriler. Kaynak tahsisleri; Spot sunucular tercih edilir. 

Altyapı API $\rightarrow$ Gateway

- `AI araştırmacıları/Veri Bilimciler:` Stable Diffusion mimarisini hazır alıp kullanmakla yetinmeyen, onun "beynini" değiştirmek veya sıfırdan eğitmek isteyen akademik ya da kurumsal uzmanlardır. AliBaba Cloud'un PAI(Platform for AI) adını verdiği tamamen AI eğitmek için tasarlanmış devasa sistemlerin kullanımıdır. Web arayüzü yerine siyah kod ekranları ve jupyter notebook gibi veri bilimci araçlarla çalışırlar. Baskın predict_type; `TXT_2_IMG`, `IMG_2_IMG`. Kaynak tahsisleri; Tüm sunucular ile ilgilenirler. 

Yönetim ve Dosya API $\rightarrow$ Gateway

#### **<span style="color:orange">Kullanıcı İsteklerinden, Çıktıya Kadar İş Akışı:</span>**
GenAI modellerinden yararlanmak isteyen belirli kullanıcılar vardır. Bunlar: End-Users, Developers, Veri bilimciler ... Bu kullanıcılar `API Request` paketleri(kimlik, ihtiyaç listesi) gönderilir. Bu istekler önce External Load Balancer'dan geçerek isteklerin şifresi çözülür. Daha sonra API Gateway'e gönderilir ve paketin içerisine bakılır. API Gateway, gelen isteğin türüne göre ilgili mikroservise (Örn: Kimlik doğrulama veya GenAI çıkarım servisleri) yönlendirme yapar. Mikroservisi için hazır pod IP adreslerinin listesine bakar ve gelen isteğin Internal Load Balancer'a iletilip iletilmeyeceğine kara verir. Eğer sistem zaten ayakta ise istek Internal Load Balancer a iletilir ve oradan uygun podlara iletilir. Eğer sistemde bütün podlar dolu ise ya da hiç pod yoksa istekleri message queue de tutar "queue_size". İşte tam bu esnada KEDA karar verir ve yeni pod oluşturmak için Cloud sistemi devreye girer. Eğer sistemde hiç pod yok ise, **KEDA**  sistemi kuyrukta 1 mesaj gördüğü an ya da aşırı yoğunluk oluştuğunda kubernetes sayesinde Cloud sistemini pod üretmesi için tetikler. Bu sistem sayesinde ekran kartının dolmasını bile beklemeden **HPA'yı** doğrudan tetikler. 

**<span style="color:orange">Cloud Native sistemi:</span>**

DevOps mühendislerinin oluşturduğu YAML dosyasına göre kubernetesler sayesinde sistem mimarisi ayaklanır. YAML dosyaları, sistemin anayasasıdır. İçerisinde, imajın hangi scriptin hangi parametre ile başlayacağı, CPU, GPU kullanım kapasiteleri, pod artırımı koşulları bulunur. Kubernetes bu YAML dosyasını veritabanına kaydeder. Kubernetes schedular, İstekler ve YAML dosyaları(kanunlar) doğrultusunda bir node belirlenir. Uygun node belirlendikten sonra kubelet uyanır ve "benim üzerimde pod kurulacakmış" der. Pod, boşta bekleyen bir sandboxtur ve birden fazla container'ın bir arada yaşayabilmesi için oluşturulmuş mantıksal bir sınırdır, yuvadır. Donanımın gücü yettiğince birden fazla pod barındırabilir. Birden fazla pod oluşturulmak istenmesinin sebebi ise bir podda problem olduğunda diğerlerinde trafiğin karşılanmaya devam etmesidir. İçerisinde, imajdan gelecek kaynak kodları ve paketlerin izole alanı ve Persistent Volume disklerden gelecek model ağırlıklarının barınacağı yuva bulunur. Daha sonra imajların içersindekiler, Container registry ile Docker Hub içerisinden podlara gönderilir. İmaj, node içerisine bir kez indiğinde, node'un yerel diskinde saklanır ki eğer aynı node üzerinde 10 tane daha aynı podu açmak gerekir ise tekrar indirilmeye gerek kalmasın. Model ağırlıkları da Storage Mount(Depolama bağlama) ile podlara yerleştirilir. İşte, Container dediğimiz yapı, imaj içerisindeki paketlerin,öğelerin pod içerisinde indirilip çalıştırılmış halidir model ağırlıkları da bu container'ı çalıştıran yakıtlar gibidir. `Yani Container, imaj ve mode ağırlıkları bir tarif ise, container da o tarifle pişirilmiş ve pod içerisinde nefes alan canlı uygulamadır.` Artık kodlar, kütüphaneler, diskler hazır, IP tanımlı(Kubernetes IP atar). Ve her şey hazır olduğunda pod içerisindeki python script'i tetiklenir. Script, mount edilen diskteki Base Model'i alır ve GPU'nun belleğine (VRAM) yükler. Yükleme bitince Pod hazır olduğuna dair bir yeşil ışık yakar ve trafik alabilmesi için kubernetes pod'a hazır mısın? diye sorar. Evet derse, pod Internal Load Balancer'ın göreceği Endpoint listesine eklenir. İşte o zaman API Gateway'den sonra istekler Internal Load Balancer'a(çok sabırsızdırlar. İsteği hemen podlara götürmek isterler.) gider ve uygun pod'a istek gönderilir. Ayakta olan sisteme örneğin her biri farklı 100 LoRA istek atan kullanıcılar Pod IP'sine ulaştıklarında yani kapıya dayandıklarında S-LoRA/Punica gibi özel ara katmanlar devreye girer. Gelen LoRA'lar basemodel e entegre edilir. Ve bu işte `Generative Request` başlamış olur. 

Ayrıca, cloud sistem içerisindeki podları kontrol eden prometheus adaptörü gibi metric-server'lar GPU kullanımı kontorllerini sağlayıp raporlar oluştururlar. Bu raporları YAML dosyası(kanuna göre) ile karşılaştırıp yeni bir pod oluşturulup oluşturulmamasına karar verir ve **HPA** ile pod oluşturulur.

<span style="color:orange">Not:</span>Bu işlemlerin gerçekleşmesi için iki temel kubernetes tetikleyicisi vardır. Statik ve Dinamik tetikleyiciler:
- **Statik Tetikleyiciler(Admin):** Resources istek türlerinin yoğunluklarına göre, örneğin kullanıcılar için 100 podun hazır tutulması için kuberneteslerin tetiklenmesi. Yalnızca kullanıcıların modelleri kullanım zamanlarından önce hazırda bekletilmesi ile ilgili bir zaman optimizasyondan söz edilebilir.
- **Dinamik Tetikleyiciler(Traffivc):** Kubernetes içerisindeki gözcü mekanizmaların ölçtükleri değerlere göre, örneğin podların %80 terlemesi, kuyruk uzunlujları vb. göre kuberneteslerin tetiklenmesi. İhtiyaç yokken podlar kapatılarak enerji tasarrufu gibi tetikleyiciler mevcuttur. örn: HPA,KEDA

<img width="2048" height="1546" alt="WhatsApp Image 2026-06-05 at 18 00 33" src="https://github.com/user-attachments/assets/af4110e8-3963-4185-8da9-deb527c3b4d8" />


### Veri setine uygun olacak şekilde sistemin daha iyi anlaşılabilmesi adına yapılan çizim:

<img width="2048" height="1460" alt="WhatsApp Image 2026-06-05 at 18 00 32" src="https://github.com/user-attachments/assets/54eefbc4-b1d7-4851-842a-c5537fcfdac5" />





Analizi çalıştırma akışı:
Collected Data → Data_Informations → 1EDA → ...

### 📂 Veri Kaynağı & Teşekkür
Bu analizde kullanılan temel veri seti Alibaba GenAI Küme İzleme (v2026)'dır. Bu mimari analizi mümkün kılan, gerçek dünya üretken yapay zeka iş yükü izlerini açık kaynak olarak paylaşan bu veri setinin orijinal yaratıcılarına ve katkıda bulunanlarına teşekkür etmek istiyorum.

# -------------------------------------------------------------------------

Run flow:
Collected Data → Data_Informations → 1EDA → ...

### 📂 Data Source & Acknowledgments
The foundational dataset used in this analysis is the **Alibaba GenAI Cluster Trace (v2026)**. I would like to acknowledge the original creators and contributors of this dataset for open-sourcing real-world generative AI workload traces, which made this architectural analysis possible.

**Original Repository:** https://github.com/alibaba/clusterdata/tree/master/cluster-trace-v2026-GenAI
