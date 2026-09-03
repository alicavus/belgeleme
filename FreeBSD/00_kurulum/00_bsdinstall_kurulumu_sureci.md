### Sistemi diskimize yüklemeden önce 
 Dizüstü cihazlarım Debian 13, Archman OS, Arch Linux gibi GNU/Linux dağıtımlara ev sahipliği yapmaktadır. Xiaomi Redmi Book Pro 14 Microsoft 11 Pro ile gelmektedir. Depolama birimi olarak iki cihazda da NVMe M.2 SSD vardır
 ve ikisi de GPT tablolama şemasını kullanır.

 Cihazlar `x86_64` UEFI kipinde açılış yapmaktalar. Dolayısıyla tüm diski ZFS veri havuzu olarak kullanamam, çünkü `FAT` biçemli bölüme ihtiyaç duyulmaktadır.

 FreeBSD kurulumları için 50 GB alan aslında hayli uygundur. Dilersek bunu 100 GB olarak da seçebiliriz.
```
/dev/nda0
├── ESP:   512M FAT32 biçemli bölüm
├── MSR:   16M Windows için saklanmış özel alan 
├── NTFS:  Windows yüklü bölüm
├── BTRFS: Çeşitli GNU/Linux içeren bölüm
├── NTFS:  Ortak depo alanı
└── ZFS:   FreeBSD için ayırdığımız alan
```
# Resmi kurulum aracı nasıl çalışır?

Yükleme ortamını çalıştırdığımızda FreeBSD'nin [el kitapçığı](https://docs.freebsd.org/en/books/handbook/bsdinstall/) önergeleri takip ederek kurulum yapmaya çalışabiliriz.

