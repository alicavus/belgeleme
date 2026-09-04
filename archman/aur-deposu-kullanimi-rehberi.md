_Not: Makale 16 Temmuz 2024 y. 20:51 TRT saatinde [Archman OS](https://archman.org/topluluk/belgeler/nasil-aur-deposu-kullanimi) bünyesinde yayımlanmıştır.
Söz konusu tarihten beri geçen süre içinde güncelliğini yitirmiş olabilir._

Konuya başlamadan evvel "depo" kavramına açıklık getirmek gerekir. İngilizce "repo", "repository" olarak karşımıza çıkan bu kavram kabaca yazılım yüklememize olanak sağlayan depolama anlamına gelmektedir.
Bu depolama kendi bilgisayarımızda mevcut olabilir (yerel, "local"), yahut uzak ("remote") diye tabir edebileceğimiz yerde. Yerel ağda da olabilir, uzak bir sunucuda da.
Depolama biçimini tartışmadığımız için genel bir ifadedir. Böyle bakınca github, sourceforge gibi platformlar da aslında birer depodur.

Dağıtım mantığından baktığımızda depo genelde paket yöneticisinin desteklediği depoları kapsamaktadır. Depo hazır derlenmiş ve paketlenmiş paketleri ihtiva edebilir,
yahut paketlemeyi mümkün kılan talimatnameleri içerebilir.

Arch Linux temelli dağıtımlarda paket yöneticisi pacman depoları  `/etc/pacman.conf` dosyasında tanımlanır. **Arch Linux sadece resmi olarak desteklediği depolardaki yazımlara desktek verir**.
Benzer biçimde Archman Linux kendi depolarına resmi destek sunar.

* AUR deposu resmi depo değildir. Kullanıcılar tarafından oluşturulan talimatnameler içerir. **Hazır halde derlenmiş paketler sunmaz**.
* AUR kullanıcılar tarafından oluşturulan depodur. Belirli kalite kontrol süreci beklense de, Arch Linux paketçiler tarafından moderasyona uğrasa da, talimatnamelerin kalitesi hakkında fikir sahibi
  olmamız için elle kontrol etmek zorundayız.
* AUR zararlı yazılım içerebilir.

> Dikkat! Bu anlatımımızda Arch Linux ve Archman Linux için anlatım yapılmaktadır. Diğer dağıtımlarda farklılık olabilir.

## I. AUR kullanımı için ön koşullar
AUR deposundan yazılım yükleyebilmemiz için `base-devel` paketinin yüklü olması gerekir. Bununla birlikte kaynak kodu yöneticisi / sürüm  kontrol sistemi `git` (ek olarak `svn`, `hg`, `bzr` gerekebilir),
İnternet'ten dosya indirmemize olanak sağlayan yönetici (`wget` vb, `curl` varsayılan olarak sisteminizde yüklü olması gerekir):  
```
$ sudo pacman -Syu
$ sudo pacman --needed base-devel git wget
```

Çalışan ağ bağlantınızın olması, AUR'a ulaşabilmenize bir mani olmaması gerekliliği, AUR altayapısının bakımda olmaması gibi temel ön koşulları dillendirmemize gerek bile yok.

AUR'a sürekli istek göndermemeye özen göstermeliyiz. Yani kendimizi bloklatmamaya özen göstermeliyiz. Sunucunun güvenlik koşullarını tetikleyecek yahut işleme kapasitesini kısıtlayacak davranışlardan sakınmalıyız.

## II. Web arayüzden paket indirip kurma
Tarayıcımızla [AUR ana sayfası](https://aur.archlinux.org/)na gittiğimizde "Paket ara" kısmına aradığımız paket adı / tanımı giriyoruz:  
![AUR Paket Arama](/archman/assets/aur-deposu-kullanimi-rehberi/aurwebpaketara.png)  
[AUR Paketler](https://aur.archlinux.org/packages) sayfasından da arama yapabiliriz:  
![AUR Paket Arama](/archman/assets/aur-deposu-kullanimi-rehberi/aurwebpaketara2.png)  
Dilediğimiz paket ismine tıklayıp "Paket ayrıntıları" sayfasına gidiyoruz:  
![Paket ayrıntıları](/archman/assets/aur-deposu-kullanimi-rehberi/aurpaketayrnt.png)  
Sağ üst tarafta bulunan "Paket Eylemleri" kutucuğunda "Anlık görüntüsünü indir" bağlantısına tıklayıp indiriyoruz:  
![Paket Eylemleri](/archman/assets/aur-deposu-kullanimi-rehberi/aurwebpakeylem.png)

Sayfayı alta doğru çevirdiğimizde (scroll ettiğimizde) "Bağımlılıklar" kısmındaki satırlarda sıralanan paketler arasında:
- Paket adının üzerinde `AUR` yazıyorsa, o paket sadece AUR'da vardır.
- Parantez içinde bulunan bağımlılıklar evveldeki paketin sağladığı özellikleri sağlar anlamındadır.
- Sonunda `(make)` yazıyorsa o paket sadece paketi inşa edilirken gerekli olduğu anlamıdadır.
- Sonunda `(optional)` yazıyorsa o paket sadece satırda anlatılan özellik için gerekli olduğu anlamındadır.

__Paket bağımlılıkları arasında sadece AUR'da bulunan bağımlılıklar varsa, evvela onları indirip, derleyip, inşa edip kurmalıyız__. Örneğin aşağıdaki görüntüde `libpamac-aur` olduğu gibi:  
![libpamac-aur](/archman/assets/aur-deposu-kullanimi-rehberi/aurpaketayrnt2.png)

Tarayıcımızla indirdiğimiz  sıkıştırılmış dosyayı ev dizininde uygun bir yere açıyoruz.  
Dosya yöneticimizle açtığımız dizine gidiyoruz, "Uçbirim'i buradan Aç..." (veya muadili neyse) seçeneğinden Uçbirim uygulamamızı (Terminal, Konsole her neyse) açıyoruz.  
```
$ makepkg -sci
```
komutunu giriyoruz.

\* Dilersek indirme, çıkartma ve kurmayı uçbirimden halledebiliriz:  
- Paket Ayrıntıları kısmındaki "Git Clone URL:" ile işaret edilen adresi kopyalayabiliriz. Bu durumda Uçbirimden
  ```
  $ git clone --depth=1 [kopyaladığımız adres]
  ```
  koşuyoruz. \[kopyaladığımız adres\] kısım kopyaladığımız sonu `.git`'li adres'tir. Örn:
  ```
  $ git clone --depth=1 https://aur.archlinux.org/pamac-aur.git
  ```  
- Paket Eylemleri kısımdaki "Anlık görüntüsünü indir" bağlantısını tarayıcımızdan kopyalayabiliriz:
  ```
  $ curl -o - [kopyaladığımız adres] | bsdtar -xf -
  ```
  \[kopyaladığımız adres\] kısım kopyaladığımız bağlantının adresidir. Örn:
  ```
  $ curl -o - https://aur.archlinux.org/cgit/aur.git/snapshot/pamac-aur.tar.gz | bsdtar -xf -
  ```
Bu anlatilan iki yöntemle paketi inşa etmek için gerekli talimatnameleri \[paket adı\] dizininde oluşturmuş oluyoruz.  
```
$ cd [paket adı]
$ makepkg -sci
```

Not: \[paket adı\] dizini önceden varsa sorun yaratabilir. Bu durumda ilkönce o dizini silin ve anlatılan iki yöntemden birini kullanın. Örn:  
```
$ rm -r pamac-aur
$ git clone --depth=1 https://aur.archlinux.org/pamac-aur.git
$ cd pamac-aur
$ makepkg -sci
```

## III. Grafik arayüzlü paket yöneticisiyle AUR paketi kurma
Genel olarak `pamac`, `octopi` yazılımlarını önermekteyim. İnternet'te sürekli yeni şeyler üretiliyor, dolayısıyla bunlar haricinde uygulamalar mevcut olabilir, onları da dilediğiniz gibi kullanabilirsiniz.

### pamac
> Archman Linux'ta pamac ön yüklü olarak gelmektedir.

Arch Linux'ta `pamac` kurulumu için bir önceki bölümde anlatılan yöntemle evvela `libpamac-aur` ve daha sonra da `pamac-aur` paketlerini AUR'dan indirip kurun.

`pamac-manager` uygulamasını (Yazılım  Ekle/Kaldır) çalıştırın.  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac0.png)

**AUR desteğini etkinleştirin:**

**Tercihler &gt; Üçüncü Parti &gt; AUR Depoları &gt; AUR desteği etkin** yolunu izleyerek desteği etkin değilse etkinleştiriyoruz:  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac1.png)  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac2.png)

