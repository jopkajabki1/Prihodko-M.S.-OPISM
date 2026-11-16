<!--
Бланк звіту до лабораторної роботи «TLS та сертифікати».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота 2. TLS та сертифікати

| Поле | Значення |
|---|---|
| ПІБ | Приходько Максим Сергійович |
| Група | F5 2.02 |
| Ідентифікатор | cs-202-21 |
| Дата виконання | 16.11.2026 |
| Середовище | GitHub Codespaces |
| Версія OpenSSL | OpenSSL 3.5.8 25 Aug 2026 |

## Параметри варіанта

<!-- Переписати з картки stand/cards/<id>.md. -->

| Параметр | Значення |
|---|---|
| Віддалений сервер | `front.cs-202-21.lab` |
| Імена для сертифіката | `www.cs-202-21.lab`, `status.cs-202-21.lab` |
| Ваш сервер (IP) | `10.53.1.1` |

<!-- Якщо стенд перезапускався під час роботи, вказати, після якого завдання. -->

## Завдання 1. HTTP і HTTPS у дампі трафіку

Вивід кроку 1.1:

```text
server: web
Host: front.cs-202-21.lab
SNI: 
protocol: HTTP/1.1 
client: 10.53.1.1
10 packets captured
10 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 80' > t1-http.txt
11:19:20.285890 IP 10.53.1.1.59828 > 10.53.0.80.80: Flags [P.], seq 1:83, ack 1, win 502, options [nop,nop,TS val 4170700716 ecr 1849487437], length 82: HTTP: GET / HTTP/1.1
....n<.MGET / HTTP/1.1
Host: front.cs-202-21.lab
server: web
Host: front.cs-202-21.lab
```

Вивід кроку 1.2:

```text
server: web
Host: front.cs-202-21.lab
SNI: front.cs-202-21.lab
protocol: HTTP/2.0 TLSv1.3
client: 10.53.1.1
23 packets captured
23 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t1-https.txt
0
1
```

| Що шукали в дампі | HTTP | HTTPS |
|---|---|---|
| рядок запиту `GET /` | 1 | 0 |
| заголовок `Host` | 2 | 0 |
| тіло відповіді (`server: web`) | є | нема |
| ім'я віддаленого сервера (кількість входжень) | 2 | 1 |

1. У якому повідомленні і в якому полі передається ім'я сервера, яке видно в дампі HTTPS?<br>У ClientHello (перший запис Handshake після TCP) — розширення server_name (SNI).

2. Чому рядок `SNI:` у відповіді сервера порожній для HTTP і заповнений для HTTPS?<br>SNI — це $ssl_server_name, ім'я з TLS-рукопотискання; у HTTP немає TLS, тому там порожньо.

3. Рядок `protocol:` показує HTTP/1.1 для запиту по HTTP і HTTP/2.0 для запиту по HTTPS, хоча обидва запити виконано однією командою `curl` без додаткових ключів. Що визначило версію HTTP у другому випадку?<br>ALPN: nginx слухає listen 443 ssl http2, curl через HTTPS пропонує h2,http/1.1, сервер обирає h2. По HTTP/1.1 поверх TCP — ALPN немає.

## Завдання 2. Рукостискання по записах

Вивід кроку 2.1 (TLS 1.3):

```text
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=330
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=118
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    EncryptedExtensions, Length=15
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Certificate, Length=987
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    CertificateVerify, Length=74
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Alert (21)
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
ALPN protocol: h2
Verify return code: 0 (ok)
```

Вивід кроку 2.2 (TLS 1.2):

```text
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=225
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=74
Received Record
  Content Type = Handshake (22)
    Certificate, Length=982
Received Record
  Content Type = Handshake (22)
    ServerKeyExchange, Length=111
Received Record
  Content Type = Handshake (22)
    ServerHelloDone, Length=0
Sent Record
  Content Type = Handshake (22)
    ClientKeyExchange, Length=33
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = Handshake (22)
    NewSessionTicket, Length=214
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = ApplicationData (23)
Sent Record
  Content Type = Alert (21)
New, TLSv1.2, Cipher is ECDHE-ECDSA-AES256-GCM-SHA384
ALPN protocol: h2
    Verify return code: 0 (ok)
```

<!-- Один рядок на кожен запис з виводу 2.1, у тому самому порядку. Напрям: К→С або С→К. -->

| № | Напрям | `Content Type` | `Inner Content Type` | Повідомлення | `Length` |
|---|---|---|---|---|---|
| 1 | К→С | Handshake (22) | — | ClientHello | 330 |
| 2 | С→К | Handshake (22) | — | ServerHello | 118 |
| 3 | С→К | ChangeCipherSpec (20) | — | ChangeCipherSpec | - |
| 4 | С→К | ApplicationData (23) | Handshake (22) | EncryptedExtensions | 15 |
| 5 | С→К | ApplicationData (23) | Handshake (22) | Certificate | 987 |
| 6 | С→К | ApplicationData (23) | Handshake (22) | CertificateVerify | 74 |
| 7 | С→К | ApplicationData (23) | Handshake (22)  | Finished | 48 |
| 8 | К→С | ChangeCipherSpec (20) | — | ChangeCipherSpec | - |
| 9 | К→С | ApplicationData (23) | Handshake (22) | Finished | 48 |
| 10 | К→С | ApplicationData (23) |  Alert (21)| Alert | - |

Вивід кроків 2.3 і 2.4:

```text
        extension_type=server_name(0), length=24
          0000 - 00 16 00 00 13 66 72 6f-6e 74 2e 63 73 2d 32   .....front.cs-2
          000f - 30 32 2d 32 31 2e 6c 61-62                     02-21.lab
        extension_type=ec_point_formats(11), length=4

21 packets captured
21 packets received by filter
0 packets dropped by kernel
TLS 1.3: 0

21 packets captured
21 packets received by filter
0 packets dropped by kernel
TLS 1.2: 2
```

| Показник | TLS 1.3 | TLS 1.2 |
|---|---|---|
| Довжина ClientHello, байт | 330 | 229 |
| Номер першого запису, тип якого спостерігач не бачить | 4 | немає |
| Кількість RTT від ClientHello до першого HTTP-запиту | 1 | 2 |
| Чи видно сертифікат у дампі (результат 2.4) | ні | так 0 |
| Обраний протокол ALPN | h2 | h2 |

1. За якими рядками виводу 2.1 визначено, з якого запису починається шифрування? Що бачить спостерігач замість типу кожного наступного запису?<br>За рядками Content Type = ApplicationData (23) — далі Inner Content Type показує, що всередині (спостерігач бачить лише «ApplicationData»).

2. Чому ClientHello у TLS 1.3 довший, ніж у TLS 1.2? Назвіть розширення, яке за рахунок цього скорочує рукостискання на один RTT.<br>Довше через key_share (плюс supported_versions, signature_algorithms, PSK-розширення); саме key_share прибирає зайвий RTT.

3. Довжина розширення `server_name` у виводі 2.3 становить 17 байт для імені з 12 символів. З яких частин складається це значення і яким воно буде для вашого імені?<br>server_name: 2 байти довжина списку + 1 байт тип + 2 байти довжина імені = 5 + 12 = 17. Для front.cs-202-21.lab (19 символів) → 24 байти.

4. Навіщо в TLS 1.3 передається запис ChangeCipherSpec, якщо в рукостисканні він нічого не змінює?<br>ChangeCipherSpec у TLS 1.3 — «пустишка» заради сумісності.

## Завдання 3. Сертифікат для власного сервера

Вивід кроків 3.1 і 3.2:

