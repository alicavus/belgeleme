Microsoft Windows, Ubuntu, Debian gibi işletim sistemlerinden FreeBSD kurulum ortamı hazırlanabilmektedir.

Yerel yansilardan DVD ISO ve disk imaj dosyaları temin edilebilir ve daha sonra o dosyalar USB belleğe yazılabilir.

Ben hazırlığımı Debian üzerinden yaptım.

Dosya indirilip sağlanması yapıldığında direk olarak USB belleğe yazabiliriz:
```
$ sudo dd if=FreeBSD-15.1-RELEASE-amd64-memstick.img of=/dev/sda status=progress
```

Fakat ben USB belleğimin bölümleme tablosunu sürekli uçurmak yerine dört bölümden oluşan MBR tablosu oluşturdum:
* `/dev/sda1`:
NTFS biçimli bölüm. Burada her işletim sisteminden ulaşılabilen dosyalar barınır. HBCD PE ISO kalıbı dosyaları burada açılmıştır. 
* `/dev/sda2`:
EXT4 biçimli bölüm. Burada ISO dosyasından direk olarak çalışabilen Linux dağıtımlarının kalıpları vardır.
* `/dev/sda3`:
ZFS biçimli bölüm. Burada FreeBSD'nin kurulum ortamı barınmaktadır.
* `dev/sda4`:
ESP bölümüdür. GNU GRUB önyükleyici kuruludur. Yapılandırma dosyasını HBCD, Linux dağıtımları ve FreeBSD ortamını açabilecek şekilde ayarladım.

OpenZFS yüklü olan dağıtımlarda dört bölüm de ulaşılabilir. Yüklü olmasa bile sanallaştırma desteği olan her Linux dağıtımından FreeBSD kurulum
ortamını ve USB belleğinizi sanal makineye takarak kurulumu gerçekleştirebilirsiniz.

```
$ sudo qemu-system-x86_64 -enable-kvm \
-cpu max -m 4G -drive file=FreeBSD-15.1-RELEASE-amd64-dvd1.iso,format=raw,media=cdrom,index=0 \
-drive file=/dev/sdX,format=raw,index=1 -vga vmware -boot menu=on
```

Ancak bu yöntem teknik bilgi ve beceri gerektirir.

### Windows

UNIX dünyasından aşina olduğumuz `dd` gibi ham very olarak yazabilen yazılım kullanmamız gerekiyor. Win32 Disk Imager, Rufus bu amaçlar için uygundur.
