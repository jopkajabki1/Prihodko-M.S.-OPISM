# Практична робота № 1

**Дисципліна:** Основи побудови інформаційних систем та мереж

**Тема:** Спостереження за процесом звернення до вебресурсу. Побудова власної моделі рівнів взаємодії

| | |
|---|---|
| **Прізвище, ім'я** | Приходько Максим |
| **Група** | F5 2.02 |
| **Номер варіанта** | 22 |
| **Домен варіанта** | dragonflybsd.org |
| **Середовище виконання** | Linux |
| **Версія curl** | 8.21.0 |
| **Дата виконання** | 09.09.2026 |

---

## Частина A. Збір експериментальних даних

### A.1. Запит із діагностичним виводом

**Команда:**

```
curl -v https://dragonflybsd.org
```

**Вивід:**

```
* Host dragonflybsd.org:443 was resolved.
* IPv6: 2001:470:1:43b:1::67
* IPv4: 199.233.90.67
*   Trying [2001:470:1:43b:1::67]:443...
* Immediate connect fail for 2001:470:1:43b:1::67: Network is unreachable
* connect to 2001:470:1:43b:1::67 port 443 from :: port 0 failed: Network is unreachable
*   Trying 199.233.90.67:443...
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* SSL Trust Anchors:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
*   CApath: /etc/ssl/certs
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / x25519 / id-ecPublicKey
* ALPN: server accepted http/1.1
* Server certificate:
*   subject: CN=dragonflybsd.org
*   start date: Jul 11 10:19:06 2026 GMT
*   expire date: Oct  9 10:19:05 2026 GMT
*   issuer: C=US; O=Let's Encrypt; CN=YE1
*   Certificate level 0: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
*   Certificate level 1: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
*   Certificate level 2: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
*   Certificate level 3: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
*   subjectAltName: "dragonflybsd.org" matches cert's "dragonflybsd.org"
* OpenSSL verify result: 0
* SSL certificate verified via OpenSSL.
* Established connection to dragonflybsd.org (199.233.90.67 port 443) from 192.168.188.129 port 33230
* using HTTP/1.x
> GET / HTTP/1.1
> Host: dragonflybsd.org
> User-Agent: curl/8.21.0
> Accept: */*
>
* Request completely sent off
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
< HTTP/1.1 302 Found
< Date: Wed, 09 Sep 2026 12:22:28 GMT
< Server: Apache/2.4.55 (DragonFly) OpenSSL/1.1.1t
< Location: https://www.dragonflybsd.org/
< Content-Length: 213
< Content-Type: text/html; charset=iso-8859-1
<
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href="https://www.dragonflybsd.org/">here</a>.</p>
</body></html>
* Connection #0 to host dragonflybsd.org:443 left intact)
```

---

### A.2. Запит без захисту з'єднання

**Команда:**

```
curl -v http://neverssl.com
```

**Вивід:**

```
* Host neverssl.com:80 was resolved.
* IPv6: 2600:1f13:37c:1400:ba21:7165:5fc7:736e
* IPv4: 34.223.124.45
*   Trying [2600:1f13:37c:1400:ba21:7165:5fc7:736e]:80...
* Immediate connect fail for 2600:1f13:37c:1400:ba21:7165:5fc7:736e: Network is unreachable
* connect to 2600:1f13:37c:1400:ba21:7165:5fc7:736e port 80 from :: port 0 failed: Network is unreachable
*   Trying 34.223.124.45:80...
* connect to 34.223.124.45 port 80 from 192.168.188.129 port 44468 failed: Connection refused
* Failed to connect to neverssl.com:80 after 21124 ms: Could not connect to server
* closing connection #0
curl: (7) Failed to connect to neverssl.com:80 after 21124 ms: Could not connect to serve
```

---

### A.3. Запит до служби доменних імен

*Windows: `Resolve-DnsName ВАШ_ДОМЕН`*

**Команда (перше виконання):**

```
dig dragonflybsd.org
```

**Вивід:**

```
; <<>> DiG 9.20.26-1~deb13u1-Debian <<>> dragonflybsd.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3296
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; MBZ: 0x0005, udp: 512
;; QUESTION SECTION:
;dragonflybsd.org.              IN      A

;; ANSWER SECTION:
dragonflybsd.org.       5       IN      A       199.233.90.67

;; Query time: 82 msec
;; SERVER: 192.168.188.2#53(192.168.188.2) (UDP)
;; WHEN: Wed Sep 09 15:25:59 EEST 2026
;; MSG SIZE  rcvd: 61
```

**Команда (повторне виконання через 5–7 хвилин):**

```
dig dragonflybsd.org
```

**Вивід:**

