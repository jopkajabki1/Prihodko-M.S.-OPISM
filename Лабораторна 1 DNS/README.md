<!--
Бланк звіту до лабораторної роботи «DNS».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Коментарі такого вигляду в GitHub не відображаються; видаляти їх не обов'язково.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ | Приходько Максим Сергійович |
| Група | F5 2.02 |
| Ідентифікатор (`sNN`) | cs-202-21 |
| Дата виконання | 2026-10-26 |
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu |

## Параметри варіанта

<!-- Переписати з картки stand/cards/sNN.md. -->

| Параметр | Значення |
|---|---|
| `$TTL`, с | 75 с |
| TTL запису SOA, с | 60 с |
| Поле `minimum` SOA, с | 90 с |
| Псевдонім (CNAME на `www`) | files |
| Другий хост і його TTL, с | test, 25 с |
| Пріоритет MX | 40 |
| Токен TXT | 3e10b08c |
| Ім'я для завдання 5 | new-nxodo |

## Завдання 1. Складові URI

### 1.1. Розбір

<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | http | - | www.cs-202-21.lab | 80 | /reports/catalog | from=61&lang=x | comments |
| 2 | http | - | files.cs-202-21.lab | 80 | /catalog/ | from=%D0%BA%D0%B8%D1%97%D0%B2 | - |
| 3 | http | user:pass | test.cs-202-21.lab | 80 | /reports | - | results |
| 4 | https | - | www.cs-202-21.lab | 8443 | /reports/../catalog | - | - |
| 5 | mailto | - | - | -  | admin@cs-202-21.lab | subject=reports | - |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | www.cs-202-21.lab | www.cs-202-21.lab | /reports/catalog?from=61&lang=x |
| 2 | files.cs-202-21.lab | files.cs-202-21.lab | /catalog/?from=%D0%BA%D0%B8%D1%97%D0%B2 |
| 3 | test.cs-202-21.lab | test.cs-202-21.lab:8443 | /reports |
| 4 | www.cs-202-21.lab | www.cs-202-21.lab | /catalog |
| 5 | - | - | - |

Декодоване значення параметра URI 2:

Чому URI 5 не має складової host, хоча містить символ `@`:

### 1.2. Перевірка (після завдання 3)

<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->
<br><br>Вивід: `curl -v 'http://www.cs-202-21.lab/reports/catalog?from=61&lang=x#comments'`

```text
* Host www.cs-202-21.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to www.cs-202-21.lab (10.53.0.80) port 80
> GET /reports/catalog?from=61&lang=x HTTP/1.1
> Host: www.cs-202-21.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 26 Oct 2026 12:53:50 GMT
< Content-Type: text/plain
< Content-Length: 146
< Connection: keep-alive
< 
server: web-a
Host: www.cs-202-21.lab
request URI: /reports/catalog?from=61&lang=x
path: /reports/catalog
query: from=61&lang=x
client: 10.53.1.1
* Connection #0 to host www.cs-202-21.lab left intact

```
<br><br>Вивід: `curl -v 'http://files.cs-202-21.lab:80/catalog/?from=%D0%BA%D0%B8%D1%97%D0%B2'`

```text
* Host files.cs-202-21.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:80...
* Connected to files.cs-202-21.lab (10.53.0.80) port 80
> GET /catalog/?from=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: files.cs-202-21.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 26 Oct 2026 12:55:04 GMT
< Content-Type: text/plain
< Content-Length: 164
< Connection: keep-alive
< 
server: web-a
Host: files.cs-202-21.lab
request URI: /catalog/?from=%D0%BA%D0%B8%D1%97%D0%B2
path: /catalog/
query: from=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1
* Connection #0 to host files.cs-202-21.lab left intact
```
<br><br>Вивід: `curl -v 'http://user:pass@test.cs-202-21.lab/reports#results'`

```text
* Host test.cs-202-21.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to test.cs-202-21.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /reports HTTP/1.1
> Host: test.cs-202-21.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 26 Oct 2026 12:56:38 GMT
< Content-Type: text/plain
< Content-Length: 102
< Connection: keep-alive
< 
server: web-b
Host: test.cs-202-21.lab
request URI: /reports
path: /reports
query: 
client: 10.53.1.1
* Connection #0 to host test.cs-202-21.lab left intact
```

Значення заголовка `Authorization` і результат декодування:
<br><br>Вивід: `echo 'dXNlcjpwYXNz' | base64 -d`
```text
 user:pass
```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:

<br>TCP (DNS уже спрацював — ім'я розірвалося в 10.53.0.80; TLS не починався, тому що з'єднання не встановилося).

```text
* Host test.cs-202-21.lab:80 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.81
*   Trying 10.53.0.81:80...
* Connected to test.cs-202-21.lab (10.53.0.81) port 80
* Server auth using Basic with user 'user'
> GET /reports HTTP/1.1
> Host: test.cs-202-21.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 26 Oct 2026 12:56:38 GMT
< Content-Type: text/plain
< Content-Length: 102
< Connection: keep-alive
< 
server: web-b
Host: test.cs-202-21.lab
request URI: /reports
path: /reports
query: 
client: 10.53.1.1
* Connection #0 to host test.cs-202-21.lab left intact
root@ns-cs-202-21:/# curl -v 'https://www.cs-202-21.lab:8443/reports/../catalog'
* Host www.cs-202-21.lab:8443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:8443...
* connect to 10.53.0.80 port 8443 from 10.53.1.1 port 55418 failed: Connection refused
* Failed to connect to www.cs-202-21.lab port 8443 after 122 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.cs-202-21.lab port 8443 after 122 ms: Couldn't connect to server
```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний | /catalog | /catalog |
| `--path-as-is` | /reports/../catalog | /catalog |

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:<br>curl сам нормалізує .. перед надсиланням; з --path-as-is він надсилає все як є, але nginx усе одно нормалізує $uri, тому шлях однаковий.

## Завдання 2. Проходження ієрархії вручну

```text

```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 | NOERROR | qr | 0 | `lab.                    172800  IN      NS      ns1.nic.lab.` | `ns1.nic.lab.            172800  IN      A       10.53.0.3` |
| 10.53.0.3 | NOERROR | qr | 0 | `cs-202-21.lab.          3600    IN      NS      ns.cs-202-21.lab.` |  `ns.cs-202-21.lab.       3600    IN      A       10.53.1.1` |
| 10.53.1.1 | NOERROR | qr aa | `ns.cs-202-21.lab.       75      IN      A       10.53.1.1` | `cs-202-21.lab.          75      IN      NS      ns.cs-202-21.lab.` | - |

Вивід `dig ns.sNN.lab A +trace`:

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.cs-202-21.lab A +trace
;; global options: +cmd
.                       82330   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 1 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 81 ms

cs-202-21.lab.          3600    IN      NS      ns.cs-202-21.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 40 ms

ns.cs-202-21.lab.       75      IN      A       10.53.1.1
cs-202-21.lab.          75      IN      NS      ns.cs-202-21.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.cs-202-21.lab) in 0 ms

```

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 cs-202-21.lab NS +norec` | 3600 | - qr | cs-202-21.lab.          3600    IN      NS      ns.cs-202-21.lab. |
| `dig @10.53.1.1 cs-202-21.lab NS +norec` | 75 | qr aa | cs-202-21.lab.          75      IN      NS      ns.cs-202-21.lab. |

1. Які відповіді є відсиланнями і за якими ознаками це визначено? <br> Відсилки — відповіді root і TLD: NOERROR, немає aa, порожній ANSWER, NS дочірньої зони в AUTHORITY та glue в ADDITIONAL.

2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було? <br> ADDITIONAL — glue-адреса NS дочірньої зони, щоб не робити окремий запит. Без неї резолверу довелося б окремо резолвити ns.cs-202-21.lab, а це ім'я всередині тієї ж делегованої зони — замкнене коло.

3. Яка з двох копій запису NS є авторитативною і чому? <br> Авторитетна копія в ANSWER від 10.53.1.1 (aa, TTL 75 із зони). Копія в AUTHORITY від TLD — це лише вказівник на делегування (немає aa, TTL 3600).

## Завдання 3. Заповнення власної зони

Вивід `zone-reload`:

```
zone cs-202-21.lab/IN: loaded serial 2026102602
OK
zone reload queued
```

<!-- Шість запитів до 10.53.1.1 з +norec, по одному на кожен рядок таблиці картки. -->

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR | qr aa | 10.53.0.80 | 75 | так |
| псевдонім CNAME | NOERROR | qr aa | www.cs-202-21.lab. | 75 | так |
| `mail` A | NOERROR | qr aa | 10.53.0.80 | 75 | так |
| `@` MX | NOERROR | qr aa | 40 mail.cs-202-21.lab. | 75 | так |
| `@` TXT | NOERROR | qr aa | "lab-token=3e10b08c" | 75 | так |
| другий хост A | NOERROR | qr aa | 10.53.0.81 | 25 | так |

Порівняння відповідей резолвера та власного сервера:
<br><br><br>Вивід: `dig files.cs-202-21.lab`

```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> files.cs-202-21.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10476
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;files.cs-202-21.lab.           IN      A

;; ANSWER SECTION:
files.cs-202-21.lab.    75      IN      CNAME   www.cs-202-21.lab.
www.cs-202-21.lab.      75      IN      A       10.53.0.80

;; Query time: 41 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Oct 26 12:23:47 UTC 2026
;; MSG SIZE  rcvd: 82

```
<br><br>Вивід: `dig @10.53.1.1 files.cs-202-21.lab +norec`
```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 files.cs-202-21.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53719
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c02e2b20c4aa9e9d010000006adf46cbba4cfc9e88719c06 (good)
;; QUESTION SECTION:
;files.cs-202-21.lab.           IN      A

;; ANSWER SECTION:
files.cs-202-21.lab.    75      IN      CNAME   www.cs-202-21.lab.
www.cs-202-21.lab.      75      IN      A       10.53.0.80

;; AUTHORITY SECTION:
cs-202-21.lab.          75      IN      NS      ns.cs-202-21.lab.

;; ADDITIONAL SECTION:
ns.cs-202-21.lab.       75      IN      A       10.53.1.1

;; Query time: 3 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Oct 26 12:25:47 UTC 2026
;; MSG SIZE  rcvd: 143

```
<br><br>Вивід: `dig cs-202-21.lab MX`
```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> cs-202-21.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33660
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;cs-202-21.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-21.lab.          75      IN      MX      40 mail.cs-202-21.lab.

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Oct 26 12:29:45 UTC 2026
;; MSG SIZE  rcvd: 63

```
<br><br>Вивід: `dig @10.53.1.1 cs-202-21.lab MX +norec`
```
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 cs-202-21.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4868
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c3a7e75c3edd48ab010000006adf48a4b54b123aecb16f6f (good)
;; QUESTION SECTION:
;cs-202-21.lab.                 IN      MX

;; ANSWER SECTION:
cs-202-21.lab.          75      IN      MX      40 mail.cs-202-21.lab.

;; AUTHORITY SECTION:
cs-202-21.lab.          75      IN      NS      ns.cs-202-21.lab.

;; ADDITIONAL SECTION:
mail.cs-202-21.lab.     75      IN      A       10.53.0.80
ns.cs-202-21.lab.       75      IN      A       10.53.1.1

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Oct 26 12:33:40 UTC 2026
;; MSG SIZE  rcvd: 140

```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a | qr rd ra | 2(CNAME + A) | 0 | 0 | 75 |
| псевдонім через 10.53.1.1 | qr aa | 2(CNAME + A) | 1(NS) | 1(A) | 75 |
| MX через resolver-a | qr rd ra | 1(MX) | 0 | 0 | 75 |
| MX через 10.53.1.1 | qr aa | 1(MX) | 1(NS) | 2(A) | 75 |

Пояснення відмінностей:<br>Відмінності зумовлені роллю сервера: авторитативний віддає повний набір даних зони (aa, NS у AUTHORITY, glue у ADDITIONAL, повний TTL), а рекурсивний — лише ANSWER з кешу (rd/ra, без службових секцій, TTL зменшується). Самі значення в ANSWER однакові.

Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```
dns_master_load: /lab/zone/cs-202-21.lab.zone:22: cs-202-21.lab: CNAME and other data
zone cs-202-21.lab/IN: loading from master file /lab/zone/cs-202-21.lab.zone failed: CNAME and other data
zone cs-202-21.lab/IN: not loaded due to errors.

```

1. Який TTL має кожен запис зони і звідки це значення взято?<br>TTL для www, files, mail, MX, TXT, NS і ns становить 75 с із $TTL, для test — 25 с (задано явно), а для SOA — 60 с із власного поля; при цьому в резолвера TTL зменшується через залишок кешу, а в авторитативного сервера він завжди повний.

2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?<br>На запит A сервер сам знаходить CNAME у межах зони й додає A цілі (у відповіді 2 записи), тоді як на явний запит CNAME повертається лише 1 запис, причому резолвер віддає цей самий ланцюжок із кешу.

