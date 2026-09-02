_Not: Makale 08 Mart 2020 y. 6:57 TRT saatinde [Archman OS](https://archman.org/topluluk/kurulum/arch-linux-2020-02-01-btrfsgnomeuefi-kurulum-rehberi/) bünyesinde yayımlanmıştır.
Söz konusu tarihten beri geçen süre içinde güncelliğini yitirmiş olabilir._

> **UYARI!**<br/>
> Bu anlatımı takip etmek isteyenler basit derecede Uçbirim kullanımına hakim olmalıdırlar. Verilen her komut kopyala-yapıştır mantığıyla değil **anlayarak** kullanılmalı.
> "Komutları kullandım harddiskim sıfırlandı" gibi hiçbir zarar konusunda mesuliyet kabul etmiyorum.

# I. GİRİŞ

**[BTRFS](https://btrfs.wiki.kernel.org) CoW** (Copy on Write) tekniğini kullanan, anlık kayıt (snapshot), depolama altbirimi (subvolume), sıkıştırma, birden fazla diski tek depolama
alanında birleştirme gibi özellikleri olan dosya sistemidir. GPL ile lisanslanmıştır.

Linux çekirdeği `5.0` sürümünden itibaren BTRFS takas alanı dosyasını desteklemektedir. Arch Linux **[linux-lts](https://www.archlinux.org/packages/core/x86_64/linux-lts/)** paketinin sürümü `5.4` olduğu için
**[linux](https://www.archlinux.org/packages/core/x86_64/linux/)** ve **[linux-lts](https://www.archlinux.org/packages/core/x86_64/linux-lts/)** ile takas alanı oluşturulabilir.
Fedora 31 ve Ubuntu 19.10 dağıtımları linux çekirdeğinin sürümü 5.3<sup>Bkz:[1](https://fedoramagazine.org/announcing-fedora-31/) ve [2](https://wiki.ubuntu.com/EoanErmine/ReleaseNotes)</sup>.
Manjaro 19'un ikinci sürüm adayı [yayımlandı](https://osdn.net/projects/manjaro/storage/xfce/), [18.1.5](https://osdn.net/projects/manjaro/storage/xfce/18.1.5) sürümündeki çekirdek sürümü ise `5.4.6`<sup>
[3](https://osdn.net/projects/manjaro/storage/xfce/18.1.5/manjaro-xfce-18.1.5-minimal-191229-linux54-pkgs.txt)</sup>

BTRFS sayesinde her dağıtım için ayrı ayrı disk bölümlemesi yapma gerekliliği kalkmıştır. Arch Linux, Ubuntu, Manjaro, Fedora dağıtımlarını ve bu dağıtımların farklı masaüstü ortamlarını aynı bölüme yükleyebiliriz.
Anlık kayıtlar sayesinde kolayca sistemlerimizi geri yükleyebiliriz, dilersek takas alanı için dosya oluşturabiliriz.

Kurulum ortamı hazırlama,sistem gereksinimleri gibi konulara girmeyeceğim, dilerseniz Demiray Muhterem hocamızın [anlatımına](https://archman.org/forum/index.php?topic=3637.0) göz atabilirsiniz.

Peki başlıyoruz.

Arch Kurulum medyasından sistemi başlatalım:  

![Screenshot-from-2020-02-23-07-13-16](https://github.com/user-attachments/assets/4995aa3f-3f2f-4e98-aa68-8ff802e5c38a)

# II. KURULUMA HAZIRLIK

## 1. Dilediğimiz klavye düzenini ve Türkçeyi düzgün biçimde gösteren uçbirim fontunu etkinleştirelim.  
```
loadkeys trq
setfont /usr/share/kbd/consolefonts/iso09.16.gz
echo "Üzüm hazmı kolaylaştırır"
ping archlinux.org
```
![Screenshot-from-2020-02-23-07-45-15](https://github.com/user-attachments/assets/00e8066c-a2fe-458b-a6aa-e4306cc711da)  
Görüldüğü üzere Türkçe karakterler sorunsuz gözüküyor ve internet bağlantımız etkin.

## 2. Disk yapılandırmasını düzenleyelim:

`fdisk -l`, `lsblk`, `ls -la /dev/disk/by-path/` gibi araçlarla hedef diskimizi belirleyelim. Daha sonra biçimlendirme araçlarının birini kullanarak hedef diskimizde boş alana
veya yeni tablolama biçimi kullanarak UEFI sistemler için en az **ESP** (EFI Sistem bölümü) ve **BTRFS** bölümler olabilecek şekilde yapılandıralım.  
```
cfdisk /dev/sda
```
![Screenshot-from-2020-02-23-07-47-40](https://github.com/user-attachments/assets/ba76d362-a44e-4114-842d-ab9f2af71f96)  
![Screenshot-from-2020-02-23-07-58-11](https://github.com/user-attachments/assets/621dbb66-84b1-4424-8c68-034b81999b0c)  

### 2.1 Daha önceden hazır alan hazırlamadıysak, eğer bölümler biçimlendirilmemiş ise onları biçimlendirelim.
```
fdisk -l
mkfs.vfat -F32 /dev/sda1
mkfs.btrfs /dev/sda2
```
![Screenshot-from-2020-02-23-08-02-04](https://github.com/user-attachments/assets/45cafe2b-2709-4801-a6d1-4c42f191a7d7)

## 3. Gerekli bağlama noktaları ve btrfs işlem hazırlıklarını yapalım:

Kurulum yaparken **arch-install** ve **btrfs-partition** adında iki dizin oluşturacağız. Bu iki dizini `/mnt` veya `/tmp` altına oluşturup sistemimizi bağlayabiliriz.  
Bağlama yaparken BTRFS nimetlerinden yararlanmayı unutmayacağız, sıkıştırma biçimini aktif edip `zstd` kullanacağız.
[Zstandard](https://facebook.github.io/zstd/) hızlı açılma sağladığı için okuma hızımız yüksek olacaktır.

### 3.1 Bağlama noktalarını oluşturuyoruz:
```
mkdir -p /tmp/{arch-install,btrfs-partition}
```

### 3.2 BTRFS bölümünü bağlıyoruz
```
mount -o compress=zstd /dev/sda2 /tmp/btrfs-partition
```

### 3.3 Sistemler için gerekli alt birimleri oluşturuyoruz.

Giriş bölümünde işaret ettiğimiz gibi boş BTRFS bölümüne istediğimiz kadar alt birimler oluşturabiliriz.
Arch Linux için xfce, gnome, cinnamon, kde ayrı ayrı masaüstü ortamı ekleyebiliriz. Keza Ubuntu, Fedora, Manjaro gibi dağıtımlar için altbirimler ilave edebiliriz.

"Ubuntu style subvolumes" (kökbirim için **@**, ev dizini için **@home**) kullanabiliriz, yahut istediğimiz biçimde isim koyabiliriz.

Biz BTRFS bölümüne her dağıtım için ayrı dizin, her dağıtım dizininde masaüstü ortamları için dizin ve masaüstü ortam dizininde ise alt birimleri oluşturacağız.
Bunu böyle yapmak yerine "dağıtım-masaüstü ortamı-alt birim" adında uzun isimli altbirim yapabiliriz.  
```
for distro in arch fedora ubuntu; do
    for desktop in xfce gnome cinnamon kde; do
        mkdir -p /tmp/btrfs-partition/$distro/$desktop
        for subvol in "@" "@home"; do
            btrfs subvolume create /tmp/btrfs-partition/$distro/$desktop/$subvol
        done
    done
done
```

### 3.4 Yükleme için gerekli altdizinleri oluşturuyoruz ve altbirimleri yükleme noktasına bağlıyoruz:

*GNOME anlatımı yaptığımız için GNOME altbirimlerini yükleme dizinine bağlayalım:*
```
mount -o compress=zstd,subvol=arch/gnome/@ /dev/sda2 /tmp/arch-install
mkdir -p /tmp/arch-install/{boot/efi,home,swap}
mount /dev/sda1 /tmp/arch-install/boot/efi
mount -o compress=zstd,subvol=arch/gnome/@home /dev/sda2 /tmp/arch-install/home
```
![Screenshot-from-2020-02-24-07-06-40](https://github.com/user-attachments/assets/accba2da-8dea-44ac-ad64-1cf840cc98de)

### 3.5 Takas alanı ayarlaması

BTRFS üzerinde takas alanı dosyaları desteklenmektedir. Ancak takas dosyanın bulunduğu altbirimin anlık kaydı bulunmamalı, dosyanın sıkıştırılması ve **CoW** özelliği olmamalı
ayrıca BTRFS bölümü tek depolama aygıtında olmalı.  
Sistemlerimizi yedekleme amaçlı sürekli anlık kayıtlarını alacağımız için swap dosyamızı ayrı altbirimde tutacağız.
Kökbirimin bağlı olduğu altbirimin snapshotunu sürekli alacağımızdan **arch/gnome/@** altında takas alanımızı kaydetmiyeceğiz.
```
btrfs subvolume create /tmp/btrfs-partition/swap
mount -o subvol=swap /dev/sda2 /tmp/arch-install/swap
```  
Görüldüğü üzere altbirim parametresinin yanına sıkıştırma algoritması eklemedik, çünkü bu altbirimde takas dosyaları muhafaza edeceğiz.

Takas dosyamızı oluşturuyoruz:
```truncate -s 0 /tmp/arch-install/swap/arch-swap
chattr +C /tmp/arch-install/swap/arch-swap
btrfs property set /tmp/arch-install/swap/arch-swap compression none
dd if=/dev/zero of=/tmp/arch-install/swap/arch-swap bs=1M count=4096 status=progress
```
`chattr +C` ile özelliğini kapattık.  
<tt>btrfs property set _object_ compression none</tt> ile dosyanın sıkıştırılmasını kaldırdık. *Dikkat ederseniz daha önce btrfs bölümünü sıkıştırmalı bağlamıştık - **3.2**'de*

Takas alanı oluşturup bağlıyoruz:<br />
```
chmod 600 /tmp/arch-install/swap/arch-swap
mkswap /tmp/arch-install/swap/arch-swap
swapon /tmp/arch-install/swap/arch-swap
```
Takas dosyasına sadece root erişebilmeli, bundan dolayı okuma ve yazma hakkı dosyanın sahibine ait olmalı, çalıştırılmasına ise gerek yok.
![Screenshot-from-2020-02-23-14-48-55](https://github.com/user-attachments/assets/74eeb460-c55b-4b20-9964-354bfd1b85a2)

Artık Kuruluma geçebiliriz.

# III. KURULUM VE YAPILANDIRMA

## 1.Temel sistemi kurma:

[`base`](https://archlinux.org/packages/core/any/base/) ve [`base-devel`](https://archlinux.org/packages/core/any/base-devel/) paketleri yüklüyoruz. Paket inşa edebilmek için 
linux başlıkları gerekebiliyor, bundan dolayı [linux-headers](https://www.archlinux.org/packages/core/x86_64/linux-headers/) yüklüyoruz.
Çekirdek ve sürücüler için [`linux`](https://www.archlinux.org/packages/core/x86_64/linux/) ve [`linux-firmware`](https://www.archlinux.org/packages/core/x86_64/linux-firmware/) yüklüyoruz,
metin düzenleyici([nano](https://www.archlinux.org/packages/core/x86_64/nano/)) ve `man` ve `info` için belgeleme araçlarını
([`man-db`](https://www.archlinux.org/packages/core/x86_64/man-db/), [man-pages](https://www.archlinux.org/packages/core/x86_64/man-pages/) ve
[texinfo](https://www.archlinux.org/packages/core/x86_64/texinfo/)) eklemeyi unutmuyoruz.
```
pacstrap -i /tmp/arch-install base{,-devel} linux{,-headers,-firmware} nano man-{db,pages} texinfo
```
![Screenshot-from-2020-02-23-09-40-42](https://github.com/user-attachments/assets/a42b70e7-b623-45fd-b72e-422245c53de3)

## 2. `systemd` yapılandırması

Hedef sistemimizin dili, klavye düzeni, zamandilimi, hostname tek vuruşta yapılandırıyoruz. `pacstrap` betiğinde `pacman` çalışırken `systemd` kurulumunda makine kimliği oluşumu tetiklendiği için `--setup-machine-id` parametresini es geçebiliriz.  
```
systemd-firstboot --root=/tmp/arch-install --locale=tr_TR.UTF-8 --keymap=trq --timezone=Europe/Istanbul --hostname=alicavus --setup-machine-id
```
![Screenshot-from-2020-02-23-10-15-35](https://github.com/user-attachments/assets/3c0be977-fd7d-41a1-b4c8-439d4a805f0a)

## 3. Dağıtımımızın dosya sistemi tablosu ayarlarını kaydediyoruz:
```
genfstab -U /tmp/arch-install &gt; /tmp/arch-install/etc/fstab
```

## 4. Hedef sistemi yapılandırma:

Hedef sistemimizi chroot ortamıyla düzenleyeceğiz  
```
arch-chroot /tmp/arch-install /bin/bash
```
![Screenshot-from-2020-02-23-09-53-47](https://github.com/user-attachments/assets/d33b1653-e31c-4eed-ae6d-888087bb8d00)

#### \* Kolaylık için tab tuşuyla otomatik tamamlama sağlayabilmek için [`bash-completion`](https://www.archlinux.org/packages/extra/x86_64/bash-completion/) paketini kuruyoruz:  
```
pacman -S --asdeps bash-completion
```
![Screenshot-from-2020-02-23-11-34-33](https://github.com/user-attachments/assets/97e17cc2-6312-43e9-ad30-d9f0a324d0f9)

### 4.1 Yerelleştirme ayarlaması
```
export LANG=tr_TR.UTF-8
nano /etc/locale.gen
```
![Screenshot-from-2020-02-23-10-03-10](https://github.com/user-attachments/assets/4bb3a5ee-04ab-4641-8ba1-0de7e2a31dea)

istediğimiz locale tanımlamalarını seçiyoruz. *Coğrafi konuma ve tercihe göre değişir. Almanya'daki kardeşlerimiz mesela Türkçeyle beraber almanca yerelleştirmeye sahip
olmak isteyebilirler.*  
![Screenshot-from-2020-02-23-10-05-07](https://github.com/user-attachments/assets/ee55710d-5c55-4651-bb21-fd4bbc6553b6)  
Gerekli işaretlemelerden sonra
```
locale-gen
```
çalıştırılır.

### 4.2 Ağ yapılandırması
```
nano /etc/hosts
```
`hosts` dosyasına kendi makine ismini ekliyoruz:  
```
127.0.0.1	localhost
::1			localhost
127.0.1.1	alicavus.localdomain	alicavus
```
![Screenshot-from-2020-02-23-10-10-26](https://github.com/user-attachments/assets/11221e68-ac96-4892-bbdc-9de866f99d27)

### 4.3 Önyükleyici ve işlemci microcode yükleme:
Önyükleme için EFISTUB, systemd-boot, GRUB, syslinux, rEFInd kullanabiliriz. Tercihimiz hangisinden yanaysa onu kuruyoruz.  
İşlemci hatalarını düzeltmeye yönelik microcode paketini kururuyoruz. AMD işlemcileri için `amd-ucode`, INTEL için `intel-ucode`:
```
pacman -S grub intel-ucode
pacman -S --asdeps efibootmgr dosfstools libisoburn os-prober
```
Eğer tek işletim sistemi kuracaksak `os-prober` paketine ihtiyaç yoktur.  
![Screenshot-from-2020-02-23-10-19-24](https://github.com/user-attachments/assets/94058952-57ed-488b-a508-54c00f82d2a8)  

*Ekleme*: Makale yayımlandıktan sonra belirli bir zaman sonra `os-prober` güvenlik sebebiyle devre dışı bırakılmıştır. Etkinleştirme için:  
```
sed -e "s|^#GRUB_DISABLE_OS_PROBER=*|GRUB_DISABLE_OS_PROBER=false|" -i /etc/default/grub
```
koşulması gerekir.  
```
grub-install --efi-directory=/boot/efi --bootloader-id="Arch Linux Bootloader" /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```
![Screenshot-from-2020-02-24-07-25-24](https://github.com/user-attachments/assets/69b131eb-097e-4a5f-9db7-96ebf874aed8)

### 4.4 Kullanıcı ekleme
Kullanıcı sistem ve `systemd` gruplarına uygun yapılandırılması gerekir.  
```
useradd -mUG adm,log,rfkill,sys,systemd-journal,lp,wheel -s /bin/bash -c "Ali Çavuş" alicavus
echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/20-wheel
passwd alicavus
```  
Kullanıcımıza şifre tanımlaması yapmazsak oturum açamayacağız. Otomatik giriş ayarlasak dahi yönetimle ilgili işlemleri yapamayacağızdan dolayı mutlaka şifre tanımlamamız gerekir.  

<img class="go2wpf-bbcode" src="https://i.ibb.co/h8M1DHz/Screenshot-from-2020-02-23-10-40-17.png" /><br /><br />Kullanıcımızı süper haklarla donatınca dilersek root girişini engelleyebiliriz, dilersek passwd ile şifre belirleyebiliriz.<br />
```
passwd -l root
```

### 4.5 GNOME masaüstü ortamını yükleme
```
pacman -S gnome{,-extra}
```
Dilediğimiz paketleri elle belirleyebiliriz.  
![Screenshot-from-2020-02-23-10-43-37](https://github.com/user-attachments/assets/886ab3af-3839-4f3f-9726-67841a38405c)  
![Screenshot-from-2020-02-23-10-43-47](https://github.com/user-attachments/assets/862d9932-da1c-4045-a1b3-0e18d015ad3d)  
![Screenshot-from-2020-02-23-11-30-16](https://github.com/user-attachments/assets/136a7cc9-3524-4371-b894-00e45b187447)  
*Tüm bileşenleri yüklemenize gerek yok. Temel bileşenleri yükleyince daha az yer kaplayan sisteminiz olur, bazı düzenlemeleri elle yapmanız sizi korkutmasın.*  
`gnome` grubunu kurunca ekran yöneticisi ve ağ yöneticisini ayrıyeten yüklememize gerek yok. Elle seçim yaptıysak `gdm` ve `network-manager`'i seçtiğimizden emin olmalıyız.  
Şimdi `gdm` ve `NetworkManager` hizmetlerini (servislerini) etkinleştirelim:  
```
systemctl enable gdm.service
systemctl enable NetworkManager.service
```
![Screenshot-from-2020-02-23-11-31-32](https://github.com/user-attachments/assets/7845f96c-3ff1-4a35-9d4d-ebead17d3f6f)


#### \* Etkili Linux destekli Güç yönetimi sağlamak için `tlp` paketini kurabiliriz.
```
pacman -S tlp
systemctl enable tlp.service
```
![Screenshot-from-2020-02-23-11-42-30](https://github.com/user-attachments/assets/f3fb54d4-125e-4104-bfa4-c9861c855c94)

### 4.6 Kurulumu tamamlama
`chroot` ortamından çıkıp bağlı takas ve depolama birimlerini ayırıyoruz, sistemi yeniden başlatıyoruz:  
```
exit
swapoff /tmp/arch-install/swap/arch-swap
umount /tmp/{arch-install/{boot/efi,home,swap,},btrfs-partition}
reboot
```
![Screenshot-from-2020-02-24-07-37-31](https://github.com/user-attachments/assets/fba51439-36dd-458e-8466-49651b07cd68)  
![Screenshot-from-2020-02-23-11-49-57](https://github.com/user-attachments/assets/1374b43d-6553-41e8-8a78-c1b7c508bb5e)  
![Screenshot-from-2020-02-23-11-51-20](https://github.com/user-attachments/assets/d7d3f175-2edd-4025-a3c6-0ee26c2ec289)  
![Screenshot-from-2020-02-23-11-57-26](https://github.com/user-attachments/assets/40a93eda-4f66-4cf7-a9b5-5dbc9d858fba)  
![Screenshot-from-2020-02-23-11-58-10](https://github.com/user-attachments/assets/a1b854f1-c3f3-4b8d-bddf-7e905ea8bdf3)  
![Screenshot-from-2020-02-23-12-00-17](https://github.com/user-attachments/assets/863c63b5-3150-4112-ac1a-50f9f2de1b7d)

# IV. KURULUM SONRASI
Dilersek otomatik betiklerle, yahut elle düzenli anlık kayıt (snapshot) alabiliriz. Snapshotlardan sistem açma gibi özellikler taşıyan `grub-btrfs` önyükleyiciyi kurabiliriz.  
Uzun uzun yedeklemeye gerek yok, tek yapmanız gereken:  
```
btrfs subvolume snapshot source [dest/]name
```

# V. NOTLAR
ESP bağlama noktasını `/boot/efi` olarak belirlememiz daha isabetli olacaktır. Kurulumlardan bir tanesini `/boot` altına bağlayınca diğerlerle çakışmaz, fakat snapshot
işleminde kernel ve initramfs kapsam dışında kalıyor.

GNOME kurulumu bitince daha önce `xfce`, `cinnamon`, `kde` hazırlamamıza gerek yoktur, anlık kayıt ekleyip daha hızlı olacaktır.
Hatta masaüstü ortamsız saf arch base kurulumu yapıp ondan dilediğimizi filizlendirebiliriz.

Fedora veya Ubuntu kuruyorsak, daha önceden hazırladığımız altbirimler işimize yarayacaktır. Dilediğimiz masaüstü ortamlı isoyu boot edip btrfs altbirimleri bağlayınca
```
unsquashfs -f -d /hedef sqfs
```
kullanarak hızlı kurulum elde edebiliriz.  
Anaconda ve ubiquity karmaşık şekildeki altbirimleri nasıl algılar birfikrim yok.