```
; <<>> DiG 9.20.26-1~deb13u1-Debian <<>> dragonflybsd.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55910
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; MBZ: 0x0005, udp: 512
;; QUESTION SECTION:
;dragonflybsd.org.              IN      A

;; ANSWER SECTION:
dragonflybsd.org.       5       IN      A       199.233.90.67

;; Query time: 86 msec
;; SERVER: 192.168.188.2#53(192.168.188.2) (UDP)
;; WHEN: Wed Sep 09 15:32:42 EEST 2026
;; MSG SIZE  rcvd: 61
```

**Зафіксовані значення:**

| Параметр | Перше виконання | Повторне виконання |
|---|---|---|
| Час виконання (год:хв) | 15:25 | 15:32 |
| IP-адреса | 199.233.90.67 | 199.233.90.67 |
| Значення TTL | 5 | 5 |

> Якщо друге значення TTL виявилося більшим за перше — це нормально: кеш резолвера встиг оновитися. Зафіксуйте як є.

---

### A.4. Контрольний ресурс

**Команда:**

```
curl -v https://google.com
```

**Вивід:**

```
* Host google.com:443 was resolved.
* IPv6: 2a00:1450:4025:807::64, 2a00:1450:4025:807::65, 2a00:1450:4025:807::66, 2a00:1450:4025:807::8b
* IPv4: 142.251.98.113, 142.251.98.101, 142.251.98.102, 142.251.98.138, 142.251.98.100, 142.251.98.139
*   Trying [2a00:1450:4025:807::64]:443...
* Immediate connect fail for 2a00:1450:4025:807::64: Network is unreachable
* connect to 2a00:1450:4025:807::64 port 443 from :: port 0 failed: Network is unreachable
*   Trying 142.251.98.113:443...
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* SSL Trust Anchors:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
*   CApath: /etc/ssl/certs
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519MLKEM768 / id-ecPublicKey
* ALPN: server accepted h2
* Server certificate:
*   subject: CN=*.google.com
*   start date: Aug 10 08:37:42 2026 GMT
*   expire date: Nov  2 08:37:41 2026 GMT
*   issuer: C=US; O=Google Trust Services; CN=WE2
*   Certificate level 0: Public key type EC/prime256v1 (256/128 Bits/secBits), signed using ecdsa-with-SHA256
*   Certificate level 1: Public key type EC/prime256v1 (256/128 Bits/secBits), signed using ecdsa-with-SHA384
*   Certificate level 2: Public key type EC/secp384r1 (384/192 Bits/secBits), signed using ecdsa-with-SHA384
*   subjectAltName: "google.com" matches cert's "google.com"
* OpenSSL verify result: 0
* SSL certificate verified via OpenSSL.
* Established connection to google.com (142.251.98.113 port 443) from 192.168.188.129 port 40100
* using HTTP/2
* [HTTP/2] [1] OPENED stream for https://google.com/
* [HTTP/2] [1] [:method: GET]
* [HTTP/2] [1] [:scheme: https]
* [HTTP/2] [1] [:authority: google.com]
* [HTTP/2] [1] [:path: /]
* [HTTP/2] [1] [user-agent: curl/8.21.0]
* [HTTP/2] [1] [accept: */*]
> GET / HTTP/2
> Host: google.com
> User-Agent: curl/8.21.0
> Accept: */*
>
* Request completely sent off
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
< HTTP/2 301
< location: https://www.google.com/
< content-type: text/html; charset=UTF-8
< content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-ljlzBeHSkVelmarU7fz7pA' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
< date: Wed, 09 Sep 2026 12:38:28 GMT
< expires: Fri, 09 Oct 2026 12:38:28 GMT
< cache-control: public, max-age=2592000
< server: gws
< content-length: 220
< x-xss-protection: 0
< x-frame-options: SAMEORIGIN
< alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
<
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
* Connection #0 to host google.com:443 left intact
```

---

### A.5. Ресурси з некоректною конфігурацією сертифіката

**Випадок 1**

```
curl -v https://expired.badssl.com
```

```
*   Trying 104.154.89.105:443...
* Host expired.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* SSL Trust Anchors:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
*   CApath: /etc/ssl/certs
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256 / secp256r1 / rsaEncryption
* ALPN: server accepted http/1.1
* Server certificate:
*   subject: OU=Domain Control Validated; OU=PositiveSSL Wildcard; CN=*.badssl.com
*   start date: Apr  9 00:00:00 2015 GMT
*   expire date: Apr 12 23:59:59 2015 GMT
*   issuer: C=GB; ST=Greater Manchester; L=Salford; O=COMODO CA Limited; CN=COMODO RSA Domain Validation Secure Server CA
*   Certificate level 0: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
*   Certificate level 1: Public key type RSA (2048/112 Bits/secBits), signed using sha384WithRSAEncryption
*   Certificate level 2: Public key type RSA (4096/152 Bits/secBits), signed using sha384WithRSAEncryption
*   subjectAltName: "expired.badssl.com" matches cert's "*.badssl.com"
* OpenSSL verify result: a
* SSL certificate OpenSSL verify result: certificate has expired (10)
* closing connection #0
curl: (60) SSL certificate OpenSSL verify result: certificate has expired (10)
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the webpage mentioned above.
```