Artık uygulamanın sol üstte bulunan arama merceğine tıklayarak AUR'dan paket aratabiliriz:  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac3.png)  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac4.png)  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac5.png)

Sonu `-bin` ile biten paketler uygulamayı kaynak kodundan derlemez, dolayısıyla paket oluşturma ve kurma daha hızlı olur. Dilediğimiz paketin indir seçeneğine tıklıyoruz ve "Uygula" diyoruz:  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac6.png)  

Çıkan pencerede oluşturulacak paket ve bağımlılıklar liastelenmektedir. Dilersek derleme dosyalarına göz atıp düzenleyebiliriz. "Uygula" diyerek paketimizin oluşturup yüklenmesini bekliyoruz:  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac7.png)  
![pamac](/archman/assets/aur-deposu-kullanimi-rehberi/pamac8.png)

### octopi
Eğer GTK+ yerine Qt temelli uygulama isterseniz octopi tam size göre. Resmi depolarda bulunmadığı için AUR deposundan `alpm_octopi_utils`,  `qt-sudo` ve `octopi` paketlerini indirip yükleyin.
AUR desteği sağlayan araçlardan `pacaur`, `paru`, `pikaur`, `trizen`, `yay` en az bir tanesi/dilediklerinizi AUR'dan yükleyin.

`octopi` uygulamasını çalıştırdıktan sonra **Araçlar &gt; Seçenekler &gt; AUR &gt; Octopi'nin kullanması gereken AUR aracını seçin** yolunu izleyerek daha önce kurmuş olduğunuz aracı seçin:  
![octopi](/archman/assets/aur-deposu-kullanimi-rehberi/octopi0.png)  
![octopi](/archman/assets/aur-deposu-kullanimi-rehberi/octopi1.png)

"Tamam" dedikten sonra AUR'da aramak için ana ekrandaki üstte yeşil uzaylı kafasına tıklıyoruz.  
![octopi](/archman/assets/aur-deposu-kullanimi-rehberi/octopi2.png)

Arama bölmesine AUR'da aradığımız paket adını yazıyoruz ve Enter tuşuna basıyoruz:  
![octopi](/archman/assets/aur-deposu-kullanimi-rehberi/octopi3.png)

Arama sonuçlarında dilediğimiz paket adına sağ tıklayıp "Yükle" diyoruz:  
![octopi](/archman/assets/aur-deposu-kullanimi-rehberi/octopi4.png)

Daha fazla paketi yüklemek için aynı şekilde yüklemek için işaretleyebiliriz.

Hazır olunca "Uygula" diyoruz:
![octopi](/archman/assets/aur-deposu-kullanimi-rehberi/octopi5.png)

Uygulama içi uçbirim açılıyor. Önergeleri takip ederek paketimizi kuruyoruz:  
![octopi](/archman/assets/aur-deposu-kullanimi-rehberi/octopi6.png)  
![octopi](/archman/assets/aur-deposu-kullanimi-rehberi/octopi7.png)  
<p>İşlem tamamlanınca paket adının yanındaki ikon yeşile dönecektir.


## IV. AUR desteği sunan Uçbirim araçlarıyla kurma
Bir önceki bölümde anlatılan araçlardanaraçlardan (`pacaur`, `paru`, `pikaur`, `trizen`, `yay`) dilediğinizi AUR'dan yükleyin. Başka araç da deneyebilirsiniz.

### yay
AUR'dan `yay-bin` paketini kurun.

AUR zengin seçenekleri sunar. Aşağıdaki tabloda bazı komutlar kısaca verilmiştir:

