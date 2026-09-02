[FreeBSD](https://freebsd.org) bilindiği üzere taş gibi sağlam, güçlü ağ desteği, sanallaştırma, "hapis" adı verilen `jail`, yani yazılımları yalıtma olanağı sağlayan özgür BSD lisansıyla
tescillenmiş BSD ailesine ait bir işletim sistemidir.

Bir haftadır elimde bulunan dizüstü bilgisayarlarımda FreeBSD'nin 15.10 sürümünü çalıştırmaya çalışıyorum. Burada nacizane tecrübelerimi elimden geldiğince havası estikçe
paylaşmaya çalışacağım.

Şu an elimde iki adet dizüstü bilgisayar bulunmaktadır.

* Xiaomi Redmi Book Pro 14 2024

Cihaz modelinden de anlaşılacağı üzere 2024 yılı serisi 14 inçlik ekrana sahip Çin iç pazarı için üretilmiş bir üründür. Microsoft Windows 11 Pro ve Microsoft Office yüklü gelmektedir.
Özelliklerine [üreticinin sayfasından](https://www.mi.com/redmi-books/14-pro/specs) ulaşabilirsiniz.

Benim cihaz gümüş rengi, Intel® Core™ Ultra 7 155H işlemciye, paylaşımlı Intel® ARC™ Graphics ekran kartına, 32GB LPDDR5x 7467MT/s anakarta gömülü RAM belleğe, 1TB PCIe 4.0 2242 NVMe SSD depolama aygıtına, NPU, TPM 2.0, parmak izi okuyucuya sahip.

Ben kendisini Intel donanıma sahip olduğu için Linux (R) 🐧 desteğinden dolayı seçtim ve açıkçası cihazdan da memnunum. Çin uyruklu satıcı cihazı Almanya'ya getirmiş,
Aliexpress üzerinden satmaktaydı. 12 Kasım 2024 tarihinde 2 222.44 BGN (Bulgar Lev'i) ücret ödedim (Avro cinsinden hesaplayınca 1 137.33 💶 oluyor). AB içi transfer
olduğu için ekstra gümrük ödemedim.

Aldığımda kararlı olan en güncel Debian sürümü cihazı tam desteklemiyordu, günümüzde ise sorun yok.

`sbctl` yazılımıyla donanım uyumlu çalışmaktadır. Arch Linux, Archman OS, Debian 13 destekleniyor, dolayısıyla UEFI Secure Boot özelliğini kapatmak gerekmemektedir.

* Dell Vostro 15 3510

Black Friday döneminde 18 Kasım 2022 tarihinde Ardes şirketinden sipariş ettim. Aldığım cihaz Ubuntu 20.04 LTS ile sertifakalanmıştı. 1299.00 BGN (€ 664.16) ücret karşılığında
gelen cihaz malaesef sorunlu BIOS yazılımına sahip ve malesef de güncellenemiyor.

Intel Core i5-1135G7 işlemciye, paylaşımlı Intel Iris Xe Graphics ekran kartına, 8 GB DDR4 2666 MHz RAM belleğe, 512GB M.2 NVMe SSD depolama birimine sahip olan cihaz 15.6 inç
ekranı vardır.

Garanti süresi dolar dolmaz BIOS'u patladı. Malesef kapatılınca ve yeniden başlatılınca hata LED'i yanıp sönmeye başlıyor, daha sonraki donanım testi açılınca hata bulamıyor.
Bu kapanma sorunundan dolayı BIOS yazılım da güncellenemiyor.

Cihazın ekranını sabitleyen vidalar ve kasaya bağlayan ayaklar yuvasından komple boşanıverdi. BIOS yazılımyla ilgisi olup olamadığını bilmiyorum, ekran uyku moduna girince
yahut ekran kapatma sinyali alınca ve rastgele olarak kararıyor. `i915` ve `xe` modeset yaparak malesef verim alınamıyor.

İçine ikinci yuvaya takılan depolama aygıtını da algılayamıyor. 1 TB HDD ve 256 GB SSD - ikisini de göremiyor.

İki cihaz da Intel WIFI aygıtına sahip.

İki cihazda da sevmediğim özellik: havalandırmayı sağlayan kasanın altındaki ızgara yuvalar malesef tümsekli biçimde değil, tamamen zemine sıfır biçimde. Xiaomi'de ızgaranın üst tarafında 
çok küçük bir çubukvari oluşum var, ama gerekli havalandırmayı sağlayabilir durumda değiller.