3. Чому запис CNAME неможливо розмістити для імені `@`?<br>CNAME для @ неможливий, бо на вершині зони вже є SOA і NS, а CNAME не може співіснувати з іншими записами

## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```text
www.cs-202-21.lab.      75      IN      A       10.53.0.80
www.cs-202-21.lab.      70      IN      A       10.53.0.80
```

**Прогноз** Формула: t = T_зміни + R, де R — залишок TTL у кеші на момент зміни. Найбільша можлива затримка — повний $TTL = 75 с.

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```text
root@ns-cs-202-21:/# dig @10.53.1.1 www.cs-202-21.lab +norec +short
10.53.0.80
```

Залишок R = 70 с (17:58:11), час zone-reload — 17:58:11, перевірка на сервері:

```text
17:58:11  www.cs-202-21.lab.  70  IN  A  10.53.0.80
zone-reload  ->  17:58:11
root@ns-cs-202-21:/# dig @10.53.1.1 www.cs-202-21.lab +norec +short
10.53.0.81
```

Очікуваний час появи нової адреси: 17:58:11 + 70 с = 17:59:21.

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```text
17:58:11 10.53.0.80
17:58:13 10.53.0.80
...
17:59:17 10.53.0.80
17:59:19 10.53.0.80
17:59:22 10.53.0.81
```

Рядок `server:` з відповіді `curl`:<br>server: web-b

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 17:58:11 |	17:59:21 | 17:59:22 |	71	| 70 |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?<br>Розбіжність +1 с: цикл опитує кожні 2 с, тому момент появи фіксується з точністю до 2 с, плюс час на zone-reload. Кеш резолвера відраховує TTL від моменту кешування (перший dig), а не від reload.

2. Якою була б найбільша можлива затримка для цього запису і за яких умов?<br>Найбільша затримка — 75 с, якщо зміну зробити одразу після кешування (R = 75), плюс до 2 с на опитування.

3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?<br>Для TTL 86400: заздалегідь зменшити TTL (наприклад, до 300 с), дочекатися, поки старі кеші істекут (не менше 86400 с), і лише потім змінювати адресу. Або тримати обидві адреси (два A), або перевести сервіс на нове ім'я. Чужі кеші не скинути.

## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення: min(TTL SOA, minimum) = min(60, 90) = 60 с.

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-nxodo.cs-202-21.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 7205
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: bd56d7ef9d6a895a010000006adf98beb5c28ac2736d7758 (good)
;; QUESTION SECTION:
;new-nxodo.cs-202-21.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-21.lab.          60      IN      SOA     ns.cs-202-21.lab. admin.cs-202-21.lab. 2026102612 3600 600 86400 90

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Oct 26 18:15:26 UTC 2026
;; MSG SIZE  rcvd: 125
```

Запит через resolver-a 18:15:50, час `zone-reload` 18:15:50, перевірка на сервері 10.53.0.80:
<br><br>date +%T  &  dig new-nxodo.cs-202-21.lab
```text
18:15:50

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> new-nxodo.cs-202-21.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 18767
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-nxodo.cs-202-21.lab.       IN      A

;; AUTHORITY SECTION:
cs-202-21.lab.          60      IN      SOA     ns.cs-202-21.lab. admin.cs-202-21.lab. 2026102612 3600 600 86400 90

;; Query time: 1 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Oct 26 18:15:50 UTC 2026
;; MSG SIZE  rcvd: 97
```
<br>zone-reload  &  date +%T
```
zone cs-202-21.lab/IN: loaded serial 2026102612
OK
zone reload queued
18:17:10
```
<br>dig @10.53.1.1 new-nxodo.cs-202-21.lab +norec +short
```
10.53.0.80
```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```text
18:18:11 NXDOMAIN
18:18:13 NXDOMAIN
...
18:19:29 NXDOMAIN
18:19:22 NOERROR
```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 18:18:11 | 18:19:22 | 18:19:21 | 61 | 60 |

<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?<br>У неіснуючого імені немає власного TTL — негативний TTL задаёт SOA зони (минимум из TTL SOA и minimum), RFC 2308.

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.<br>Да, совпадает: 60 = 60, потому что min(60, 90) = 60, а TTL SOA в файле тоже 60. Если бы minimum был меньше TTL SOA — в AUTHORITY было бы меньше.

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?<br>Скрипт до создания получит NXDOMAIN и закеширует его на 60 с; сразу после создания тот же резолвер всё ещё вернёт NXDOMAIN. Исправление: проверять напрямую авторитативный сервер (dig @10.53.1.1), либо сбрасывать кеш/ждать 60 с, либо создавать запись до первой проверки.

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | 0 msec | 0 msec |
| B: нове ім'я в зоні `lab.` | 41 msec | 0 msec |
| C: ім'я в неіснуючому TLD | 81 msec | 0 msec |
| D: `www.sNN.lab` після перезапуску резолвера | 402 msec | 0 msec |

Вивід вимірювань:
<br> A=y$RANDOM.cs-202-21.lab; dig $A | grep 'Query time'; dig $A | grep 'Query time'
```text
;; Query time: 0 msec
;; Query time: 0 msec
```
<br>B=x$RANDOM.lab; dig $B | grep 'Query time'; dig $B | grep 'Query time'
```text
;; Query time: 41 msec
;; Query time: 0 msec
```
<br>C=x$RANDOM.zz$RANDOM; dig $C | grep 'Query time'; dig $C | grep 'Query time'
```text
;; Query time: 81 msec
;; Query time: 0 msec
```
<br>dig www.cs-202-21.lab | grep 'Query time'  &  dig www.cs-202-21.lab | grep 'Query time'
```tex
;; Query time: 402 msec
;; Query time: 0 msec
```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | 0 msec | 0 msec | auth |
| B | 41 msec | 0 msec | TLD |
| C | 81 msec | 0 msec | root |
| D | 402 msec | 0 msec | root + TLD + auth |

Значення `time_namelookup` з `curl` після перезапуску резолвера: 0.403551

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:
<br> docker compose -f stand/compose.yaml exec root tail -n 20 /var/log/named/queries.log
```text
26-Oct-2026 17:48:29.970 client @0x713b18006078 10.53.0.53#11173 (.): query: . IN NS -E(0)DC (10.53.0.2)
26-Oct-2026 17:48:30.051 client @0x713b0c06ac88 10.53.0.53#27218 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
26-Oct-2026 17:48:30.131 client @0x713b0c06ac88 10.53.0.53#59370 (ns.cs-202-21.lab): query: ns.cs-202-21.lab IN A -E(0)DC (10.53.0.2)
26-Oct-2026 17:48:30.616 client @0x713b18006078 10.53.0.54#10142 (.): query: . IN NS -E(0)DC (10.53.0.2)
26-Oct-2026 17:48:30.697 client @0x713b0c06ac88 10.53.0.54#35763 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
26-Oct-2026 17:48:30.778 client @0x713b0c06ac88 10.53.0.54#35809 (ns.cs-202-21.lab): query: ns.cs-202-21.lab IN A -E(0)DC (10.53.0.2)
26-Oct-2026 18:35:52.534 client @0x713b0c06ac88 10.53.0.53#57113 (x1104.zz26215): query: x1104.zz26215 IN A -E(0)DC (10.53.0.2)
26-Oct-2026 18:36:41.383 client @0x713b0c06ac88 10.53.0.53#16154 (.): query: . IN NS -E(0)DC (10.53.0.2)
26-Oct-2026 18:36:41.463 client @0x713b18006078 10.53.0.53#11478 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
26-Oct-2026 18:36:41.544 client @0x713b18006078 10.53.0.53#18141 (www.cs-202-21.lab): query: www.cs-202-21.lab IN A -E(0)DC (10.53.0.2)
26-Oct-2026 18:43:29.515 client @0x713b0c06ac88 10.53.0.53#14769 (.): query: . IN NS -E(0)DC (10.53.0.2)
26-Oct-2026 18:43:29.595 client @0x713b18006078 10.53.0.53#14565 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
26-Oct-2026 18:43:29.676 client @0x713b0c06ac88 10.53.0.53#58052 (www.cs-202-21.lab): query: www.cs-202-21.lab IN AAAA -E(0)DC (10.53.0.2)
26-Oct-2026 18:43:29.676 client @0x713b18006078 10.53.0.53#45087 (www.cs-202-21.lab): query: www.cs-202-21.lab IN A -E(0)DC (10.53.0.2)
```
<br>docker compose -f stand/compose.yaml exec tld tail -n 20 /var/log/named/queries.log
```text
26-Oct-2026 17:48:30.252 client @0x738f0c06ac88 10.53.0.53#24233 (ns.cs-202-21.lab): query: ns.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 17:48:30.899 client @0x738f0c06ac88 10.53.0.54#37647 (ns.cs-202-21.lab): query: ns.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 17:51:25.354 client @0x738f0c06ac88 10.53.0.53#20297 (new2-kqjwe.cs-202-21.lab): query: new2-kqjwe.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 18:32:46.676 client @0x738f04001ee8 10.53.0.53#19284 (x12099.lab): query: x12099.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 18:36:41.664 client @0x738f04001ee8 10.53.0.53#5899 (www.cs-202-21.lab): query: www.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 18:43:29.796 client @0x738f0c06ac88 10.53.0.53#64853 (www.cs-202-21.lab): query: www.cs-202-21.lab IN A -E(0)DC (10.53.0.3)
26-Oct-2026 18:43:29.796 client @0x738f04001ee8 10.53.0.53#7582 (www.cs-202-21.lab): query: www.cs-202-21.lab IN AAAA -E(0)DC (10.53.0.3)
```

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.<root ≈ значення рядка C, TLD ≈ рядка B, свій ≈ рядка A; D ≈ їхня сума (що підтверджується зростанням часу запиту).>

