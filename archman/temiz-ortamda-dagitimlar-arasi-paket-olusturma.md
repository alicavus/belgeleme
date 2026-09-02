_Not: Makale 15 Eylül 2023 s. 16:39 TRT saatinde [Archman OS](https://archman.org/topluluk/belgeler/nasil-temiz-ortamda-dagitimlar-arasi-paket-olusturma/) bünyesinde yayımlanmıştır.
Söz konusu tarihten beri geçen süre içinde güncelliğini yitirmiş olabilir._

**systemd** tartışmasız çok büyük kolaylıklar sağlamaktadır. Bu nimetlerden bir tanesi de **systemd-nspawn** adlı yazılımdır. Bu yazılım çeşitli `systemd` tabanlı dağıtımı hafif sanal makine konteynırında çalıştırır.

Arch Linux ve Archman OS varsayılan olarak `systemd-nspawn` yazılımıyla yüklü geldikleri için kullanıma hazır haldedir.

### I. Paket oluşturma
Genelde dağıtımlar sisteme iki şekilde yazılım eklerler: hazır derlenmiş yazılımın uygun biçimde paketlenmiş halini uzak sunucudan indirip sisteme açarlar ya da kaynak kodundan bilgisayarda oluşturulan derlenmiş
yazılımı sisteme eklerler.

`make` gibi araçlarla kaynaktan yazılım derlemek meşakkatli ve vakit alan bir uğraştır. Elle çekirdek veya sürücü derlemek zorunda kalanlar bunu az çok bilirler. Bu zorluğu aşmak için, yazılımlar arası bağlantıları daha
kapsamlı ve sistematik hale getirmek için dağıtım geliştiricileri paket yöneticisi adı verilen yazılımları kullanırlar.

Debian temelli dağıtımlar `apt` ve `dpkg`, Redhat temelli dağıtımlar `yum`, `dnf` ve `rpm` kullanırlar. Arch Linux ve bu tabanı kullanan dağıtımlar `pacman` kullanırlar.

Paketler çeşitli şekilde sıkıştırılır ve içlerine kontrol sağlaması yapmak için çeşitli bilgiler eklenir.

### II. Paket oluşturmada temiz ortamın önemi
Paket oluşumunda temiz ortam önemlidir. Derleme esnasında derleme makinasında istemsiz yere farklı yazılımlarla etkileşen paket bileşenleri istenmedik sonuçlara yol açabilir. Sizin oluşturduğunuz paket sizin makinenizde
çalışıp diğer cihazlarda soruna yol açabilir. Diyelim ki `A` yazılımı `B`, `C`, `D` yazılımlarına ihtiyaç duysun. Sizde `D` yazılımı kurulu olsun, bu durumda `B` ve `C` yazılımlarını kurunca `A`
yazılımı çalışabilir durumda olcaktır. Ne var ki `D` yazılımı kurulu olmayan başka bir makinede `B` ve `C` yazılımı kurmak `A` yazılımını çalışır hale getirmeyecektir.

Bununla beraber güvenlikle ilgili sorunlar ortaya çıkabilir.

Tabi eski derlemelerden ve paket oluşumlarından kalan artıklar eğer temizlenmediyse oluşan paketimizde gereksiz artıklar hatta sorunlara sebep olabilir.

İşte bundan dolayı paket oluşturma ortamı temiz olmalı, yani en temel bileşenlerden oluşmalı.

### III. chroot ortamında paket oluşturma
Belirli dizine bir dağıtımın temel bileşenlerini kopyalayalım. Bu dizini kök dizin ayarlayan ortamda (`chroot` ortamında) paket oluşturabiliriz. İşimiz bittiğinde söz konusu dizini silebiliriz.

### IV. konteynır ortamında paket oluşturma
Konteynırlar geleneksel tüm sistemi sanallaştırmaya göre daha hafif ve hızlı oldukları için tercih edilebilirler. Geçen sene Docker hizmetlerine çeşitli kota sınırları getirildi bu da beraberinde yapılabilecek işlere
sınırlama getirdi.  
`qemu`, `Virtualbox`, `vmware` gibi yazılımlar elbette platformlar arası derleme sağlamak için kullanılabilirler.

Tüm bu `chroot` ortamı, konteynır ve sanallaştırma tekniğini bir anda sunan yazılım var. Evet `systemd`'nin `nspawn` yazılımı.

`systemd-nspawn` yukarıda sözü geçen kolaylıkları sağlayabilir. `--boot` seçeneği kullanarak canlı olarak çalıştırabilir, anlık kayıt alabilir, çalışan sanal makineleri kapatıp açabiliriz.

Herşey `systemctl` ile ayarlandığı için kolayca sistem açılışında istediğimiz makineleri çalışır yapabiliriz.

Paketlemek istediğimiz dağıtımların temel bileşenlerini içeren dizinler oluşturabiliriz.

Örneğin Debian GNU/Linux ve Ubuntu için debootsrap kullanabiliriz:
```
$ sudo debootstrap --arch=amd64 --variant=minbase stable [dizin] http://ftp.linux.org.tr/debian/
```

#### **Arch Linux için paket yapımı:**
İlkönce Arch Linux temel bileşenlerimizi bir dizinine yükleyelim:  
```
$ sudo pacstrap -K -c [dizin] base base-devel
```
Geleneksel `chroot` yerine `systemd-nspawn` kullanacağız:  
```
$ sudo systemd-nspawn -D [dizin]
```
Bizi konteynırın `root` kullanıcısı karşılayacaktır. `root` kullanıcısının şifresini ayarlayalım:  
```
# passwd
# exit
```
Şimdi bu dizinden boot edip paket oluşturacak olan paketçi kullanıcımızı ekleyelim:  
```
$ sudo systemd-nspawn -b -D [dizin]
```
Sistem çalışınca oluşturduğumuz şifreyle oturumu açıyoruz. Oturum açınca paketçi kullanısını ekliyoruz ve şifresini de belirliyoruz:  
```
# useradd -mUG adm,log,sys,systemd-journal,wheel -s /bin/bash -c "Paketçi Kullanıcı" paketci
# echo "%wheel ALL=(ALL:ALL) ALL" > /etc/sudoers.d/20-wheel
# passwd paketci
```
Artık hazırız.  
```
# exit
```
diyerek çıkıyoruz.

Arch Linux için paket oluşturmaya kalktığımızda oluşturduğumuz Arch Linux sanal makineyi çalıştırıp paketci kullanıcımızla oturum açıp paket oluşturabiliriz. Temel sistemimize zarar vermemek için `-x` ya da `--ephemeral`
ile koşuyoruz:  
```
$ sudo systemd-nspawn -b -x -D [dizin]
```

`-x` veya `--ephemeral` ile çalıştırılınca dizinimizin geçici kopyası oluşturuluyor. Aşağıdaki örnekte Debian geçici konteynırı çalıştırılmış durumda:  
![Screenshot-from-2023-09-15-16-13-14](/archman/assets/Screenshot-from-2023-09-15-16-13-14.png)

Oluşan kopyaya istediğimiz dosyaları çift taraflı kopyalayabiliriz.

Örneğin AUR'den `naps2-bin` paketini oluşturalım:  
```
$ sudo systemd-nspawn -b -x -D [dizin]
login: paketci
```
Açılan kabuk ortamında:  
```
$ sudo pacman -Syu
$ sudo pacman -S git
$ git clone -b naps2-bin --depth=1 https://github.com/archlinux/aur.git naps2-bin
$ cd naps2-bin
$ makepkg -s
```

Sanal konteynırın içinden oluşan paketimizi kopyalayınca rahatça kapatabiliriz:  
```
$ sudo systemctl poweroff
```
