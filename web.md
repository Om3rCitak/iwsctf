#Web (SQL Injection use Cookie)

Merhaba arkadaşlar. Adeo Intern & Work stajyer alımları için ufak bir ctf düzenlemişti. CTF'in bana göre en zevkli ve genelde katılımcıların çözümde en zorladığı sorunun sizler için anlatmak istedim. Başlayalım;

Soruyu açtığımızda bize bir txt dosyası veriyordu ve tt nin içinde bu bilgiler yer alıyor.

>http://ctf.adeo.com.tr/ adresine açtıktan sonra aşağıdaki kullanıcı adını ve şifreyi giriniz. 
>Soru: IWS üyelerimiz, DB ismi nedir ?
>http://ctf.adeo.com.tr/
>k.adi: adeo
>sifre: @deoP@ssW0rd
	
Yukarıdaki adrese gittiğimizde basit bir login form bizi bekliyor. verilen şifreler ile giriş yaptıktan sonra ekranda kullanıcı adımız ve şifremiz gözüküyor.

![cat](http://omercitak.net/iws/resim1.png) 

Burada ilk önce login formda sql inj olabilir diye birkaç deneme yaptım, login form sağlam çıktı. Ekran çok basitti, ne bir input nede kullanıcıdan veri alan başka birşey yoktu. Ozaman geriye kısıtlı seçenek kalıyor. Bu seçeneklerin başında session ve cookie geliyor. Browserdaki cookileri kontrol etmek için Firefox'un Cookie Manager adında güzel bir plugini var. Cookie edit işlemleriniz için kullanabilirsiniz ki bende onu kullanıyorum. Hemen cookie manageri açalım.

![cat](http://omercitak.net/iws/resim2.png) 

Gördüğümüz üzere ctf.adeo.com.tr domaini üzerinde "token" adında değeri "YWRlbw%3D%3D" olan bir cookie var. %3D, '=' karakterinin url decode hali. yani "YWRlbw==". be metni herhangi bir base64 decode aracı ile decode ettiğinizde "adeo" sonucuna ulaşıyorsunuz. Demekki arayüzde yazan bilgileri token cookiesinin değerini sorgulayarak çekiyorki burada bir güvenlik açığı oluşsun :)

hemen adeo' metnini (tırnak dahil) basa64 encode yapıp, cookie'nin yeni değer olarak kaydedelim ve sayfayı yenileyelim. 

![cat](http://omercitak.net/iws/resim3.png) 

Vee bum! sql inj tespit ettik. Tabi font rengi bgcolor ile aynı olduğunda ctrl+a ile rahatça yazan hatayı görebilirsiniz.
Şimdi bug'a ait payloadımızı yazıp, base64 encode edip gönderelim.

payload : 
```sql
adeo') and 1=0 union select 0,database(),0#
```

base64 encode hali : 
```
YWRlbycpYW5kIDE9MCB1bmlvbiBzZWxlY3QgMCxkYXRhYmFzZSgpLDAj
```

payload için açıklama : 
adeo'dan sonra "')" yazarak mevcut sorgunun parantezini kapamış olduk.
"and 1=0" ile, ilk parantezini kapadığımız sorgunun false dönüp, 2. sorgumuzun çalışmasını sağlamış olduk.
"union select 0,database(),0#" ilede bilindiği üzere mysql de database() fonksiyonu mevcut veritabanın ismini verir.
peki neden "0,database,0" ? Çünki veritabanında userların tutulduğu tabloda 3 kolon var. id, username, password. ekrana basılan ise sadece 2. ve 3. kolon. biz bu sorguda database() fonksiyonunu 2. kolona yazdık, username yerinde çıktı verdi. 3. kolon yerine yazsaydık password yerinde çıktı vermiş olacaktı.

![cat](http://omercitak.net/iws/resim4.png) 

ve gördüğünüz gibi database ismini çıktı olarak verdi. 

```
Flag : bugunhavagunesli
```