```text
subject=CN = Pebble Root CA 7c4918
notBefore=Nov 16 11:11:56 2026 GMT
notAfter=Nov 16 11:11:56 2056 GMT

curl: (60) SSL certificate problem: self-signed certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

Вивід кроку 3.3 (останні рядки `lego` і рядки журналу nginx):

```text
2026/11/16 12:01:33 [INFO] [www.cs-202-21.lab, status.cs-202-21.lab] acme: Obtaining SAN certificate
2026/11/16 12:01:33 [INFO] [status.cs-202-21.lab] AuthURL: https://ca.lab:14000/authZ/Vra9iEmuXkBqzI7Pu5KOGSIgqvwOOWiE7V-OlckMFQs
2026/11/16 12:01:33 [INFO] [www.cs-202-21.lab] AuthURL: https://ca.lab:14000/authZ/nfl4sqW-_W3lrc3DJ6QW3pgA344BUJUzTvQhyNh14gc
2026/11/16 12:01:33 [INFO] [www.cs-202-21.lab] acme: Could not find solver for: tls-alpn-01
2026/11/16 12:01:33 [INFO] [www.cs-202-21.lab] acme: use http-01 solver
2026/11/16 12:01:33 [INFO] [status.cs-202-21.lab] acme: Could not find solver for: tls-alpn-01
2026/11/16 12:01:33 [INFO] [status.cs-202-21.lab] acme: use http-01 solver
2026/11/16 12:01:33 [INFO] [www.cs-202-21.lab] acme: Trying to solve HTTP-01
2026/11/16 12:01:37 [INFO] [www.cs-202-21.lab] The server validated our request
2026/11/16 12:01:37 [INFO] [status.cs-202-21.lab] acme: Trying to solve HTTP-01
2026/11/16 12:01:44 [INFO] [status.cs-202-21.lab] The server validated our request
2026/11/16 12:01:44 [INFO] [www.cs-202-21.lab, status.cs-202-21.lab] acme: Validations succeeded; requesting certificates
2026/11/16 12:01:44 [INFO] Wait for certificate [timeout: 30s, interval: 500ms]
2026/11/16 12:01:44 [INFO] [www.cs-202-21.lab] Server responded with a certificate.
10.53.0.10 - - [16/Nov/2026:12:01:33 +0000] "GET /.well-known/acme-challenge/i0eVOf_uwSEMzIe9i-1L3079vt2gTRPxiwuUWDWxxdg HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:12:01:33 +0000] "GET /.well-known/acme-challenge/i0eVOf_uwSEMzIe9i-1L3079vt2gTRPxiwuUWDWxxdg HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:12:01:33 +0000] "GET /.well-known/acme-challenge/i0eVOf_uwSEMzIe9i-1L3079vt2gTRPxiwuUWDWxxdg HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:12:01:37 +0000] "GET /.well-known/acme-challenge/6v5Sp5FhW66mFgC3wmvYLYip_1NncSfLK8EHuru_1bk HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:12:01:37 +0000] "GET /.well-known/acme-challenge/6v5Sp5FhW66mFgC3wmvYLYip_1NncSfLK8EHuru_1bk HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:12:01:37 +0000] "GET /.well-known/acme-challenge/6v5Sp5FhW66mFgC3wmvYLYip_1NncSfLK8EHuru_1bk HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
```

Вивід кроку 3.4:

```text
subject=CN = www.cs-202-21.lab
issuer=CN = Pebble Intermediate CA 478265
notBefore=Nov 16 12:01:44 2026 GMT
notAfter=Nov 16 12:01:43 2031 GMT
serial=01DFE5C8D696CF8D
X509v3 Subject Alternative Name: 
    DNS:www.cs-202-21.lab, DNS:status.cs-202-21.lab

