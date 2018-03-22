---
tags:
- soft
- nginx
- security
comments: true
date: 2014-06-04T07:51:19Z
title: Лучшие настройки HTTPS (TLS) Nginx
lastmod: 2014-07-09T07:51:19Z
slug: ssl-tls-in-nginx
---

Для начала приведу [правильную](https://www.ssllabs.com/projects/best-practices/index.html "Qualys SSL Labs - Projects / SSL/TLS Deployment Best Practices") конфигурацию, которую вы можете утащить к себе. И, да, под HTTPS я [имею в виду](https://www.howsmyssl.com/s/about.html#tls-vs-ssl "About · How's My SSL?") [TLS](http://en.wikipedia.org/wiki/Transport_Layer_Security "Transport Layer Security — Wikipedia").

<!--more-->

---

{{< gist paskal 628882bee1948ef126dd >}}

---

Ниже разъяснение значимых моментов.

    ssl_certificate /etc/nginx/ssl/domain.net.pem;

Путь к общему сертификату, который должен содержать ваш сертификат, и сертификат авторизационного сервера, пример того, как его собрать:

    cat public_domain.net.pem sub.class1.server.ca.pem > domain.net.pem

Корневой сертификат (ca.pem), эта цепочка содержать не должна, так как он в любом случае есть в списке доверенных сертификатов сервера. Советую подумать о [самоподписанном сертификате](http://vitus-wagner.livejournal.com/916596.html "vitus_wagner: Зачем вам подорожная, хамы? Вы же неграмотны!").

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

Протоколы общения клиента и сервера. SSLv3 небезопасен, на замену ему создан TLSv1, который мы и используем. Отказом от SSLv3 мы теряем поддержку IE6, по данным Cloudfare на май 2014 года это [0.000002%](http://blog.cloudflare.com/the-web-is-world-wide-or-who-still-needs-rc4/ "The Web is World-Wide, or who still needs RC4? | CloudFlare Blog") трафика (292 уникальных USER-AGENT). [RC4](http://blog.cloudflare.com/killing-rc4/ "Killing RC4 (softly) | CloudFlare Blog") [must](http://blog.cloudflare.com/tracking-our-ssl-configuration/ "Tracking our SSL configuration | CloudFlare Blog") [die](http://blog.cloudflare.com/killing-rc4-the-long-goodbye/ "Killing RC4: The Long Goodbye | CloudFlare Blog"), ваш браузер давно [поддерживает более безопасное шифрование](https://www.howsmyssl.com/ "How's My SSL?").

    add_header Strict-Transport-Security max-age=31536000;

Надолго (считайте, навечно), говорим клиенту, что общаться с этим сервером можно только и только по HTTPS.

    add_header X-Frame-Options DENY;

Запрещаем показ сайта в frame, iframe, object, защищает от man in the middle атаки ([explanation](https://developer.mozilla.org/en-US/docs/Web/HTTP/X-Frame-Options "The X-Frame-Options response header — HTTP | MDN")).

    ssl_prefer_server_ciphers on;

Предпочитаем выбор метода шифрования сервера, поскольку на нём мы можем выбрать, какие методы использовать предпочтительней ([docs](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers "Module ngx_http_ssl_module"), [ru_docs](http://nginx.org/ru/docs/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers "Модуль ngx_http_ssl_module")).

    ssl_ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS;

Только безопасные методы шифрования, [Forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy "Forward secrecy — Wikipedia, the free encyclopedia") где это возможно.

---

Бонусом полезные в nginx вещи, не относящиееся к ssl.

    listen 443 deferred spdy ssl;
    listen [::]:443 deferred ssl spdy ipv6only=on;

Слушаем порт 443 IPv4 и IPv6 на всех интерфейсах, deferred ускоряет работу соединений linux-сервера ([info](http://www.techrepublic.com/article/take-advantage-of-tcp-ip-options-to-optimize-data-transmission/ "Take advantage of TCP/IP options to optimize data transmission — TechRepublic")), spdy включает использование [быстрого](http://blog.chromium.org/2013/11/making-web-faster-with-spdy-and-http2.html "Chromium Blog: Making the web faster with SPDY and HTTP/2") протокола [SPDY](https://en.wikipedia.org/wiki/SPDY "SPDY — Wikipedia"), если клиент поддерживает это ([docs](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen "Module ngx_http_core_module"), [ru_docs](http://nginx.org/ru/docs/http/ngx_http_core_module.html#listen "Модуль ngx_http_core_module"), [результат внедрения SPDY Яндексом](http://habrahabr.ru/company/yandex/blog/222951/ "Совместный эксперимент команд Яндекс.Почты и Nginx: действительно ли SPDY ускорит интернет? / Блог компании Яндекс / Хабрахабр"))

server {
	listen 80 default_server;
	listen [::]:80 default_server ipv6only=on;
	server_name _;
	return 301 https://$host$request_uri;
}

Здесь приведён правильный способ перенаправить домен с www на домен без www (или наоборот). default_server в [listen](http://nginx.org/ru/docs/http/ngx_http_core_module.html#listen "Модуль ngx_http_core_module") означает что, если не сработал иной блок, будет использован этот. В данном случае — мы перенаправим пользователя, откуда бы он не пришёл, на `https://domain.net`.

[Результат теста безопасности](https://www.ssllabs.com/ssltest/analyze.html?d=terrty.net "Qualys SSL Labs — Projects / SSL Server Test / terrty.net") сервера с такими настройками.

Результат таких настроек: в Firefox главная страница этого сайта была получена за 180мс.

**Huge thanks to [Hynek Schlawack](https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/ "Hardening Your Web Server’s SSL Ciphers")**.