2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?<br>A не проходить через root/TLD, оскільки делегування cs-202-21.lab (NS + glue) вже є у кеші резолвера з попередніх завдань.

3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?<br>П'ять доменів — п'ять процесів вирішення; перші можуть відбуватися паралельно, але кожен новий домен може вимагати повного проходження ієрархії, тому перше відкриття відбувається найповільніше (+ TCP/TLS). Повторне — з кешу.

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:<br>resolver-a після закінчення TTL → SERVFAIL; resolver-b (serve-stale) → старий A з TTL 0.

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```text
test.cs-202-21.lab.     25      IN      A       10.53.0.81
test.cs-202-21.lab.     25      IN      A       10.53.0.81
18:55:01
```

Запити із заміром часу (крок 5):

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 test.cs-202-21.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 11803
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.cs-202-21.lab.            IN      A

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Oct 26 18:56:10 UTC 2026
;; MSG SIZE  rcvd: 47


real    0m0.024s
user    0m0.009s
sys     0m0.002s

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 test.cs-202-21.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 36270
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.cs-202-21.lab.            IN      A

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Oct 26 18:56:10 UTC 2026
;; MSG SIZE  rcvd: 47


real    0m0.022s
user    0m0.006s
sys     0m0.005s

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.54 test.cs-202-21.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51479
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.cs-202-21.lab.            IN      A

;; ANSWER SECTION:
test.cs-202-21.lab.     30      IN      A       10.53.0.81

;; Query time: 1801 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Mon Oct 26 18:56:12 UTC 2026
;; MSG SIZE  rcvd: 63


real    0m1.824s
user    0m0.005s
sys     0m0.008s
```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | SERVFAIL | - | 0m0.024s |
| resolver-a, другий | SERVFAIL | - | 0m0.022s |
| resolver-b | NOERROR | 30 | 0m1.824s |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text
18:59:10
18:59:10 10.53.0.81
```

Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):

```text
ns.cs-202-21.lab. admin.cs-202-21.lab. 2026102612 3600 600 86400 90
```

### Постмортем