subject=CN = Pebble Intermediate CA 478265
issuer=CN = Pebble Root CA 7c4918
```

Вивід кроку 3.5:

```text
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/16 12:03:58 [notice] 99#99: signal process started
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
Certificate chain
 0 s:CN = www.cs-202-21.lab
   i:CN = Pebble Intermediate CA 478265
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 16 12:01:44 2026 GMT; NotAfter: Nov 16 12:01:43 2031 GMT
---
Verify return code: 21 (unable to verify the first certificate)
```

Вивід кроку 3.6:

```text
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/16 12:14:17 [notice] 125#125: signal process started
server: cs-202-21 (https)
Host: www.cs-202-21.lab
SNI: www.cs-202-21.lab
protocol: TLSv1.3
server: cs-202-21 (https)
Host: status.cs-202-21.lab
SNI: status.cs-202-21.lab
protocol: TLSv1.3
Certificate chain
 0 s:CN = www.cs-202-21.lab
   i:CN = Pebble Intermediate CA 478265
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 16 12:01:44 2026 GMT; NotAfter: Nov 16 12:01:43 2031 GMT
 1 s:CN = Pebble Intermediate CA 478265
   i:CN = Pebble Root CA 7c4918
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 16 11:11:56 2026 GMT; NotAfter: Nov 16 11:11:56 2056 GMT
---
Verify return code: 0 (ok)
```

| Параметр | Значення |
|---|---|
| Видавець сертифіката сервера | Pebble Intermediate CA 478265 |
| Видавець сертифіката видавця | Pebble Root CA 7c4918 |
| Імена в SAN | `www.cs-202-21.lab`, `status.cs-202-21.lab` |
| Строк дії (від – до) | `Nov 16 12:01:43 2031 GMT` `Nov 16 11:11:56 2056 GMT` |
| Тип ключа сервера і алгоритм підпису |  `id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256` `rsaEncryption, 2048 (bit); sigalg: RSA-SHA256` |
| Кількість сертифікатів, які надсилає сервер, у кроці 3.5 і в кроці 3.6 | 3.5 - 1   3.6 - 2 |

1. Хто і як перевірив, що ви контролюєте обидва імена? Вкажіть рядки виводу, які це підтверджують.<br>Центр сертифікації ACME (LetsEncrypt-Pebble-VA). За допомогою валідатора протоколу ACME через метод HTTP-01, звертаючись до NGINX за спеціальними шляхами.<br>`2026/11/16 12:01:33 [INFO] [www.cs-202-21.lab] acme: Trying to solve HTTP-01`<br>`2026/11/16 12:01:37 [INFO] [www.cs-202-21.lab] The server validated our request`<br>`10.53.0.10 - - [...] "GET /.well-known/acme-challenge/... HTTP/1.1" 200 87`

2. Чому в кроці 3.5 перевірка не пройшла, хоча корінь центру сертифікації вказано в `--cacert`, а сертифікат виданий правильно?<br>Сервер NGINX надіслав лише один сертифікат (кінцевий), але не передав проміжний (Intermediate CA). Через це клієнт (curl) не зміг побудувати повний ланцюжок до довіреного кореня з --cacert, хоча сам корінь був налаштований.

3. У виводі ланцюжка для сертифіката сервера вказано `id-ecPublicKey` і `RSA-SHA256`. Що означає кожне з цих значень і чому вони не суперечать одне одному?<br>id-ecPublicKey — тип власного ключа сертифіката (ECDSA P-256), RSA-SHA256 — алгоритм підпису видавця над цим сертифікатом. Це різні речі, вони не суперечать одна одній.

4. Що станеться з перевіркою вашого сервера після перезапуску стенда і чому? Що в реальній інфраструктурі відповідає цій ситуації?<br>Після перезапуску стенда Pebble перестворює корінь — сертифікат перестає проходити перевірку, тому потрібно перевипустити його (кроки 3.1, 3.3, 3.6) та зафіксувати це у звіті. У реальності цьому відповідає ротація кореневого ЦС / короткоживучі сертифікати та автооновлення за протоколом ACME.

## Завдання 4. Діагностика несправностей

Вивід кроку 4.1:

```text
=== f1.cs-202-21.lab
curl: (60) SSL: no alternative certificate subject name matches target host name 'f1.cs-202-21.lab'
Certificate chain
 0 s:CN = legacy.cs-202-21.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 16 11:11:54 2026 GMT; NotAfter: Feb 14 11:11:54 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 16 11:11:54 2026 GMT; NotAfter: Nov 15 11:11:54 2031 GMT
---
Verify return code: 0 (ok)
=== f2.cs-202-21.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f2.cs-202-21.lab
   i:O = Foreign Org, CN = Foreign Intermediate CA
   v:NotBefore: Nov 16 11:11:54 2026 GMT; NotAfter: Feb 14 11:11:54 2027 GMT
 1 s:O = Foreign Org, CN = Foreign Intermediate CA
   i:O = Foreign Org, CN = Foreign Root CA
   v:NotBefore: Nov 16 11:11:54 2026 GMT; NotAfter: Nov 15 11:11:54 2031 GMT
