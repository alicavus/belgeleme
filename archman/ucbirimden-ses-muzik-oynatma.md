_Not: Makale 31 Ocak 2023 s. 12:00 TRT saatinde [Archman OS](https://archman.org/topluluk/belgeler/nasil-ucbirimden-ses-muzik-oynatma/) bünyesinde yayımlanmıştır.
Söz konusu tarihten beri geçen süre içinde güncelliğini yitirmiş olabilir._

Arka planda ses oynatmak için illa ki ayrı grafik arabirimden pencere açmanıza gerek yoktur. Eğer çalışırken arka planda radyo veya beğendiğiniz bir parça dinlemek istiyorsanız bunu çeşitli yöntemleri vardır.
İşe odaklı çalışmalarımızda fazladan pencereler dikkatimizi dağıtabilir. Elbette "istemediğimiz" pencereleri farklı çalışma masaüstü alanlarına taşıyabiliriz.  
Ben uçbirimden arka planda çalan radyo dinlemesini severim. Fazla müdahale gerektirmez, ilave pencere vs. gerektirmez.  
Varsayılan olarak `mpv` uygulaması ses oynatırken pencere açmaz. Bu da arka planda rahatça müzik dinlememize olanak sağlar:  
```
$ mpv ses-kaynagı
```

Uçbirimden başlattığımız uygulamalar uşbirim kapanınca sonlanır. İşte bunu aşmak için nohup komutunu kullanacağız. Aşağıdaki örnekte İskeçe'den yayın yapan Kral FM'i uçbirimden dinleme ayarlanmaktadır:  
```
$ nohup mpv http://onairmediagroup.live24.gr/kralfm100xanthi/listen.pls
```  
ya da uçbirimin `disown` özelliğinden yararlanabiliriz:  
```
$ mpv http://onairmediagroup.live24.gr/kralfm100xanthi/listen.pls & disown
```  
Artık rahatça uçbirimi kapatabiliriz. Radyo yayını arkaplanda oynatılmaya devam edecektir.

Sıkılıp arka plandaki yayını kapatmak istersek görev yöneticisi / uçbirimden bunu sonlandırabiliriz. Uçbirimden sonlandırmak için:  
```
$ ps x | grep mpv
2994 ?        SLl    0:08 mpv http://onairmediagroup.live24.gr/kralfm100xanthi/listen.pls
```  
koşuyoruz.

İşlem kimlliği (`PID`) `2994` olduğu için bu değeri taşıyan işlemi sonlandıralım:  
```
$ kill -9 2994
```  
Eğer işlem değeri farketmeksizin sonlandırmak istiyorsak bir uygulamanın tüm işlemlerini sonlandırabiliriz. Bunun için işlem değeri filtrelememize gerek kalmadığı için direk olarak aşağıdaki  komutu girebiliriz:  
```
$ killall -9 mpv
```

Herekese keyifli ve bol müzikli çalışmalar dilerim :smile:
