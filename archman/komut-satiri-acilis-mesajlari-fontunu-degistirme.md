_Not: Makale 26 Kasım 2022 s. 14:48 TRT saatinde [Archman OS](https://archman.org/topluluk/belgeler/nasil-komut-satiri-acilis-mesajlarinin-fontunu-degistirme/) bünyesinde yayımlanmıştır.
Söz konusu tarihten beri geçen süre içinde güncelliğini yitirmiş olabilir._

Arch Linux'un önyükleme başlangıç ram dosya sistemi (`initramfs`) oluşturan `mkinitcpio` komutu aynı isimli paketle sistemlerimizde varsayılan olarak kurulu gelmektedir.  
Türkçe'yi veya dilediğimiz başka dilleri destekleyen yazı biçimi ekleyerek açılış esnasında sistem mesajlarını düzgün göstermesini sağlayabiliriz. Eklediğimiz yazı biçimi dosyası
`.psf`, `.psfu`, `.fnt` uzantılı olmalıdır, dosya ayrıca `.gz` biçiminde sıkıştırılmış da olabilir.

Yapmamız gereken:  
+ `/etc/vconsole.conf` dosyasına yazı biçimi dosyasının adını belirleme:  
```
KEYMAP=trq
FONT=LatArCyrHeb-16+
```
+ `/etc/mkinitcpio.conf` dosyasına `consolefont` kancasını eklemek:
```
HOOKS=(base udev autodetect modconf block filesystems keyboard <strong>consolefont</strong> fsck)
```
ve  
```
$ sudo mkinitcpio -P
```  
koşmak.

Özel uçbirim yazı biçimi (font) tercihiniz varsa `/usr/share/kbd/consolefonts` dizinine yükleyip belirleyebilirsiniz. Ancak yaptığımız değişimler sistem dosyalarıyla çakışmamasına özen göstermeliyiz.