---
Verify return code: 20 (unable to get local issuer certificate)
=== f3.cs-202-21.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f3.cs-202-21.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 16 11:11:54 2026 GMT; NotAfter: Feb 14 11:11:54 2027 GMT
---
Verify return code: 21 (unable to verify the first certificate)
=== f4.cs-202-21.lab
curl: (60) SSL certificate problem: certificate has expired
Certificate chain
 0 s:CN = f4.cs-202-21.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Apr 30 11:11:54 2026 GMT; NotAfter: Oct 27 11:11:54 2026 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 16 11:11:54 2026 GMT; NotAfter: Nov 15 11:11:54 2031 GMT
---
Verify return code: 10 (certificate has expired)
=== f5.cs-202-21.lab
server: web, f5
Certificate chain
 0 s:CN = f5.cs-202-21.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 16 11:11:54 2026 GMT; NotAfter: Feb 14 11:11:54 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 16 11:11:54 2026 GMT; NotAfter: Nov 15 11:11:54 2031 GMT
---
Verify return code: 0 (ok)
```

| Сервер | Помилка `curl` | `Verify return code` | Яка перевірка не пройшла | Рядок виводу, що це доводить | Що має змінити власник сервера |
|---|---|---|---|---|---|
| f1 | `no alternative certificate subject name matches target host name` | `0 (ok)` | `Перевірка відповідності імені сервера сертифікату` | `no alternative certificate subject name matches target host name 'f1.cs-202-21.lab'` | `Додати правильне доменне ім'я до SAN (Subject Alternative Name) сертифіката або отримати новий сертифікат.` |
| f2 | `unable to get local issuer certificate` | `20 (unable to get local issuer certificate)` | `Перевірка довіри до видавця` | `Verify return code: 20 (unable to get local issuer certificate)` | `Замінити сертифікат на виданий довіреним для клієнтів ЦСа` |
| f3 | `unable to get local issuer certificate` | `21 (unable to verify the first certificate)` | `Перевірка ланцюжка сертифікатів (сервер надсилає лише 1 сертифікат без проміжного Intermediate CA)` | `Verify return code: 21 (unable to verify the first certificate)` | `Налаштувати конфігурацію вебсервера на передачу повного ланцюжка сертифікатів` |
| f4 | `certificate has expired` | `10 (certificate has expired)` | `Первірка строку дії сертифікату` |`NotAfter: Oct 27 11:11:54 2026 GMT` | `Оновити сертифікат` |
| f5 | `0k` | `немає` | - | `server: web, f5   Verify return code: 0 (ok)` | - |

1. Для сервера з помилкою `unable to get local issuer certificate` визначте, яка це несправність: сервер не надсилає проміжний сертифікат чи сертифікат виданий центром, якому клієнт не довіряє. Чим ці дві несправності відрізняються у виводі `openssl s_client`?<br>У f3 сервер не надсилає проміжний сертифікат — у виводі openssl s_client ланцюжок містить лише 1 сертифікат (0 s:...) і Verify return code: 21 (unable to verify the first certificate).
У f2 сертифікат видано недовіреним ЦС — ланцюжок повний (2 сертифікати 0 і 1), але Verify return code: 20 (unable to get local issuer certificate).

2. Якщо серед ваших серверів є сервер з невідповідним ім'ям, чому `openssl s_client` повертає для нього `Verify return code: 0 (ok)`, а `curl` повідомляє про помилку? Якщо такого сервера немає, дайте відповідь для загального випадку.<br>openssl s_client виконує лише криптографічну перевірку ланцюжка і за замовчуванням не перевіряє збіг імені хоста (Hostname validation), тоді як curl вимагає обов'язкової відповідності доменного імені полям SAN сертифіката.

