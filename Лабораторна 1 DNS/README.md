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
| Псевдонім (CNAME на `www`) | |
| Другий хост і його TTL, с | |
| Пріоритет MX | |
| Токен TXT | |
| Ім'я для завдання 5 | |

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

```
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

```
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

```
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
```
 user:pass
```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:

<br>TCP (DNS уже спрацював — ім'я розірвалося в 10.53.0.80; TLS не починався, тому що з'єднання не встановилося).

```
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

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок 70, найбільша можлива затримка: 75

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```text
root@ns-cs-202-21:/# dig @10.53.1.1 www.cs-202-21.lab +norec +short
10.53.0.80
```

Залишок 75 і час (крок 4), час 13:13:39 (крок 5), перевірка на сервері (крок 6):

```text
root@ns-cs-202-21:/# dig @10.53.1.1 www.cs-202-21.lab +norec +short
10.53.0.80
```

Очікуваний час появи нової адреси (підставити R у формулу):

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```text

```

Рядок `server:` з відповіді `curl`:

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| | | | | |

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?

2. Якою була б найбільша можлива затримка для цього запису і за яких умов?

3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?

## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення:

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```text

```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):

```text

```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):

```text

```

| Негативну відповідь закешовано | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| | | | | |

<!-- Якщо завдання повторювалося з іншим ім'ям (new2-…), зазначити це й причину. -->

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?

2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.

3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити?

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні | | |
| B: нове ім'я в зоні `lab.` | | |
| C: ім'я в неіснуючому TLD | | |
| D: `www.sNN.lab` після перезапуску резолвера | | |

Вивід вимірювань:

```text

```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A | | | |
| B | | | |
| C | | | |
| D | | | |

Значення `time_namelookup` з `curl` після перезапуску резолвера:

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```text

```

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.

2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?

3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?

## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```text

```

Запити із заміром часу (крок 5):

```text

```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | | | |
| resolver-a, другий | | | |
| resolver-b | | | |

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```text

```

Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):

```text

```

### Постмортем

1. Що сталося:
2. Вплив на клієнтів resolver-a і resolver-b:
3. Хронологія з виміряним часом:
4. Чому кеш допоміг або не допоміг:
5. Що слід змінити:

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?

2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.

3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?

4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.

## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```text

```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено (RFC, документація, додатковий експеримент на стенді) і що з'ясувати не вдалося:

## Декларація використання ШІ

Використовувалися інструменти ШІ: <!-- так / ні -->

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | | | |
