Elimizde FreeBSD kurulum ortamını kapsayabilecek kadar depolama alanına sahip USB belleğe QEMU kullanarak günümüz bilgisayarlarından çalışabilecek ortam hazırlayabiliriz.

Benim anlatıda 15 GB civarı bir bellek kullanılmıştır. Gerçek senaryoda ise çoklu ortamlı bir bellek hazırladım, anlatı gereksiz uzamaması için konuyu basite indirgemek zorundayım.

Günümüz `x86_64` / `amd64` mimariye sahip bilgisayarlar ve dizüstülerimiz UEFI yazılımıyla gelmektedirler. Varsayılan olarak Secure Boot açık geldiği için bu cşhazlarda güvenli kabul edilen anahtarlar
tarafından imzalanmayan UEFI yazılımı çalıştırmayı reddederler. İmzalanmamış diğer işletim sistemlerini çalıştırabilmemiz için Secure Boot ayarını BIOS ayarlar kısmından kapatmamız, yahut
imzayı kontrol etsin ama zorunlu kılmasın gibi seçenek varsa onu sedçmeliyiz.

Piyasada işlemcisi 64 bit olup da 32 bit UEFI yazılımına sahip cihazlar da mevcuttur. Dolayısıyla bu seçeneği de göz önünde bulundurmalıyız.

USB belleğimizin bölümleme tablosu MBR de olsa, GPT de olsa günümüzdeki dağıtımlar sorunsuz olarak açıla bilmekteler. Gerekli olan şey o bellek üzerinde FAT16 veya FAT32 biçemli bölüm bulunması
ve bölüm'de "fallback" konumda bir uygun EFI uygulaması bulunması. 32 bit için dosya adı `bootia32.efi` olmalı, 64 bit içinse `bootx64.efi`. Harfler büyük ya da küçük olabilirler.

\EFI\BOOT\BOOTIA32.EFI  
\EFI\BOOT\BOOTX64.EFI

Bazı cihazlar Windows'un `\EFI\Microsoft\Boot\bootmgfw.efi` konumu varsayılan olarak tanınmlı geliyor. Bu konumdaki Windows uygulamasını çalıştırabiliyorlar. Keza bazı cihazlarda
NTFS bölümünden başlatma da desteklenmektedir.

Bu bilgiyi anlatmamım sebebi, çalışan USB bellek yapabilmek için gerekli asgari koşulları gözden geçirebildik, değil mi?

Dolayısıyla eliniz altında bulunan MBR / GPT bölümleme tablolarına sahip USB belleklerimizi komple uçurmamıza gerek yoktur. Misal diyelim ki cihazımız 32 GB olsun. 10 GB alan hayli büyük alandır,
biz bunu disk yöneticisinden alan açıp buraya
- Hedef işletim sisteminin anlayabileceği ve kök dizin oluşturmasına olanak sağlayan ortamın ve boş yeterli alanın bulunduğu bölüm
- FAT biçemli ESP bölümünde BOOTIA32.EFI / BOOTX64.EFI dosyaları  
eklememiz yeterli olacaktır.

İndirdiğimiz ve doğruladığımız FreeBSD ortamını ve `/dev/sdX` USB cihazımızı qemu'ya normal disk olarak monte ediyoruz. Burada dikkat edilmesi husus yanlış cihaz bağlanmamı gerekir. Günümüz
bilgisayarları NVMe SSD olarak depolamalara sahip olduğu için `/dev/nvmeX` biçiminde adlandırılıyorlar.

QEMU'yu çalıştıralım:
```
$ sudo qemu-system-x86_64 -enable-kvm \
-cpu max -m 4G -drive file=FreeBSD-15.1-RELEASE-amd64-dvd1.iso,format=raw,media=cdrom,index=0 \
-drive file=/dev/sdX,format=raw,index=1 -vga vmware -boot menu=on
```
![Loader](/FreeBSD/00_kurulum/assets/bsdinstall/00_qemu_freebsd_loader.png)  
`b` tuşuna yahut "Enter" tuşuna basarak açılışa devam ediyoruz:  
![StartBSDMenu](/FreeBSD/00_kurulum/assets/bsdinstall/01_qemu_startbsdinstall_dialog.png)  
Yön tuşlarıyla "Live System" seçeneğine "Enter" yahut `l` tuşuna basarak çalışan sistemin açılmasını sağlıyoruz.

FreeBSD'nin yönetici / yetkili kök kullanıcısı diğer UNIX benzerleri sistemlerde olduğu gibi `root`dur. login promptuna onu girip enterliyoruz:  
![Root Login](/FreeBSD/00_kurulum/assets/bsdinstall/02_qemu_root_login_live.png)  
Sistem bize parola sorarsa direk "Enter" tuşuna basıyoruz.

QEMU USB belleğimizi normal disk olarak taktığı için, FreeBSD konuğu için USB belleği `/dev/ada0` olarak adlandıracaktır. Emin olmak için:  
```
# ls /dev/ada*
```
komutunu giriyoruz.

USB belleğin bölümleme tablosunu uçurmak için:
```
# gpart destroy -F ada0
```
girebiliriz.

`gpart` kullanarak buraya MBR tablosu oluşturuyoruz. Benim USB belleğim 16 GB bellek olduğu için yarı yarıya gündelik ortak kullanım alanı ve FreeBSD böülümü ayarlamayaa karar verdim:  
```
# gpart create -s MBR /dev/ada0
# gpart add -i 1 -s 7G -t !7 ada0
# gpart add -i 2 -s 7G -t !18 ada0
# gpart add -i 3 -t efi ada0
# gpart set -a active -i 3 ada0
```
7GB bölüm NTFS bölümü, 7GB bölümü de Compac Diagnostique olarak ayarladık. Geri kalan alanın tamamını ESP bölümü olarak ayarladık.