3. Які з п'яти серверів відповіли б на запит `curl -k`? Що при цьому втрачається порівняно з `--cacert`?<br>Відповідають усі 5 серверів. Повністю втрачається захист від атак «Людина посередині» (Man-in-the-Middle, MitM), оскільки клієнт ігнорує будь-які проблеми з сертифікатами.

## Завдання 5. Ціна нового з'єднання

**Прогноз** (записати до вимірювання). Скільки RTT мине до завершення кожного етапу:

| Етап | TLS 1.3, нове з'єднання | TLS 1.2, нове з'єднання | Повторне використання з'єднання |
|---|---|---|---|
| `time_connect` | 1 | 1 | 0 |
| `time_appconnect` | 2 | 3 | 0 |
| `time_starttransfer` | 3 | 4 | 1 |

Вивід кроків 5.2 і 5.3:

```text
0.101281 0.151715 0.208276 0.258775 0.258865 2 1
0.050602 0.100756 0.155162 0.205596 0.205728 2 1
0.050566 0.100777 0.155119 0.205627 0.205699 2 1

0.050617 0.100803 0.205380 0.255995 0.256123 2 1
0.050612 0.100789 0.205842 0.256326 0.256424 2 1
0.050567 0.100792 0.204910 0.255361 0.255472 2 1

0.050618 0.100774 0.155186 0.205674 0.205760 2 1
0.000022 0.000000 0.000000 0.050335 0.050416 2 0
0.000021 0.000000 0.000000 0.050934 0.051030 2 0
```

<!-- Значення в мілісекундах. Для нових з'єднань — медіана трьох вимірювань. -->

| Запит | `namelookup` | `connect` | `appconnect` | `starttransfer` | `total` | `num_connects` |
|---|---|---|---|---|---|---|
| TLS 1.3, нове | 50,6 | 100,8 | 155,2 | 205,6 | 205,7 | 1 |
| TLS 1.2, нове | 50,6 | 100,8 | 205,4 | 256,0  | 256,1 | 1 |
| 5.3, перший запит | 50,6 | 100,8 | 155,2 | 205,7 | 205,8 | 1 |
| 5.3, другий запит | 0,02 | 0 | 0 | 51,0 | 50,4 | 0 |
| 5.3, третій запит | 0,02 | 0 | 0 | 50,9 | 50,3 | 0 |

RTT, визначений вимірюванням, і з якого значення його визначено:  connect − namelookup ≈ 50 мс

1. Виразіть кожне виміряне значення через RTT. Де вимірювання відрізняється від цілого числа RTT і чим це пояснюється?<br>
Усі метрики `curl` є кумулятивними (від початку запиту `t = 0`):

Для **нових з'єднань**:
- `time_namelookup` ≈ **1 RTT** (`50,6 мс`) — один DNS-запит/відповідь (лише у найпершому запуску 5.2 було `101,3 мс` ≈ **2 RTT**);
- `time_connect` ≈ **2 RTT** (`100,8 мс`) — 1 RTT на DNS + 1 RTT на TCP-рукостискання;
- `time_appconnect` (TLS 1.3) ≈ **3 RTT + ~5 мс** (`155,2 мс`) — 2 RTT (`connect`) + 1 RTT на TLS 1.3 рукостискання;
- `time_appconnect` (TLS 1.2) ≈ **4 RTT + ~5 мс** (`205,4 мс`) — 2 RTT (`connect`) + 2 RTT на TLS 1.2 рукостискання;
- `time_starttransfer` / `time_total` (TLS 1.3) ≈ **4 RTT + ~5–6 мс** (`205,6 / 205,7 мс`) — `appconnect` + 1 RTT на HTTP-запит і відповідь;
- `time_starttransfer` / `time_total` (TLS 1.2) ≈ **5 RTT + ~6 мс** (`256,0 / 256,1 мс`) — `appconnect` + 1 RTT на HTTP-запит і відповідь.

Для **повторного використання з'єднання** (крок 5.3, запити 2 і 3):
- `time_namelookup` ≈ **0 RTT** (`0,02 мс`), `time_connect` = **0 RTT**, `time_appconnect` = **0 RTT** — DNS не виконується, а TCP/TLS-сесія перевикористовується;
- `time_starttransfer` / `time_total` ≈ **1 RTT** (`50,3–51,0 мс`) — лише HTTP-запит і відповідь.