| Komut | Açıklama |
| --: | :-: |
| `yay` | `yay -Syu` komutuyla aynı anlamı taşıyor.|
| `yay <sorgu>` | Paket Yükleme için seçim menüsü sunar. |
| `yay -Bi <dizin>` | Bağımlılıkları kurar ve yerel `PKGBUILD` inşa eder. |
| `yay -G <AUR paketi>` | AUR'dan PKGBUILD dosyasını indirir. |
| `yay -Gp <AUR paketi>` | AUR'dan `PKGBUILD` dosyasının içeriğini yazar. |
| `yay -Ps` | Sistem istatistiklerini yazar. |
| `yay -Syu --devel` | Sistem yükseltmesi gerçekleştirir, ayrıca geliştirme paketleri güncellemeleri sınar. |
| `yay -Syu --timeupdate` | Sistem yükseltmesi gerçekleştirir, güncelleme tanımlamak için PKGBUILD düzenleme tarihine bakar. |
| `yay -Wu <AUR Package>` | Paket oylamasını iptal eder (`AUR_USERNAME` ve `AUR_PASSWORD` çevre değişkenleri tanımlaması gerektirir). |
| `yay -Wv <AUR Package>` | Paket oylar (`AUR_USERNAME` ve `AUR_PASSWORD`> çevre değişkenleri tanımlaması gerektirir). |
| `yay -Y --combinedupgrade --save` | Birleşik yükseltmeyi varsayılan olarak ayarlar. |
| `yay -Y --gendb` | Geliştirme güncelleştirmesi için kullanılan geliştirme paketi veritabanı oluşturur.| 
| `yay -Yc` | Gereksiz bağımlılıkları temizler. |

Uçbirimde `-h`, `--help` seçeneklerinden biriyle koşarak yardım alabilirsiniz. Ayrıca `man yay` komutunu koşun.

## V. Üçüncü taraf depo kullanma
Arch Linux topluluğunun üyelerinin bir kısmı özel depolarında AUR'da yararlı buldukları paketleri inşa edip yayınlamaktadırlar. Bu AUR'dan gelebilecek olası tehlikelerle birlikte ekstra
güvenlik zaafiyeti içermektedir. Dolayısıyla böyle deponun kullanılması tamamen kullanıcıların öz seçimidir.

> `/etc/pacman.conf` dosyasında pacman ayarlarıyla beraber depolar da tanımlanmaktadır.<br />
> Dikkat! Bu dosyadaki yapacağınız hasar `pacman`'i işlemsiz hale getirebilir. Dolayısıyla ilkönce yedekleyin:
> ```
> $ sudo cp /etc/pacman.conf /etc/pacman.conf.yedek
> ```
> Yedeğinizi geri yüklemek için:
> ```
> $ sudo mv /etc/pacman.conf.yedek /etc/pacman.conf
> ```
> koşun.

Arch Linux wikisinde <a href="" target="_blank" rel="noopener">[Gayrıresmi kullanıcı depoları](https://wiki.archlinux.org/title/Unofficial_user_repositories) maddesini inceleyin.

Örnek olarak Chaotic AUR deposunu ekleyelim:  
```
$ sudo pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com
$ sudo pacman-key --lsign-key 3056513887B78AEB
$ sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst'
$ sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'
$ cat << EOF | sudo tee --append /etc/pacman.conf
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
EOF
$ sudo pacman -Syu
```

## VI. AUR'un github yansısını kullanma

Bazen çeşitli nedenlerden dolayı AUR'a ulaşım kısıtlı olabilmektedir. Bu durumlarda AUR'un [github yansısını](https://github.com/archlinux/aur) kullanabiliriz. AUR'da barınan paketler ayrı dal (`branch`) olarak depoda eklidir.
Paket adının son halini indirmek için:
- `https://github.com/archlinux/aur/archive/refs/heads/<paket adi>.zip` yolunu takip edebiliriz:
  ```
  $ curl -sLo - https://github.com/archlinux/aur/archive/refs/heads/naps2-bin.zip | bsdtar xf -
  ```
- `git` klonlama yaparken `-b <paket adı>` / `--branch=<paket adı>` belirterek istediğimiz paketi indirebiliriz:
  ```
  $ git clone --branch=naps2-bin --depth=1 https://github.com/archlinux/aur.git naps2-bin
  ```