**Випадок 2**

```
curl -v https://wrong.host.badssl.com
```

```
* Host wrong.host.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
*   Trying 104.154.89.105:443...
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* SSL Trust Anchors:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
*   CApath: /etc/ssl/certs
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256 / secp256r1 / rsaEncryption
* ALPN: server accepted http/1.1
* Server certificate:
*   subject: CN=*.badssl.com
*   start date: Jul 28 20:03:02 2026 GMT
*   expire date: Oct 26 20:03:01 2026 GMT
*   issuer: C=US; O=Let's Encrypt; CN=YR2
*   Certificate level 0: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
*   Certificate level 1: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
*   Certificate level 2: Public key type RSA (4096/152 Bits/secBits), signed using sha256WithRSAEncryption
*   Certificate level 3: Public key type RSA (4096/152 Bits/secBits), signed using sha256WithRSAEncryption
*  subjectAltName does not match hostname wrong.host.badssl.com
* SSL: no alternative certificate subject name matches target hostname 'wrong.host.badssl.com'
* closing connection #0
curl: (60) SSL: no alternative certificate subject name matches target hostname 'wrong.host.badssl.com'
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the webpage mentioned above.
```

**Випадок 3**

```
curl -v https://self-signed.badssl.com
```

```
*   Trying 104.154.89.105:443...
* Host self-signed.badssl.com:443 was resolved.
* IPv6: (none)
* IPv4: 104.154.89.105
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* SSL Trust Anchors:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
*   CApath: /etc/ssl/certs
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256 / secp256r1 / rsaEncryption
* ALPN: server accepted http/1.1
* Server certificate:
*   subject: C=US; ST=California; L=San Francisco; O=BadSSL; CN=*.badssl.com
*   start date: Sep  8 21:00:17 2026 GMT
*   expire date: Sep  7 21:00:17 2028 GMT
*   issuer: C=US; ST=California; L=San Francisco; O=BadSSL; CN=*.badssl.com
*   Certificate level 0: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
*   subjectAltName: "self-signed.badssl.com" matches cert's "*.badssl.com"
* OpenSSL verify result: 12
* SSL certificate OpenSSL verify result: self-signed certificate (18)
* closing connection #0
curl: (60) SSL certificate OpenSSL verify result: self-signed certificate (18)
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the webpage mentioned above.
```

> Якщо використано альтернативний спосіб із параметром `--resolve` — зазначити це та навести фактичну команду.

---

## Частина B. Власна модель рівнів

**Кількість виділених груп:** ___

| № | Назва групи (власне формулювання) | Рядки виводу, віднесені до групи | Обґрунтування |
|---|---|---|---|
| 1 | відповідь сервера | HTML | сторінка сайту |
| 2 | DNS | | ip адреси |
| 3 | handshake | | встановлення зв'язку |
| 4 | перевірка сертифікатів | не вставив рядки бо "ламається" таблиця | |
| 5 | запит на сторінку | GET / HTTP/2 | |


*Групи впорядковано від найближчої до користувача (№ 1) до найближчої до апаратного забезпечення. Зайві рядки вилучити, за потреби — додати.*

**Рядки, які не вдалося віднести до жодної групи:**

| Рядок виводу | Причина утруднення |
|---|---|
| | |
| | |
| | |

---

## Контрольні питання

**1. Скільки рядків діагностичного виводу передує отриманню даних сторінки (завдання A.1)?**

> 53

**2. Які рядки наявні у виводі A.1 і відсутні у виводі A.2? Чим це зумовлено?**

> Багато адже в мене "Failed to connect to neverssl.com:80 after 21124 ms: Could not connect to server"

**3. Звідки у виводі з'явилося значення `443`, якщо його не було вказано в адресі?**

> DNS надав

**4. Як змінилося значення TTL між двома запитами (A.3)? Що означає це число?**

> Ніяк. Час у секундах протягом якого DNS зберігає кеш

**5. Чим відрізняються між собою три причини помилок із завдання A.5? Сформулювати кожну однією фразою.**

