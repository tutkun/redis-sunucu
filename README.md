## Redis, Laravel-Echo-Server ve Laravel Üçlüsü
Bu yazıda Laravel'in harika özellikleriyle birlikte `socket.io` kullanımına değineceğim.

### Nedir Socket.IO
Socket.IO uygulamanızda bir Giriş/Çıkış(IO) birimi olarak görev alır. `Laravel` uygulamanızdaki aksiyonları(eylemleri, olayları, event'leri v.s.) `Redis`'e JSON data olarak iletir. Örn: `{"message":"X yeni bir fotoğraf yükledi"}` şeklinde... `Redis` ile RAM üzerinde tutulan bu veriler `Redis` tarafından sıraya alınır. Bu arada `laravel-echo-server` sunucusu `Redis` üzerindeki bu hareketleri dinlemektedir. Bir eylem oluştuğunda tüm kullanıcılar bu haberi yollar. Tabii bu gönderim işlemi de kendi içinde 3'e ayrılmaktadır. `PrivateChannel`, `PublicChannel` ve `Channel`.

#### Channel (Herkese Açık Kanal)
"Ziyaretçi Sayısı" gibi bir şey için de kullanılabilir. Aslında siteye yetkili olmadan dahi giriş yapan her bir kullanıcıya; özetle herkese gösterilecek yayınları temsil eder.

###### Örnek kullanımı:
```javascript
Echo.channel('orders')
    .listen('OrderShipped', (e) => {
        console.log(e.order.name);
    });
```

#### Private Channel (Özel Kanal)
Bir kurala uyan birim kullanıcı(ya|lara) ilgili yayın(ı|ları) gönderir. Bu kural(lar), `/routes/channel.php` dosyasında belirtilir(ler). Belli bir zümreye özel oldukları için Özel Kanal(PrivateChannel) olarak ifade edilirler.

###### Örnek kullanımı:
```javascript
// Özel kanalın birden fazla olayı dinlemesi
Echo.private('orders')
    .listen(...)
    .listen(...)
    .listen(...);
```

#### Presence Channel (Bilgi [durum] Kanalı)
Mesaj gönderilmesine izin vermeyen; bunun yerine bulunduğu sohbet uygulamasında `online` kullanıcıların `durum`undan diğer kullanıcıları haberdar etmesi gibi alanlarda kullanılabilirler.

###### Örnek kullanımı:
```javascript
Echo.join(`chat.${roomId}`)
    .here((users) => {
        //
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    });
```

###### config/channel.php
```php
Broadcast::channel('chat.*', function ($user, $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

En kapsamlı anlatım sanırım [burada](https://komelin.com/articles/realtime-apps-laravel-echo-tips-and-tricks).

[Önemli bir kaynak](https://jplhomer.org/2017/01/building-realtime-chat-app-laravel-5-4-vuejs)

[Redis.io](www.redis.io/download) kısmından redis sunucu paketini indirip, uygun bir yere yerleştirin. Laravel paketi içerisinde olması gerekmez.
[Redis Kurulumu](https://www.cloudways.com/blog/redis-for-queuing-in-laravel-5)

[Laravel-Echo-Server](https://github.com/tlaverdure/laravel-echo-server) adresinden laravel ile uyumlu broadcast ve event işlemleri için kullanacağımız NodeJS tabanlı sunucuyu indiriyoruz. Bu, Redis ile Laravel arasında bağlantı kurmamızı sağlayacak.

[laravel-echo-server ve socket.io](http://stackoverflow.com/questions/43711067/laravel-echo-server-and-socket-io-on-homestead-get-typeerror-cannot-read-proper) kurulumları.

Uygulamaların [çalışan bir örneği](https://stackoverflow.com/questions/40049438/broadcasting-with-laravel-echo-laravel-echo-server-and-socket-io-wont-work) için bu referans alınabilir.

Echo'yu VueJS ile uyumlu çalışması ve güvenlik önlemlerine dahil olması için [yetkilendirmek](https://github.com/tlaverdure/laravel-echo-server/issues/129) gerekir.

### Yetkilendirme Seçenekleri
[JWT ile laravel-echo-server kullanımı](https://laravel.io/forum/10-09-2016-howto-broadcasting-laravel-echo-laravel-echo-server-and-jwt)

[Modern Rest Api Laravel](http://esbenp.github.io/2017/03/19/modern-rest-api-laravel-part-4)

Youtube'da [Fransızın Anlatımı](https://www.youtube.com/watch?v=WrsI6qf0KSA&feature=youtu.be&t=870&ab_channel=Grafikart.fr)

[Laravel 5.4](http://laravel.com/docs/master) ki bu zaten kurulu olacaktır :)

[predis/redis]() paketi de kurulu olmalı

[Event & Scheduling](https://mattstauffer.co/blog/laravel-5.0-event-scheduling) ile zamanlı çalıştırma

[Notification Örneği](https://github.com/cretueusebiu/laravel-web-push-demo/blob/master/app/Http/Controllers/NotificationController.php)

[Matt Stauffer](https://mattstauffer.co/blog/introducing-laravel-passport)'in güzel anlatımı.