Aslında çalıştırdığınız kurulum ortamı bir kurulu FreeBSD'dir. Bilgisayarı açıp başlatma aygıtından FreeBSD'nin önyükleyicisini çalıştırdığınız anda itibaren
siz kurulu bir sistemi kullanmaktasınız. Bizi karşılayan karşılama ekranı [basit bir kabuk betiğidir](https://cgit.freebsd.org/src/tree/release/rc.local).  
Dolayısıyla `/etc/rc.local` dosyasını sıfırlamamız direk sistemi açmaya yönlendirecektir, bunun `l` tuşuna basarak "Live System" seçeneğini seçmekten bir farkı yoktur.  
![Açılış seçenekleri](/FreeBSD/00_kurulum/assets/bsdinstall/01_qemu_startbsdinstall_dialog.png)

"Install" kur seçeneği kullanıcıları kolaylaştırmak için hazırlanmış bir araçtır.

Normalde açılışta karşımıza çıkan `bsdinstall` aracı elle yapmamız gereken müdahaleleri otomatik yapmaya çalışıyor.

Kurulum şöyle ilerliyor:
- Kurulum ortamı salt okunur olarak tasarlandığı için [açılış esnasında](https://cgit.freebsd.org/src/tree/libexec/rc/rc.conf) `/tmp` `/var` dizinleri için 20 MB ve 32 MB, toplamda 52 MB kadar RAM bellek alanı "md" yani "bellek temelli disk" oluşturuluyor.
- `bsdinstall` `/tmp` dizinine `bsdinstall_etc`, `bsdinstall_boot` oluşturuyor.
- Otomatik kurulum başlatılıyor:
  * Kullanıcıdan klavye düzeni seçmesi isteniliyor. Başarılı olunca tercihler `/tmp/bsdinstall_etc/rc.conf.keymap` dosyasına yazılıyor.
  * Makina adı ayarlanması yapılıyor. Başarılı olunca tercihler `/tmp/bsdinstall_etc/rc.conf.hostname` dosyasına yazılıyor.
  * Dağıtım seti temelli yahut paket yöneticisi temelli kurulum yapmamız soruluyor. Tercihimize göre kurulu medyasında mevcut yazılımları mı yoksa
    sunucudan mı indirmemiz soruluyor.
    + Kapsamdışı kurulumlarda, dağıtım setleri / yerli paket deposunda temel bileşenlerin olup olmadığı kontrol edilir
    + Çevrimiçi kurulumlarda Ağ yapılandırmasına geçiliyor.
      İşte tam bu esnada iki dizüstü bilgisayarımda kablosuz ağlara bağlanma esnasında donuverdi. `SIGINT` yakalama olduğu için çıkış sağlanamıyor. Elle
      yaptığım denemelerde donanım temelli sorun olduğu fikrine vardım. Çeşitli hilelerle yamamaya çalıştığım betikleri `SIGINT` devre di;i biraksam da
      donma oluyor.
    + Ağ yapılandırması başarılı olursa dağıtım setleri için yansı seçimi safhasına geçiliyor. Paket temelli kurulumda ise sunucular zaten FreeBSD'nin global yansıdan
      en yakın yansıya otomatik yönlendiriliyor
  * Bölümleme safhasına geçiliyor.
    + Kurulum aracı cihaz üreticisi, ürün modeli, anakart modeline bakarak özel çözümleme yapılma gerekilip gerekilmediğini kestirmeye çalışıyor. Burada aldığı bağzı kararlar
      sistem seçilen özelliği desteklese dahi onu yanlış algıladığı için hata veriyor.
    + Bölümleme otomatik `ufs`, otomatik `zfs`, kabuk ortamı bölümleme ve elle `partedit` aracılığıyla seçilmesi isteniyor. Otomatik kurulumlarda
      tüm disk kesinlikle seçilmemeli, aksi halde diskte tüm yüklü işletim sistemleri silinecektir. Eğer imzalamak için diskte yedeklemediğiniz imza
      sertifikalarınız bulunuyorsa makineniz kilitlenebilir, servise açtırmaya götürmeniz gerekecektir.
    + Otomatik `zfs` veri setleri kurulumlarda `ROOT`, `ROOT/default`, `/home`, `/tmp`, `/usr`, `/usr/ports`, `/usr/src`, `/var`, `/var/audit`, `/var/crash`,
      `/var/log`, `/var/mail`, `/var/tmp` veri setleri ayarlanıyor. Bazılarında `exec=off`, `setuid=off` gibi güvenlik ayarları yapılarak söz konusu veri setlerinde belirli kısıtlamalar
       etkinleştiriliyor.
    + Elle bölümlemelerde kullanıcılardan bağlamak yapmak istedikleri konumlamaları ve bağlama ayarlarıyapmasına olanak veriyor. Yapılan ayarların sağlıklı olup olmadığı değerlendiriliyor.
    + Kabuk ortamında biçimlendirmelerde kullanıcıdan gerekli havuz ve veri setlerini oluşturup söz konusu kök ağacı `/mnt` altına bağlanıp, dosya sistemi tablolamayı `/tmp/bsdinstall_etc/fstab`
      dosyasına eklenmesi istenilmektedir.
    + Bölümleme bitince bölümler `/mnt` altın bağlanmaya çalışılıyor.
  * Dağıtım setleri / paketler sisteme yüklenilmeye çalışılıyor. Ağ veya yerel depoda sorun oluşmazsa seçilen yazılımları kuruyor.
  * Önyükleyici ayarlaması yapılıyor. UEFI `x86_64` sistemlerde `/boot/loader.efi` dosyasını ESP bölümünde `/efi/freebsd/loader.efi` konuma kopyalayıp
    `efibootmgr` aracını kullanarak anakartın sistem değişkenlerine FreeBSD'nin önyükleyicisini ekliyor.
  * Kök kullanıcı şifresi ayarlaması yapılıyor. "Skip" seçeneği kullanılırsa yönetici şifre boş oluyor.
  * Çevrimdışı yüklemelerde Ağ yapılandırma ayarlarını tekrar soruyor. Eğer kablosuz ağ ayarlama aşamasında donanımınız cevap vermşyorsa bu kurulumu burada asılı halde bırakacaktır.
    Zorlayarak tekrar başlattığınızda kurulum bitmemiş konumda olacaktır.
  * Zaman dilimi ayarlaması, tarih ve saat kontrolü `tzsetup` aracılığıyla yapılıyor.
  * Açılışta `sshd` (güvenli kabuk sunucusu), `ntpd` (zaman eşitlemesi), `local-unbound` (DNS yerel belleği yönetimi), `powerd` (işlemci frekansını otomatik ayarlama),
    `moused` uçbirimde PS/2 fare ayarlama, `virtual_oss` (sanal ses aygıtı oluşturma ve yok etme), `dumpdev` çekirdek çökmelerini `/var/crash` kusma
    ayarlamaları yapılmaktadır.
  * Güvenlik ayarlamaları yapılandırma:
    + çekirdek `sysctl` ayarlamaları: diğer kullanıcılara ait işlemlerin gizlenmesi (`security.bsd.see_other_uids=0`), diğer gruplara ait işlemlerin gizlenmesi (`security.bsd.see_other_gids=0`),
      kodeste bulunan işlemlerin gizlenmesi (`security.bsd.see_jail_proc=0`), `dmesg` yetkisiz kişilerden kısıtlanması (`security.bsd.unprivileged_read_msgbuf=0`),
      yetkisiz kişilerden işlem hata takibi kısıtlanması (`security.bsd.unprivileged_proc_debug=0`), işlemlere rasgele kimlik atanması (`kern.randompid=1`),
    + rc init sistemi ayarlamaları: `/tmp` açılışta temizlenmesi, `syslogd` ağ socketi devre dışı bırakma (uzaktan kayıt alma)
    + sanal uçbirimi `güvensiz` işaretleyerek tek kullanıcı kipine yönetici şifresiz oturum açması engellenmesi
    + önyükleyici `loader.conf` ayarları: DTrace destructive-mode kipini kapatma (`security.bsd.allow_destructive_dtrace=0`) ve yetkisiz
      kullanıcıların çekirdek ortam değişkenlerini değiştirmesi engelleme (`security.bsd.unprivileged_kenv_read=0`)
  * Donanım yazılımı yüklemesi. `fwsetup` koşarak depolarda uygun donanım modülü var mı yok mu aratılıp, ağ bağlantısı varsa sisteme yükleniyor
  * Sisteme interactive modta kullanıcı ekleme. Kullanıcı adı, gerçek ad, giriş sınıfı, ait olduğu gruplar, şifre, ev dizini, ev dizini izinleri soruluyor. Kullanıcıları
    yönetici gruplarına ekleme yönetici vasıfları sağlamasına yol açar: `wheel` (`su` komutunu çalıştırma hakkı), `operator` (ada* disklere erişim hakkı, yedekleme, sistemi kapatma hakkı),
    `video` grubuna ekleme ise drm aygitlarına erişim hakkı verir.
  * Çıkıştan bir önceki durak: "Finish" seçerek çıkışa ilerleme, tekrar kullanıcı ekleme, kök kullanıcı parola seçimi, makine adı, ağ yapılandırması, açılış hizmetleri etkinleştirme, güvenlik ayarlamaları,
  * zaman dilimi seçimi, donanım yazılımı yükleme ve belegelendirme yükleme. Belgelendirme ağ bağlantısı gerektirir.
- Kurulum başarıyla tamamlanınca ayarlar kaydediliyor
- Elle müdahale yapılmak istenilip istenilmediği soruluyor.
- Sistemi kapatma ve yeniden bailatma menüsü geliyor.  
  
     
