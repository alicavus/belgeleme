_Not: Makale 08 Mart 2020 y. 6:57 TRT saatinde [Archman OS](https://archman.org/topluluk/belgeler/nasil-kurulu-archman-ve-diger-arch-linux-tabanli-dagitimlari-calisan-iso-haline-getirme/) bünyesinde yayımlanmıştır.
Söz konusu tarihten beri geçen süre içinde güncelliğini yitirmiş olabilir._

Bu belgede bilgisayarımıza kurulu Arch Linux tabanlı dağıtımın çalışan ISO ortamı haline getirilmesi anlatılacaktır. Söz konusu işlem tamamen "amatör" diyebileceğimiz, yani resmi dağıtım yapımı sürecini
kapsamayan kendi kullanımımıza yönelik çalışmayı kapsamaktadır. Bu belge hiç bir şekilde resmi veya gayrı resmi mesuliyet isnadı kabul etmez, herhangi garantisi tahakkukunda bulunmaz.
Kendin yap (DIY, "Do It Yourself") mantığına uygun olarak çıkabilecek sorunlarla söz konusu kullanıcının az buçuk çözüm süreçlerine hakim olmalıdırlar.

Yöntem tarafımca denenmiştir.

### 0. Gerekli ortam:
* En son Arch Linux kurulum [ISO ortamı](https://archlinux.org/download/)
* Sabit diskimizde gerekli alan
* `rsync`
* `squashfs-tools`
* `git`
* `libisoburn`
* `qemu`, `virtualbox`, `vmware` gibi sanallaştırma yazılımı
* bolca sabır, zaman + kahve

### 1. İlkönce çalışma dizinlerimizi oluşturalım:  
```
$ mkdir -p ~/workdir/{iso,myinstall}
```

### 2. Güncel Arch Linux ISO ortamını açalım:  
```
$ bsdtar -xf archlinux-2022.10.01-x86_64.iso -C ~/workdir/iso
$ find ~/workdir/iso -type d -exec chmod 0755 {} \;
$ find ~/workdir/iso -type f -exec chmod 0644 {} \;
```

### 3. Kurulu sistemimizi çalışma dizinine kopyalayalım:  
```
$ sudo rsync -av /* ~/workdir/myinstall --exclude=dev/*  --exclude=home/$USER/* --exclude=media/* --exclude=mnt/* --exclude=proc/* --exclude=sys/* --exclude=tmp/* --exclude=var/cache/pacman/pkg/* --exclude=var/lib/pacman/sync* --exclude=var/tmp/*
```

### 4. Çalışma dizinindeki sisteme `arch-install-scripts`, `mkinitcpio-archiso` paketlerini kuralım:  
```
$ sudo pacman -S arch-install-scripts mkinitcpio-archiso -r ~/workdir/myinstall/
```

### 5. Kullanıcı dizinindeki ayar dosyalarını (`.config`, `.cinnamon`, `.mozilla`, `.bashrc`, `.pam_environment`, `.bash_profile`, `.xinitrc` vs...) `~/workdir/myinstall/home/$USER` dizinine kopyalayım.  
```
$ cp -av ~/.{config,cinnamon,mozilla,{ba,z}shrc,pam_environment,{bash_,z}profile,xinitrc} ~/workdir/myinstall/home/$USER
```

### 6. `~/workdir/myinstall/etc/mkinitcpio.conf` dosyasına `archiso` ve `archiso_loop_mnt` kancalarını ekleyelim:  
Not: Eğer söz konusu ayar dosyasında `udev` yerine `systemd` eklediysek eski haline çevirmeliyiz (`systemd` silip `udev` ekliyoruz), `archiso` malesef `systemd` ayarı ile düzgün çalışmıyor.

### 7. ISO ortamımız için initramfs dosyasını oluşturalım:  
```
$ sudo ~/workdir/myinstall/usr/bin/arch-chroot ~/workdir/myinstall mkinitcpio -P
$ sudo cp ~/workdir/myinstall/boot/vmlinuz-linux* ~/workdir/myinstall/boot/initramfs-linux*.img ~/workdir/iso/arch/boot/x86_64/
$ sudo rm -f ~/workdir/iso/boot/arch/x86_64/*fallback*
```
Not: Kullandığımız çekirdeğe(-klere) göre `~/workdir/myinstall/boot` dizininde oluşan dosyaların isimleri farklılık gösterebilir (örn. `vmlinuz-linux-lts`, `vmlinuz-linux-zen` gibi),
bunları `~/workdir/iso/arch/boot/x86_64/` dizinine kopyalayınca önyükleyici ayarlarına düzgün biçimde eklemeliyiz.

### 8. ISO ortamımız için güncel paket listesini ekleyelim:  
```
$ pacman -Q -r ~/workdir/myinstall > ~/workdir/iso/arch/pkglist.x86_64.txt
```

### 9. Gereksiz dosya ve dizinleri temizleyelim:  
`~/workdir/myinstall` dizininde:

* `/var/cache`, `/var/log`, `/var/spool`, `/var/db/sudo/lectured` dizinlerinde gereksiz dosyaları siler ya da içeriğini sıfırlayabiliriz.
* `/home` dizininde diğer yerel kullanıcılara ait dizinleri silmeliyiz:
<pre>$ sudo ~/workdir/myinstall/usr/bin/arch-chroot ~/workdir/myinstall userdel -fr <i>kullanici</i></pre>
* `/etc` dizininde kişisel verileri silmeliyiz (örn. Ağ şifreleri), gereksiz `systemd` servislerini devre dışı bırakmalıyız.
* `/etc/fstab` dosyasını sıfırlamalıyız, varsa `/etc/crypttab` silmeliyiz:
```
$ sudo truncate -s 0 ~/workdir/myinstall/etc/fstab
```
* `/boot` dizininin içeriğini silmeliyiz:
```
$ sudo rm -rfv ~/workdir/myinstall/boot/*
```
* `~/workdir/iso` dizininde eski önyükleyici yapılandırmalarını kaldırmalıyız:
```
$ rm -rfv ~/workdir/iso/{syslinux,EFI,arch/grubenv}
```

### 10. Sistem airootfs imajı ve sağlama dosyası oluşturma:  
```
$ rm ~/workdir/iso/arch/x86_64/airootfs.*
$ sudo mksquashfs ~/workdir/myinstall ~/workdir/iso/arch/x86_64/airootfs.sfs -b 1M -comp xz -Xbcj x86 -Xdict-size 1M
$ cd ~/workdir/iso/arch/x86_64/
$ sha512sum airootfs.sfs > airootfs.sha512
```

### 11. `~/workdir/iso` dizini erişimleri ayarlama:  
```
$ sudo find ~/workdir/iso -exec chown $USER:$USER {} \;
$ sudo find ~/workdir/iso -type d -exec chmod 0755 {} \;
$ sudo find ~/workdir/iso -type f -exec chmod 0644 {} \;
```

### 12. Önyükleyici kurma ve ayarlama:  
```
$ git clone https://github.com/limine-bootloader/limine.git --branch=v4.x-branch-binary --depth=1 ~/workdir/limine-bin
cp ~/workdir/limine-bin/limine*{sys,bin} ~/workdir/iso
cat << EOF > ~/workdir/iso/limine.cfg
DEFAULT_ENTRY=1
TIMEOUT=30
VERBOSE=yes
INTERFACE_BRANDING=My Custom Arch ISO
INTERFACE_BRANDING_COLOUR=2
TERM_BACKGROUND=88000000
:Custom Arch ISO
    COMMENT=Boot custom Archiso
    PROTOCOL=linux
    KERNEL_PATH=boot:///arch/boot/x86_64/vmlinuz-linux
    MODULE_PATH=boot:///arch/boot/intel-ucode.img
    MODULE_PATH=boot:///arch/boot/x86_64/initramfs-linux.img
    CMDLINE=archisobasedir=arch archisolabel=CUSTOMISO
:UEFI shell (UEFI only)
    COMMENT=Run UEFI shell
    PROTOCOL=chainload
    IMAGE_PATH=boot:///shellx64.efi
:Boot next
    COMMENT=Boot next available boot option
    PROTOCOL=chainload_next
EOF
```
Farklı çekirdekler için yeni girdiler ekleyebilir veya mevcut olanı düzenleyebilirsiniz.  
Not: limine önyükleyicisi 4 GiB ve üzeri ISO dosyalarını çalıştıramaz.

### 13. ISO kalıbı oluşturma:  
```
$ xorriso -as mkisofs -eltorito-boot limine-cd.bin -no-emul-boot -boot-load-size 4 -boot-info-table --efi-boot limine-cd-efi.bin -iso-level 3 -full-iso9660-filenames -joliet -joliet-long -rational-rock -volid CUSTOMISO -output ~/workdir/mycustom.iso ~/workdir/iso
```  
Çalışma dizininde `mycustom.iso` oluştulunca, sanallaştırma yazılımıyla test edebiliriz. İşlem başarıyla bittiğinde ISO dosyamızı taşıyıp çalışma dizini `~/workdir` komple kaldırabiliriz:  
```
$ sudo rm -rf ~/workdir
```