| Випадок | Причина недовіри |
|---|---|
| `expired` | старий сертифікат не гарантує захищеність сайту |
| `wrong.host` | домени не співпадають |
| `self-signed` | потрібна зовнішня незалежна оцінка |

**6. Три рядки з власних виводів, про які не йшлося на лекції 1:**

| № | Рядок виводу | Джерело (номер завдання) |
|---|---|---|
| 1 | ALPN: curl offers h2,http/1.1 | А.1 |
| 2 | GET / HTTP/1.1 | А.1 |
| 3 | та інші | |

*Пояснення до цих рядків не потрібне.*

---

## Висновки

Виконавши дану практичну роботу, я попрактикувався у навичках використання команд `curl` та `dig`. Під час виконання завдань я поглибив розуміння того, що відбувається, коли я заходжу на вебсайт. Я дізнався, як за допомогою `dig` можна отримати інформацію про DNS-записи домену, зокрема його IP-адресу та TTL. Також я зрозумів, що DNS використовується для перетворення доменного імені на IP-адресу, за якою комп’ютер може знайти потрібний сервер. За допомогою `curl` я навчився отримувати інформацію від вебсерверів та переглядати HTTP-відповіді. Практична робота допомогла краще зрозуміти взаємодію між клієнтом, DNS та вебсервером. Отримані навички є корисними для подальшого вивчення комп’ютерних мереж, адміністрування та кібербезпеки, а також допоможуть краще розуміти принципи роботи Інтернету.

**D.1. Що виявилося неочевидним або несподіваним**

*Назвати конкретно, з посиланням на рядок виводу.*

> Що TSL = 5    dragonflybsd.org.       5       IN      A       199.233.90.67

**D.2. Чому саме така кількість груп у частині B**

*На якій підставі ухвалено рішення. Що змусило б його змінити.*

> На підставі своєї необізнаності. Знав би більше -- написав

**D.3. Питання, яке залишилося без відповіді**

> Нема

---

## Використання штучного інтелекту

*Розділ обов'язковий. Заповнюється незалежно від того, чи використовувався ШІ. Детальні вимоги — у документі «Політика використання технологій штучного інтелекту».*

**Факт використання:** використано
**Установлений рівень для цієї роботи:** Р3 — ШІ як співвиконавець

**Фактичний рівень використання:** Р3 — ШІ як співвиконавець

### Використані системи

| Система | Версія або модель | Період використання |
|---|---|---|
|chatgpt | безкоштовна | 5 хв |

### Промпти

*Наводити дослівно, у тому вигляді, у якому запит було надано системі. Переказ не приймається.*

| № | Розділ роботи | Текст промпта |
|---|---|---|
| 1 | А.5 | dig і якийс днс видає ттл де воно |
| 2 | Виснововк | розшир висновок до 150 слів Виконавши дану практичну роботу я попрактикувався у навичках використання curl та dig. Поглибив розуміння того -- що відбувається коли я заходжу на сайт |
| 3 | ШІ | проаналізуй чат та визначи рівень Рівень Назва Зміст Р1 Без ШІ Робота виконується в контрольованих умовах, що виключають застосування ШІ. Оцінюється самостійне володіння матеріалом Р2 ШІ для підготовки ШІ допускається на етапі пошуку, добору джерел, структурування та планування. Кінцевий текст і висновки формулює студент самостійно Р3 ШІ як співвиконавець ШІ допускається для генерування ідей, чернеток, зворотного зв'язку та доопрацювання. Завдання побудовано так, що самого лише ШІ недостатньо для досягнення потрібного результату. Оцінюється також те, як студент оцінив, змінив та інтегрував отриманий від ШІ матеріал Р4 ШІ як основний інструмент Використання ШІ передбачено умовою завдання. Оцінюється критичне мислення та фахова компетентність, виявлені в керуванні системою ШІ Р5 Дослідницьке використання Завдання передбачає нестандартне застосування ШІ для розв'язання фахової задачі. Спосіб використання узгоджується з викладачем |

### Дії з отриманим результатом

| № промпта | Що перевірено | Що змінено | Що відхилено і чому |
|---|---|---|---|
| 1 | google.com.    300    IN    A    142.250.x.x | - | - |
| 2 | Висновок | - | - |
| 3 | | | |

### Підтвердження

Підтверджую, що всі наведені в цьому звіті виводи команд отримано мною особисто внаслідок фактичного виконання відповідних дій, а відомості цього розділу є повними та достовірними.

> Виводи `curl`, `dig` та інші артефакти не можуть бути згенеровані. Це стосується будь-якого рівня використання ШІ.

---

## Примітки виконавця

*(необов'язковий розділ: що не спрацювало, які команди довелося змінити, які виникли труднощі)*

> Неможу вставити рядки у таблицю В бо ламається структура