**Чому є відхилення від цілого числа RTT:**
Основна похибка (~**5 мс**) з'являється на етапі TLS-рукостискання (`appconnect − connect` становить `54,4 мс` замість `50 мс` у TLS 1.3 і `104,6 мс` замість `100 мс` у TLS 1.2), і далі вона переноситься в `starttransfer` і `total`. Це витрати процесорного часу на асиметричну криптографію (генерація/обмін ключами ECDHE, формування й перевірка підписів) та обробку TLS на клієнті й сервері. Інші переходи (`connect − namelookup ≈ 50,2 мс`, `starttransfer − appconnect ≈ 50,4–50,6 мс`) практично точно дорівнюють 1 RTT (лише +0,2–0,6 мс на локальну обробку стека й HTTP).

2. Чому `time_namelookup` не близький до нуля, хоча резолвер уже має ім'я в кеші? За яких умов він був би приблизно вдвічі більшим?<br>namelookup не нульовий, тому що під час роботи через netem запит до резолвера теж іде по мережі; вдвічі більшим він був би за холодного кешу / двох запитів (A + повтор) або якщо резолвер розташований далеко.

3. Сервіс виконує 200 послідовних запитів до цього сервера. Розрахуйте, скільки часу займе лише очікування мережі, якщо кожен запит встановлює нове з'єднання TLS 1.3, нове з'єднання TLS 1.2 і якщо всі запити йдуть через одне з'єднання.<br>TLS 1.3 (нове з'єднання щоразу): 200 * 205.7 мс = 41,140 мс (≈41.1 с).<br>TLS 1.2 (нове з'єднання щоразу): 200 * 256.1 мс = 51,220 мс (≈51.2 с).<br>Через одне з'єднання (Connection Reuse): 205.8 мс + 199 * 50.4 мс = 10,235.4 мс (≈10.2 с).

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:<br>У завданні 5 власний прогноз для `time_connect`, `time_appconnect` і `time_starttransfer` (1, 2 і 3 RTT для TLS 1.3) виявився на 1 RTT меншим за реальні вимірювання (рядки `0.050602 0.100756 0.155162 0.205596 0.205728 2 1` і `0.050617 0.100803 0.205380 0.255995 0.256123 2 1`): `time_connect` склав ~2 RTT (`100,8 мс`), а не 1 RTT, тому що `curl` вимірює всі значення кумулятивно від початку запиту, а DNS-запит (`time_namelookup` ≈ 50,6 мс) у середовищі з `netem` також витратив 1 RTT. Крім того, несподіваним було те, що в завданні 4 для `f1.cs-202-21.lab` команда `openssl s_client` повернула `Verify return code: 0 (ok)`, хоча `curl` відхилив з'єднання з помилкою `SSL: no alternative certificate subject name matches target host name 'f1.cs-202-21.lab'`.

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено і що з'ясувати не вдалося:<br>Не до кінця зрозуміло, чому в найпершому вимірюванні кроку 5.2 (рядок `0.101281 0.151715 0.208276 0.258775 0.258865 2 1`) етап `time_namelookup` тривав `~101,3 мс` (тобто 2 RTT), тоді як у всіх наступних запусках із новим з'єднанням він стабільно займав `~50,6 мс` (1 RTT). За умовами стенда затримка DNS може подвоюватися при холодному кеші резолвера або через послідовні запити `A`/`AAAA`, проте з одного лише виводу `curl` (без окремого дампа трафіку `tcpdump` на UDP/TCP-порт 53) не вдалося перевірити, скільки саме DNS-пакетів було відправлено під час першого запиту.

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | DeepSeek V4.1 Flash | поясни як робить роботу і що куди вставлять + бланк + методичка + посилання на форк + мій snn | використовувались відповіді на питання |
| 1 | Gemini 4 Hydrohen | перевір чи все ок та придумай щось для висновку |  |
