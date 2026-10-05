# Практична робота № 2

**Дисципліна:** Основи побудови інформаційних систем та мереж (ОК-13)

**Тема:** Структура повідомлень прикладного протоколу HTTP. Формування запиту вручну

| Поле | Значення |
|---|---|
| Студент (прізвище, ім'я, по батькові) | Приходько Максим Сергійович |
| Група | F5 2.02 |
| Номер варіанта | 22 |
| Індивідуальний домен | example.net |
| «Чужий» домен для завдання A.3.1 (варіант ± 20) | tnpu.edu.ua (-20) |
| Середовище виконання | linux |
| Дата виконання | 05.10.2026 |

> Бланк заповнюють, не змінюючи структури розділів. Порожні заготовки блоків коду замінюють власними виводами. Позначки-підказки в кутових дужках вилучають.

---

## Частина A. Збір експериментальних даних

### Завдання A.1. Формування запиту вручну

**Команда:**

```
nc -C example.net 80
```

**Набраний запит:**

```
GET / HTTP/1.1
Host: example.net
Connection: close
```

**Відповідь:**

```
HTTP/1.1 200 OK
Date: Mon, 05 Oct 2026 19:05:58 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: close
Server: cloudflare
Last-Modified: Fri, 02 Oct 2026 16:11:02 GMT
Allow: GET, HEAD
Accept-Ranges: bytes
Age: 8573
Cache-Control: max-age=14400
cf-cache-status: HIT
CF-RAY: a45eb363cd73b607-WAW
alt-svc: h3=":443"; ma=86400

241
<!doctype html><html lang=en><head><meta charset=utf-8><link rel=icon href=data:,><meta name=viewport content="width=device-width,initial-scale=1"><title>Example Domain</title><style>html{color-scheme:light dark;background:light-dark(#eee,#222)}body{font:16px/1.6 system-ui,sans-serif;max-width:26em;margin:auto;padding:25vh 2em 2em;text-align:center}</style></head><body><p>This domain is for use in documentation examples without needing permission. This is not a service; avoid relying on it for testing and monitoring purposes.</p><script src=/s.js></script></body></html>

0

```

---

### Завдання A.2. Запит без поля `Host` у версії 1.1

**Команда:**

```
printf 'GET / HTTP/1.1\r\nConnection: close\r\n\r\n' | nc example.net 80
```

**Вивід:**

```
HTTP/1.1 400 Bad Request
Server: cloudflare
Date: Mon, 05 Oct 2026 18:57:35 GMT
Content-Type: text/html
Content-Length: 155
Connection: close
CF-RAY: -

<html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>cloudflare</center>
</body>
</html>
```

---

### Завдання A.3. Вплив поля `Host` на відповідь сервера

#### A.3.1. Чуже доменне ім'я в полі `Host`

**Команда:**

```
printf 'GET / HTTP/1.1\r\nHost: tnpu.edu.ua\r\nConnection: close\r\n\r\n' | nc example.net 80
```

**Вивід:**

```
HTTP/1.1 409 Conflict
Date: Mon, 05 Oct 2026 18:57:57 GMT
Content-Type: text/plain; charset=UTF-8
Content-Length: 16
Connection: close
X-Frame-Options: SAMEORIGIN
Referrer-Policy: same-origin
Cache-Control: private, max-age=0, no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Server: cloudflare
CF-RAY: a45ea7abd90b9e2a-WAW

error code: 1001
```

#### A.3.2. Неіснуюче ім'я в полі `Host`

**Команда:**

```
printf 'GET / HTTP/1.1\r\nHost: opism-pr02.invalid\r\nConnection: close\r\n\r\n' | nc example.net 80
```

**Вивід:**

```
HTTP/1.1 409 Conflict
Date: Mon, 05 Oct 2026 18:58:21 GMT
Content-Type: text/plain; charset=UTF-8
Content-Length: 16
Connection: close
X-Frame-Options: SAMEORIGIN
Referrer-Policy: same-origin
Cache-Control: private, max-age=0, no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Server: cloudflare
CF-RAY: a45ea8447aedb607-WAW

error code: 1001
```

#### A.3.3. Запит без поля `Host` у версії 1.0

**Команда:**

```
printf 'GET / HTTP/1.0\r\n\r\n' | nc example.net 80
```

**Вивід:**

```
HTTP/1.1 403 Forbidden
Date: Mon, 05 Oct 2026 18:59:39 GMT
Content-Type: text/plain; charset=UTF-8
Content-Length: 17
Connection: close
Cache-Control: private, max-age=0, no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Referrer-Policy: same-origin
X-Frame-Options: SAMEORIGIN
Server: cloudflare
CF-RAY: a45eaa28fbe134f7-WAW

error code: 1003
```

Зведення результатів наведено в **Додатку Д**.

---

### Завдання A.4. Два запити в одному з'єднанні

**Команда:**

```
printf 'GET /opism-pr02-12345 HTTP/1.1\r\nHost: example.net\r\n\r\nGET / HTTP/1.1\r\nHost: example.net\r\nConnection: close\r\n\r\n' | nc -C example.net 80
```

**Вивід:**

```
HTTP/1.1 404 Not Found
Date: Mon, 05 Oct 2026 19:00:07 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: cloudflare
Age: 7391
Cache-Control: max-age=14400
cf-cache-status: HIT
CF-RAY: a45eaadbcb6a7311-WAW
alt-svc: h3=":443"; ma=86400

241
<!doctype html><html lang=en><head><meta charset=utf-8><link rel=icon href=data:,><meta name=viewport content="width=device-width,initial-scale=1"><title>Example Domain</title><style>html{color-scheme:light dark;background:light-dark(#eee,#222)}body{font:16px/1.6 system-ui,sans-serif;max-width:26em;margin:auto;padding:25vh 2em 2em;text-align:center}</style></head><body><p>This domain is for use in documentation examples without needing permission. This is not a service; avoid relying on it for testing and monitoring purposes.</p><script src=/s.js></script></body></html>

0

HTTP/1.1 200 OK
Date: Mon, 05 Oct 2026 19:00:07 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: close
Server: cloudflare
Last-Modified: Fri, 02 Oct 2026 16:11:02 GMT
Allow: GET, HEAD
Accept-Ranges: bytes
Age: 8222
Cache-Control: max-age=14400
cf-cache-status: HIT
CF-RAY: a45eaadd3bcc7311-WAW
alt-svc: h3=":443"; ma=86400

241
<!doctype html><html lang=en><head><meta charset=utf-8><link rel=icon href=data:,><meta name=viewport content="width=device-width,initial-scale=1"><title>Example Domain</title><style>html{color-scheme:light dark;background:light-dark(#eee,#222)}body{font:16px/1.6 system-ui,sans-serif;max-width:26em;margin:auto;padding:25vh 2em 2em;text-align:center}</style></head><body><p>This domain is for use in documentation examples without needing permission. This is not a service; avoid relying on it for testing and monitoring purposes.</p><script src=/s.js></script></body></html>

0
```

**Кількість отриманих відповідей:**
2

**Коди стану отриманих відповідей:**
HTTP/1.1 404 Not Found & HTTP/1.1 200 OK
---

### Завдання A.5. Запит за допомогою клієнтської програми

**Команда:**

```
curl -v --http1.1 http://example.net/ -o /dev/null
```

**Вивід:**

```
*   Trying [2606:4700:10::6814:1508]:80...
* Immediate connect fail for 2606:4700:10::6814:1508: Network is unreachable
* connect to 2606:4700:10::6814:1508 port 80 from :: port 0 failed: Network is unreachable
*   Trying [2606:4700:10::ac42:af3b]:80...
* Immediate connect fail for 2606:4700:10::ac42:af3b: Network is unreachable
* connect to 2606:4700:10::ac42:af3b port 80 from :: port 0 failed: Network is unreachable
* Host example.net:80 was resolved.
* IPv6: 2606:4700:10::6814:1508, 2606:4700:10::ac42:af3b
* IPv4: 104.20.21.8, 172.66.175.59
*   Trying 104.20.21.8:80...
* Established connection to example.net (104.20.21.8 port 80) from 192.168.188.129 port 46176
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0* using HTTP/1.x
> GET / HTTP/1.1
> Host: example.net
> User-Agent: curl/8.21.0
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Mon, 05 Oct 2026 19:01:03 GMT
< Content-Type: text/html; charset=utf-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Server: cloudflare
< Last-Modified: Fri, 02 Oct 2026 16:11:02 GMT
< Allow: GET, HEAD
< Accept-Ranges: bytes
< Age: 8278
< Cache-Control: max-age=14400
< cf-cache-status: HIT
< CF-RAY: a45eac3ac8e0c051-WAW
< alt-svc: h3=":443"; ma=86400
<
{ [589 bytes data]
100    577   0    577   0      0   2811      0                              0
* Connection #0 to host example.net:80 left intact
```

---

### Завдання A.6. Запит через захищене з'єднання

**Ресурс, на якому виконано завдання:** <власний домен / `iana.org`>

**Підстава для використання резервного ресурсу (заповнюють за потреби):**

**Команда:**

```
<текст команди>
```

**Набраний запит:**

```
<текст запиту>
```

**Вивід:**

```
<повний вивід>
```

---

## Частина B. Розбір полів заголовка

Розбирається відповідь, отримана в завданні A.1.

**Загальна кількість полів заголовка у відповіді:**

| № | Поле заголовка | Значення | Призначення (власне формулювання) | Походження: сервер / проміжний вузол / не визначено | Обґрунтування |
|---|---|---|---|---|---|
| 1 | HTTP/1.1 | 200 OK | Статус відповіді | сервер | `Генерується вебсервером для підтвердження обробки запиту` |
| 2 | Date | Mon, 05 Oct 2026 19:05:58 GMT | Час створення запиту | сервер | `Заголовок створений сервером для фіксації часу` |
| 3 | Content-Type | text/html; charset=utf-8 | Визначаж тип данних | сервер | `Вказується сервером залежно від формату ресурсу `|
| 4 | Transfer-Encoding | chunked | Застосовує передачу даних частинами | cервер | `Застосовує передачу даних частинами` |
| 5 | Connection | close | Стан мережевого з'єднання | сервер | `Вказується сервером для керування життєциклом сокета` |
| 6 | Server | cloudflare | Сервіс, що обробив запит | проміжний вузол | `Вказує на те, що запит перехоплено мережею доставки контенту` |
| 7 | Last-Modified | Fri, 02 Oct 2026 16:11:02 GMT | Дата та час останньої модифікації цільового ресурсу на серверах | сервер | `Встановлюється системою або розробником для відстеження актуальності файлу` |
| 8 | Allow | GET, HEAD | Перелік HTTP-методів, які підтримує та на які реагує цей ресурс | сервер | `Визначається конфігурацією вебсервера для конкретного маршруту` |
| 9 | Accept-Ranges | bytes | Повідомляє про підтримку сервергом запитів на передачу частини файлу | сервер | `Повідомляє про підтримку сервергом запитів на передачу частини файлу` |
| 10 | Age | 4652 | Час у секундах, протягом якого об'єкт знаходився в кеші проміжного проксі | проміжний вузол | `Розраховується проксі-сервером (Cloudflare) з моменту кешування ресурсу` |
| 11 | Cache-Control | max-age=14400 | Керує правилами кешування, задаючи максимальний час актуальності документа | сервер | `Задається політикою кешування вебсервера або розробника сайту` |
| 12 | cf-cache-status | HIT | Показує статус кешування проксі | проміжний вузол |  `Специфічний заголовок інфраструктури Cloudflare` |
| 13 | CF-RAY | a45e53aab914b607-WAW | Унікальний ідентифікатор транзакції та запиту для відстеження | проміжний вузол | `Генерується автоматично вузлами мережі Cloudflare` |
| 14 | alt-svc | h3=":443"; ma=86400 | Повідомляє про доступність альтернативних протоколів підключення | проміжний вузол | `Додається сервером або CDN для переходу на швидші протоколи` |

> Рядок наводять на кожне поле, яке реально надійшло. Зайві рядки вилучають, за потреби додають нові. Поле, походження якого встановити не вдалося, зазначають із позначкою «не визначено» та поясненням утруднення.

---

## Частина D. Висновки

Обсяг — 150–300 слів. Висновки спираються на власні спостереження.

**D.1.** Що з поведінки сервера виявилося неочевидним або несподіваним. Конкретно, з посиланням на рядок виводу.

Найбільш несподіваним виявився результат у завданні A.4 під час передачі двох запитів в одному TCP-з'єднанні: на перший запит до неіснуючого ресурсу `GET /opism-pr02-12345 HTTP/1.1` сервер повернув статус `HTTP/1.1 404 Not Found` (причому віддав його з кешу, про що свідчать рядки `cf-cache-status: HIT` та `Age: 7391`), але передав тіло відповіді розміром 241 байт (у шістнадцятковій системі) зі стандартною HTML-сторінкою <title>Example Domain</title>, яке є абсолютно ідентичним тілу успішної відповіді `HTTP/1.1 200 OK` на другий запит `GET / HTTP/1.1`. Також неочевидною була реакція сервера в завданні A.3.3 на запит `GET / HTTP/1.0` без заголовка Host: хоча специфікація `HTTP/1.0 `не вимагає наявності цього поля, вузол Cloudflare відхилив запит зі статусом `HTTP/1.1 403 Forbidden` та тілом `error code: 1003`. Крім того, у завданні A.5 клієнт curl спочатку двічі безуспішно намагався підключитися за IPv6-адресами (Immediate connect fail for 2606:4700:10::6814:1508: Network is unreachable), перш ніж автоматично переключитися на IPv4 (Trying 104.20.21.8:80...).

**D.2.** Яке з полів заголовка викликало найбільше утруднення при визначенні походження (частина B) та з якої причини.

Найбільше утруднення у частині B викликало визначення точного походження стандартних полів `Date`, `Transfer-Encoding: chunked`, `Connection: close` та `Cache-Control: max-age=14400`. Наявність у відповіді полів Server: `cloudflare`, `cf-cache-status: HIT`та A`ge: 8573 `прямо вказує на те, що відповідь була сформована та віддана кешуючим проміжним вузлом CDN Cloudflare, без прямого звернення до первинного вебсервера у момент запиту. Через роботу реверс-проксі неможливо без доступу до конфігурації основного сервера однозначно встановити, які з цих заголовків надійшли від первинного сервера під час кешування, а які були згенеровані чи модифіковані вузлом Cloudflare при формуванні відповіді клієнту.

**D.3.** Яке питання залишилося без відповіді після виконання роботи.

Після виконання роботи відкритим залишилося питання щодо механізму кешування помилок мережею Cloudflare у завданні A.4: чому на запит до довільного неіснуючого шляху `/opism-pr02-12345` проксі-сервер повертає статус `cf-cache-status: HIT` зі значним часом перебування в кеші (Age: 7391), проте без заголовків `Last-Modified `та `Allow`, які присутні у відповіді 200 OK? Чи означає це, що CDN використовує єдиний закешований запис сторінки-заглушки 404 Not Found для будь-яких відсутніх URI на домені `example.net`?

---

## Контрольні питання

**1.** У завданні A.1 сервер не надсилав відповіді, доки не було введено порожній рядок. Чим це зумовлено?

Порожній рядок свідчить про завершення тіла запиту

**2.** Порівняйте результати завдань A.1, A.2 та A.3.1–A.3.3 (таблиця Додатка Д). За яких значень поля `Host` і за якої версії протоколу сервер обслуговує запит, а за яких — ні? Яку задачу розв'язує поле `Host`? Відповідь має посилатися на конкретні рядки ваших виводів.

Коли сервер обслуговує запит: Лише у завданні A.1 за протоколом HTTP/1.1 із коректним полем Host: example.net. Сервер повертає HTTP/1.1 200 OK та цільову HTML-сторінку (<title>Example Domain</title>).

Коли сервер НЕ обслуговує запит:

    `Без поля Host у HTTP/1.1 (A.2): повертає HTTP/1.1 400 Bad Request (<h1>400 Bad Request</h1>), бо в версії 1.1 цей заголовок є синтаксично обов'язковим.`

    `З чужим або неіснуючим іменем у Host (A.3.1 — tnpu.edu.ua, A.3.2 — opism-pr02.invalid): повертає HTTP/1.1 409 Conflict (error code: 1001), бо вузол Cloudflare за цією IP-адресою не обслуговує вказані домени.`

   `Без поля Host у HTTP/1.0 (A.3.3): повертає HTTP/1.1 403 Forbidden (error code: 1003), оскільки проксі Cloudflare блокує звернення напряму за IP-адресою без вказання цільового домену.`

Host забезпечує віртуальний хостинг та маршрутизацію на проксі-сервері.

**3.** Скільки відповідей надійшло у завданні A.4 і з якими кодами стану? Чи залежить відповідь сервера на порту 80 від запитаного шляху — і що це говорить про роль цього сервера? Якщо надійшла одна відповідь, знайдіть у ній поле заголовка, яке це пояснює, або зазначте, що такого поля немає. Якщо надійшло дві — що це означає для клієнтської програми, яка завантажує сторінку з великою кількістю вкладених ресурсів?

Дві відповіді. Від запитаного шляху залежить лише код стану та набір заголовків. Отримання двох відповідей у межах одного TCP-сеансу демонструє роботу постійних з'єднань та конвеєрної обробки. Для браузера, який завантажує сторінку з багатьма вкладеними ресурсами, це означає суттєве прискорення роботи: клієнту не потрібно витрачати час і мережеві ресурси на встановлення нового TCP-з'єднання для кожного окремого файлу — усі їх можна завантажити послідовно через одне вже відкрите з'єднання.

**4.** Які поля заголовка програма `curl` додала самостійно (завдання A.5)? Ці поля не є обов'язковими — сервер відповів і без них у завданні A.1. З якою метою їх додано?

`User-Agent: curl/8.21.0` — ідентифікує клієнтську програму та її версію.
`Accept: */*` — реалізує механізм узгодження вмісту

**5.** За якими ознаками у вашому виводі виявляється присутність проміжного вузла? Якщо таких ознак не виявлено, поясніть, що з цього випливає.

`Server: cloudflare` — вказує, що TCP-з'єднання та HTTP-запит обробив не первинний вебсервер, а граничний сервер мережі `Cloudflare`.

**6.** Три рядки, про які не йшлося на лекції, наведено в **Додатку В**.

---

## Додаток В. Відповіді на питання 6

| № | Рядок виводу | Джерело (номер завдання) |
|---|---|---|
| 1 | `CF-RAY: a45eb363cd73b607-WAW` | A.1 |
| 2 | `cf-cache-status: HIT` | A.1 |
| 3 | `alt-svc: h3=":443"; ma=86400` | A.1 |

---

## Додаток Д. Зведення результатів завдання A.3

**Вузол, з яким установлювалося з'єднання (у всіх пробах однаковий):**

| Проба | Значення поля `Host` | Версія | Код стану | Обсяг тіла відповіді | Збігається з A.1 (так / ні) |
|---|---|---|---|---|---|
| A.1 (вихідна) | | 1.1 | `200 Ok` | 577 байт | — |
| A.2 | поле відсутнє | 1.1 | `400 Bad Request` | 155 байт | ні |
| A.3.1 | | 1.1 | `409 Conflict` | 16 байт | ні |
| A.3.2 | `opism-pr02.invalid` | 1.1 | `409 Conflict` | 16 байт | ні |
| A.3.3 | поле відсутнє | 1.0 | `403 Forbidden` | 17 байт | ні |

**Висновок за таблицею (2–4 речення):** що саме змінювалося у запиті від проби до проби і як на це реагував сервер.

У пробах змінювалися наявність та значення поля `Host`, а також версія протоколу HTTP (1.1 та 1.0) при незмінному TCP-з'єднанні з тим самим вузлом `example.net`. Сервер успішно повернув цільову сторінку (`200 OK`, 577 байт) лише у вихідній пробі A.1, де вказано коректне доменне ім'я `Host: example.net`. За відсутності поля `Host` у версії `1.1` сервер відхилив запит через порушення синтаксису стандарту `400 Bad Request`, а у версії `1.0` — заблокував пряме звернення без імені хоста `403 Forbidden`, `error code: 1003`. При підстановці чужого або неіснуючого домену проксі-вузол `Cloudflare` прийняв запит синтаксично, але повернув помилку маршрутизації `409 Conflict`, `error code: 1001`, оскільки не зміг зіставити вказані імена з жодним віртуальним хостом на своїй IP-адресі.

---

## Декларування використання технологій штучного інтелекту

Для цієї роботи встановлено **рівень Р3 — ШІ як співвиконавець**.

Виводи команд частини A не можуть бути згенеровані та мають бути отримані внаслідок фактичного виконання команд.

**Чи використовувалися технології ШІ під час виконання роботи:** так

Якщо так, заповнюють таблицю. Якщо ні, таблицю вилучають.

| № | Інструмент (назва, версія) | Етап роботи | Дослівний текст запиту (промпту) | Як використано результат |
|---|---|---|---|---|
| 1 | Gemini 3.5 Flash-Lite | B | вивід та фото таблиці. промт: `Шо тут заголовки` | заповнено таблицю |
| 2 | Gemini 3.1 PRO | Виснокок, Контрольні питання | заповни за роботою | відредашовано та заповнено |

**Підтвердження:** усі виводи команд, наведені в частині A, отримано внаслідок фактичного виконання команд на зазначеному індивідуальному домені.

---

*ОПІСМ (ОК-13) · Практична робота № 2 · бланк звіту*