1. Що сталося: <br>Стався штучний інцидент із повною недоступністю власного авторитативного сервера (auth), який обслуговує доменну зону cs-202-21.lab. Це призвело до того, що під час запитів нових або застарілих даних резолвери втратили джерело актуальної інформації.
2. Вплив на клієнтів resolver-a і resolver-b:<br>Для клієнтів resolver-a: Повний збій обслуговування (помилка резолюції). Користувачі або програми не зможуть отримати IP-адресу хоста, що призведе до помилок підключення (наприклад, неможливості відкрити вебсайт або завантажити ресурси). <br>Для клієнтів resolver-b: Послуга залишилася доступною без помітних перерв для користувачів. Резолвер повернув старі дані з кешу (NOERROR), хоча сам запит виконувався дещо довше через таймаут спроби опитування недоступного сервера.
3. Хронологія з виміряним часом:<br>18:55:01 — Зафіксовано кешування запису test.cs-202-21.lab з початковим TTL (наприклад, 25 або 30 секунд) в обох резолверах.<br>~18:55:30 — Сплив час життя TTL запису в кеші, а власний сервер був зупинений.<br>18:56:10 — Запит до звичайного резолвера (resolver-a, @10.53.0.53) повернув статус SERVFAIL за час близько 0.024s / 0.022s (сервер швидко здався, оскільки записи прострочені, а опитати авторитет неможливо).<br>18:56:12 — Запит до резолвера з serve-stale (resolver-b, @10.53.0.54) успішно повернув стару адресу 10.53.0.81 із застарілим TTL за час 1.824s (затримка зумовлена очікуванням таймауту звернення до зупиненого сервера перед видачею stale-даних).<br>18:59:10 — Час успішного перезапуску авторитативного сервера (named-start) та момент відновлення повноцінної роботи системи (цикл спостереження зафіксував першу успішну відповідь 10.53.0.81).
4. Чому кеш допоміг або не допоміг:<br>Звичайний кеш (resolver-a) не допоміг, оскільки суворо дотримується стандартів TTL: після закінчення терміну дії запису він вважається невалідним, і резолвер намагається оновити його на авторитетному сервері. Оскільки сервер лежав, резолвер видав помилку SERVFAIL. Stale-кеш (resolver-b) допоміг, оскільки реалізує механізм serve-stale (RFC 8767). Якщо авторитетний сервер недоступний або відповідає з помилкою, резолвер не відкидає прострочений запис одразу, а продовжує віддавати його клієнтам, запобігаючи каскадним збоям у роботі інфраструктури.
5. Що слід змінити:<br>Увімкнути serve-stale глобально: Для підвищення відмовостійкості мережі налаштувати функцію serve-stale (або stale-answer-enable) на всіх робочих резолверах.<br>Моніторинг та алертинг: Налаштувати систему моніторингу (наприклад, Prometheus + Grafana або системи сповіщень) на доступність авторитетних серверів зон, щоб оперативно реагувати на їхню зупинку.<br>Оптимізація TTL: Збільшити час TTL для критично важливих незмінних записів, щоб знизити навантаження і підвищити стійкість кешу під час короткочасних мережевих аварій.

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?<br>TTL 0 — serve-expired повертає застарілі дані, але позначає TTL 0, щоб клієнти більше не кешували їх.

2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.<br>Перший запит до resolver-a триває довше (повторні спроби до непрацюючого сервера), другий — швидше, оскільки сервер уже позначений як недоступний (fail fast). Якщо час однаковий — резолвер щоразу проходить таймаут.

3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?<br>Після named-start резолвер тримає сервер в infra-кеші як недоступний і повторює запит із затримкою; крім того, сервер завантажує зону.

4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.<br>serve-stale шкодить під час планового переїзду (завдання 4): клієнти довго отримують стару адресу після закінчення TTL.

## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```text
; Zone cs-202-21.lab - starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 75
@     60 IN SOA ns.cs-202-21.lab. admin.cs-202-21.lab. (
            2026102612 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            90         ; minimum
        )
@     IN NS ns.cs-202-21.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www                 IN A     10.53.0.81
files               IN CNAME www
mail                IN A     10.53.0.80
@                   IN MX    40 mail
@                   IN TXT   "lab-token=3e10b08c"
test  25            IN A     10.53.0.81
new-nxodo           IN A     10.53.0.80
new2-kqjwe           IN A     10.53.0.80
```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:<br>Неочевидною виявилася поведінка механізму serve-stale під час першого запиту на прострочений запис при падінні авторитетного сервера: резолвер робить таймаут-опитування перед тим, як віддати кешовані дані. Це видно за часом виконання real 0m1.824s у завдання 7.<br>Під час виконання завдання 4 пункту 7 while true; do echo "$(date +%T) $(dig +short www.sNN.lab)"; sleep
2; done не повертав NOERROR хоча я чекав по 5 хв.

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:<br>Усі питання лабораторної роботи та поведінка ієрархії DNS повністю з'ясовані за допомогою документації BIND, RFC 2308 та RFC 8767, а також експериментально підтверджені логами.

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | DeepSeek V4.1 Flash Free | Поясни покроково з командами що куди вставлять + методичка + md + силка на форк | багато чого. 167 264 tokens використано |
| 2 | DeepSeek V4.1 Flash Free | Перевір правильність виконання | змінив якщо щось не те було |
| 3 | Gemini 3.5 Flash-Light | перевір та заповни другу таблицю  | фінальна перевірка бо діпсік закінчився, заповнення другої таблиці, висновок |
