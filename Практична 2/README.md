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
Host: exemple.net
Connectoin: close
```

**Відповідь:**

```
HTTP/1.1 200 OK
Date: Mon, 05 Oct 2026 18:00:37 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: close
Server: cloudflare
Last-Modified: Fri, 02 Oct 2026 16:11:02 GMT
Allow: GET, HEAD
Accept-Ranges: bytes
Age: 4652
Cache-Control: max-age=14400
cf-cache-status: HIT
CF-RAY: a45e53aab914b607-WAW
alt-svc: h3=":443"; ma=86400

241
<!doctype html><html lang=en><head><meta charset=utf-8><link rel=icon href=data:,><meta name=viewport content="width=device-width,initial-scale=1"><title>Example Domain</title><style>html{color-scheme:light dark;background:light-dark(#eee,#222)}body{font:16px/1.6 system-ui,sans-serif;max-width:26em;margin:auto;padding:25vh 2em 2em;text-align:center}</style></head><body><p>This domain is for use in documentation examples without needing permission. This is not a service; avoid relying on it for testing and monitoring purposes.</p><script src=/s.js></script></body></html>
```

---

### Завдання A.2. Запит без поля `Host` у версії 1.1

**Команда:**

```
printf 'GET / HTTP/1.1\r\nConnection: close\r\n\r\n' | nc exemple.net 80
```

**Вивід:**

```
HTTP/1.1 400 Bad Request
Server: cloudflare
Date: Mon, 05 Oct 2026 18:02:08 GMT
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
printf 'GET / HTTP/1.1\r\nHost: tnpu.edu.ua\r\nConnection: close\r\n\r\n' | nc exemple.net 80
```

**Вивід:**

```
HTTP/1.1 409 Conflict
Date: Mon, 05 Oct 2026 18:04:56 GMT
Content-Type: text/plain; charset=UTF-8
Content-Length: 16
Connection: close
X-Frame-Options: SAMEORIGIN
Referrer-Policy: same-origin
Cache-Control: private, max-age=0, no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Server: cloudflare
CF-RAY: a45e5a02efe1b613-WAW

error code: 1001
```

#### A.3.2. Неіснуюче ім'я в полі `Host`

**Команда:**

```
printf 'GET / HTTP/1.1\r\nHost: opism-pr02.invalid\r\nConnection: close\r\n\r\n' | nc exemple.net 80
```

**Вивід:**

```
HTTP/1.1 409 Conflict
Date: Mon, 05 Oct 2026 18:06:38 GMT
Content-Type: text/plain; charset=UTF-8
Content-Length: 16
Connection: close
X-Frame-Options: SAMEORIGIN
Referrer-Policy: same-origin
Cache-Control: private, max-age=0, no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Server: cloudflare
CF-RAY: a45e5c826d1d5411-WAW

error code: 1001
```

#### A.3.3. Запит без поля `Host` у версії 1.0

**Команда:**

```
printf 'GET / HTTP/1.0\r\n\r\n' | nc exemple.net 80
```

**Вивід:**

```
HTTP/1.1 403 Forbidden
Date: Mon, 05 Oct 2026 18:08:17 GMT
Content-Type: text/plain; charset=UTF-8
Content-Length: 17
Connection: close
Cache-Control: private, max-age=0, no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Expires: Thu, 01 Jan 1970 00:00:01 GMT
Referrer-Policy: same-origin
X-Frame-Options: SAMEORIGIN
Server: cloudflare
CF-RAY: a45e5eed9c6160c9-WAW

error code: 1003
```

Зведення результатів наведено в **Додатку Д**.

---

### Завдання A.4. Два запити в одному з'єднанні

**Команда:**

```
printf 'GET /opism-pr02-12345 HTTP/1.1\r\nHost: exemple.net\r\n\r\nGET / HTTP/1.1\r\nHost: exemple.net\r\nConnection: close\r\n\r\n' | nc -C exemple.net 80
```

**Вивід:**

```
\nConnection: close\r\n\r\n' | nc -C exemple.net 80
HTTP/1.1 301 Moved Permanently
Date: Mon, 05 Oct 2026 18:10:04 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Server-Timing: cfEdge;dur=29,cfOrigin;dur=0
Location: https://exemple.net/opism-pr02-12345
Report-To: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=9hzk%2FjusMq8QcW83HPCS5oHEzgPQJJ9vLOnyU1RdwYdaRtkLn0YcPtQFc5x3DUknyY8%2FufVt%2BqtxMNxDyA6bD%2F8tKx%2BB6M0QO6%2BOBg%2FgUQZ5TLeCIY3f6fn04bYeWQ%3D%3D"}]}
Nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
Server: cloudflare
CF-RAY: a45e61895cecb613-WAW
alt-svc: h3=":443"; ma=86400

216
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>cloudflare</center>
<script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/v31edd6df95cf4e85bb4c19e7a9bdbcba1788362987495" integrity="sha512-iIg7k2xntmwu6/uSb5tpc/hySgZc4eoL31yB29W6tJFo2akwjPWcEqnCEdJvGexCL0KEQwVYv5BlowfhVz26hg==" data-cf-beacon='{"version":"2024.11.0","token":"9ddf7088d3594dabac233b5d35ef57c8","r":1,"spa":2}' crossorigin="anonymous"></script>
</body>
</html>

0

HTTP/1.1 301 Moved Permanently
Date: Mon, 05 Oct 2026 18:10:04 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: close
Server-Timing: cfEdge;dur=3,cfOrigin;dur=0
Location: https://exemple.net/
Report-To: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=kpHg7IP4S%2B4FS3%2B5%2BtaFKEt2zMfOzL0HWyHL%2BLYL%2FiNRKx7%2FaxBbkc7WZ0N2wZGQy9vAWffbS%2F8LaYJxFFhoXbVjz7ta6mBi%2BxU0%2B3BaI0wZ121ySSMPeO%2BILCopng%3D%3D"}]}
Nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
Server: cloudflare
CF-RAY: a45e618a0d0cb613-WAW
alt-svc: h3=":443"; ma=86400

216
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>cloudflare</center>
<script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/v31edd6df95cf4e85bb4c19e7a9bdbcba1788362987495" integrity="sha512-iIg7k2xntmwu6/uSb5tpc/hySgZc4eoL31yB29W6tJFo2akwjPWcEqnCEdJvGexCL0KEQwVYv5BlowfhVz26hg==" data-cf-beacon='{"version":"2024.11.0","token":"9ddf7088d3594dabac233b5d35ef57c8","r":1,"spa":2}' crossorigin="anonymous"></script>
</body>
</html>

0
```

**Кількість отриманих відповідей:**
2

**Коди стану отриманих відповідей:**
301 Moved Permanently для обох
---

### Завдання A.5. Запит за допомогою клієнтської програми

**Команда:**

```
curl -v --http1.1 http://exemple.net/ -o /dev/null
```

**Вивід:**

```
*   Trying 172.67.177.213:80...
* Host exemple.net:80 was resolved.
* IPv6: 2606:4700:3035::6815:5bbb, 2606:4700:3033::ac43:b1d5
* IPv4: 172.67.177.213, 104.21.91.187
* Established connection to exemple.net (172.67.177.213 port 80) from 192.168.188.129 port 44574
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0* using HTTP/1.x
> GET / HTTP/1.1
> Host: exemple.net
> User-Agent: curl/8.21.0
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 301 Moved Permanently
< Date: Mon, 05 Oct 2026 18:13:37 GMT
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Location: https://exemple.net/
< Report-To: {"group":"cf-nel","max_age":604800,"endpoints":[{"url":"https://a.nel.cloudflare.com/report/v4?s=sG6Wt8V9Kvqwr3NXdYq6J3%2F8xU90Tw99FvXKi6qdMaxFyI%2By5lFbJJns0uw2AvkFjhPWJggQgREZNpk3MlFwHtldmtdtxjdB6cXGBbFqUOoJOs3RZ7so%2FQAyPUm9hA%3D%3D"}]}
< Nel: {"report_to":"cf-nel","success_fraction":0.0,"max_age":604800}
< Server: cloudflare
< CF-RAY: a45e66be098160c9-WAW
< alt-svc: h3=":443"; ma=86400
<
{ [178 bytes data]
100    167   0    167   0      0   1019      0                              0
* Connection #0 to host exemple.net:80 left intact
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
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |
| 6 | | | | | |
| 7 | | | | | |
| 8 | | | | | |

> Рядок наводять на кожне поле, яке реально надійшло. Зайві рядки вилучають, за потреби додають нові. Поле, походження якого встановити не вдалося, зазначають із позначкою «не визначено» та поясненням утруднення.

---

## Частина D. Висновки

Обсяг — 150–300 слів. Висновки спираються на власні спостереження.

**D.1.** Що з поведінки сервера виявилося неочевидним або несподіваним. Конкретно, з посиланням на рядок виводу.

<текст>

**D.2.** Яке з полів заголовка викликало найбільше утруднення при визначенні походження (частина B) та з якої причини.

<текст>

**D.3.** Яке питання залишилося без відповіді після виконання роботи.

<текст>

---

## Контрольні питання

**1.** У завданні A.1 сервер не надсилав відповіді, доки не було введено порожній рядок. Чим це зумовлено?

<відповідь>

**2.** Порівняйте результати завдань A.1, A.2 та A.3.1–A.3.3 (таблиця Додатка Д). За яких значень поля `Host` і за якої версії протоколу сервер обслуговує запит, а за яких — ні? Яку задачу розв'язує поле `Host`? Відповідь має посилатися на конкретні рядки ваших виводів.

<відповідь>

**3.** Скільки відповідей надійшло у завданні A.4 і з якими кодами стану? Чи залежить відповідь сервера на порту 80 від запитаного шляху — і що це говорить про роль цього сервера? Якщо надійшла одна відповідь, знайдіть у ній поле заголовка, яке це пояснює, або зазначте, що такого поля немає. Якщо надійшло дві — що це означає для клієнтської програми, яка завантажує сторінку з великою кількістю вкладених ресурсів?

<відповідь>

**4.** Які поля заголовка програма `curl` додала самостійно (завдання A.5)? Ці поля не є обов'язковими — сервер відповів і без них у завданні A.1. З якою метою їх додано?

<відповідь>

**5.** За якими ознаками у вашому виводі виявляється присутність проміжного вузла? Якщо таких ознак не виявлено, поясніть, що з цього випливає.

<відповідь>

**6.** Три рядки, про які не йшлося на лекції, наведено в **Додатку В**.

---

## Додаток В. Відповіді на питання 6

| № | Рядок виводу | Джерело (номер завдання) |
|---|---|---|
| 1 | | |
| 2 | | |
| 3 | | |

---

## Додаток Д. Зведення результатів завдання A.3

**Вузол, з яким установлювалося з'єднання (у всіх пробах однаковий):**

| Проба | Значення поля `Host` | Версія | Код стану | Обсяг тіла відповіді | Збігається з A.1 (так / ні) |
|---|---|---|---|---|---|
| A.1 (вихідна) | | 1.1 | | | — |
| A.2 | поле відсутнє | 1.1 | | | |
| A.3.1 | | 1.1 | | | |
| A.3.2 | `opism-pr02.invalid` | 1.1 | | | |
| A.3.3 | поле відсутнє | 1.0 | | | |

**Висновок за таблицею (2–4 речення):** що саме змінювалося у запиті від проби до проби і як на це реагував сервер.

<текст>

---

## Декларування використання технологій штучного інтелекту

Для цієї роботи встановлено **рівень Р3 — ШІ як співвиконавець**.

Виводи команд частини A не можуть бути згенеровані та мають бути отримані внаслідок фактичного виконання команд.

**Чи використовувалися технології ШІ під час виконання роботи:** <так / ні>

Якщо так, заповнюють таблицю. Якщо ні, таблицю вилучають.

| № | Інструмент (назва, версія) | Етап роботи | Дослівний текст запиту (промпту) | Як використано результат |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |

**Підтвердження:** усі виводи команд, наведені в частині A, отримано внаслідок фактичного виконання команд на зазначеному індивідуальному домені.

---

*ОПІСМ (ОК-13) · Практична робота № 2 · бланк звіту*