FreeBSD kurulum ortamında NTFS biçimlendirme bulunmadığı için es geçiyoruz.

ESP bölümünü biçimlendiriyoruz. Malesef FAT32 sorun çıkardığı için FAT16 olarak işlemi yapıyoruz. FreeBSD'nin önyükleyicisi küçük ebatta olduğu için sorun çıkarmayacaktır:  
```
# newfs_msdos -L EFI /dev/ada0s2
```
`zpool` kullanarak zfs veri havuzu oluşturuyoruz. Benim gibi sık sık çeşitli havuzlar ve veri setleri düzenliyorsanız ve belleğinizde bunların imazısını silmediyseniz hata verebilir.  
```
# zpool create -m / -o cachefile=none -o altroot=/mnt -O primarycache=none FreeBSDInstallPool ada0s2
```
![Zpool Error](/FreeBSD/00_kurulum/assets/bsdinstall/03_qemu_partitioning_00.png)  
olduğu gibi size uyarı veriyorsa,  
```
# zpool create -m / -o cachefile=none -o altroot=/mnt -O primarycache=none -f FreeBSDInstallPool ada0s2
```
koşuyoruz.

Havuzumuz bağlı olduğundan emin olmak için `mount` komutundan yararlanıyoruz. Veri setimizin özelliklerine nispeten hızlı işlem sağlayan `zstd-9` sıkıştırmayı etkınleştiriyoruz:
```
# mount
# zfs set compression=zstd-9 FreeBSDInstallPool
```

Belleğin ESP bölümünü bağlayıp FreeBSD önyükleyicisini kopyalıyoruz:
```
# mkdir /tmp/efi
# mount_msdosfs /dev/ada0s3 /tmp/efi
# mkdir -p /tmp/efi/efi/boot
# cp /boot/loader.efi /tmp/efi/efi/boot/bootx64.efi
# cp /boot/loader_ia32.efi /tmp/efi/efi/boot/bootia32.efi
```

ESP bölümün bağını kaldırıp, DVD ortamını USB belleğe kopyalıyoruz:
```
# umount /tmp/efi
# for el in /bin /boot /COPYRIGHT /etc /lib* /media /net /packages /rescue /root /sbin /usr; do
        cp -rv $el /mnt$el
done
```  
![Sync Install Media](/FreeBSD/00_kurulum/assets/bsdinstall/05_qemu_sync_media__start.png)  
işlemin başarıyla bitmesini bekliyoruz ve `tmpfs` olarak veya bölüm bağlamak için kullandığımız dizinleri USB belleğimizde oluştıruyoruz:  
```
mkdir /mnt/dev /mnt/proc /mnt/tmp /mnt/mnt /mnt/var
```

USB belleğimizdeki dosya sistemi tablosu `fstab`'da geçerli bağlama noktasını belirtiyoruz:  
```
# ee /mnt/etc/fstab
```
`fstab` dosyasının içeriği bu olmalı:  
```
FreeBSDInstallPool / zfs ro 0 0
```  
`ro` olmasının sebebi çalışan ortam olduğu içindir. `rw` 'le ayarlamamız normal disk gibi kullanılmasını açacak, bu da USB belleğimizin daha önce ölmesine sebep verecektir.

Metin düzenleyicinin menüsü `ESC` tuşuna basarak beliriyor.
![Text editor](/FreeBSD/00_kurulum/assets/bsdinstall/06_qemu_etc_fstab.png)  
![Text editor](/FreeBSD/00_kurulum/assets/bsdinstall/06_qemu_etc_fstab_save_menu_00.png)  
![Text editor](/FreeBSD/00_kurulum/assets/bsdinstall/06_qemu_etc_fstab_save_menu_01.png)  

FreeBSD önyükleyici `/boot/loader.conf` ayarlarına ZFS modülünü yükletiyoruz ve Veri havuzunun adını doğru biçimde bildiriyoruz:
```
# ee /mnt/boot/loader.conf
```
`loader.conf`:
```
zfs_load="YES"
vfs.mountfrom="zfs:FreeBSDInstallPool"
```

USB belleğimizin bölümünü ayırıp sanal makinemizi kapatıyoruz:
```
# zfs set mountpoint=legacy FreeBSDInstallPool
# zpool export -f FreeBSDInstallPool
# poweroff
```
![Text editor](/FreeBSD/00_kurulum/assets/bsdinstall/07_loader_conf.png)  
![Text editor](/FreeBSD/00_kurulum/assets/bsdinstall/08_qemu_poweroff.png)  

QEMU kapanınca USB belleğin NTFS bölümünü `gparted` yahut
```
sudo mkntfs -fQCIL alicavus /dev/sda1
```
koşarak biçimlendirebiliriz.

![Text editor](/FreeBSD/00_kurulum/assets/bsdinstall/09_format_ntfs_partition.png)

Eğer şahsi anahtarlarımızla EFI dosyalarını imzalayabiliyorsak, USB belleğimizin FreeBSD `bootia32.efi` ve `bootx64.efi` önyükleyicilerini imzalarsak makinemizde Secure Boot'u kapatmadan
FreeBSD'den sistemimizi başlatabiliriz.  
Not: İmza sadece önyükleyicilere güvendiğinizi bildirir. Secure Boot gereksinimleri FreeBSD'ye entegre etme halen bir süreçtir. <sup>[\*](https://wiki.freebsd.org/SecureBoot), [\*\*](https://freebsdfoundation.org/freebsd-uefi-secure-boot/)</sup>
