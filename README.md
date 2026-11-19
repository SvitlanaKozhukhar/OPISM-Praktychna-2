# Практична робота № 2

**Дисципліна:** Основи побудови інформаційних систем та мереж (ОК-13)

**Тема:** Структура повідомлень прикладного протоколу HTTP. Формування запиту вручну

| Поле | Значення |
|---|---|
| Студент (прізвище, ім'я, по батькові) |Кожухар Світлана Василівна|
| Група |21|
| Номер варіанта |6|
| Індивідуальний домен |kpi.ua|
| «Чужий» домен для завдання A.3.1 (варіант ± 20) |openssl.org|
| Середовище виконання |Killercoda Ubuntu Playground|
| Дата виконання |19.11.2026|

> Бланк заповнюють, не змінюючи структури розділів. Порожні заготовки блоків коду замінюють власними виводами. Позначки-підказки в кутових дужках вилучають.

---

## Частина A. Збір експериментальних даних

### Завдання A.1. Формування запиту вручну

**Команда:**

```
nc -C kpi.ua 80
```

**Набраний запит:**

```
GET / HTTP/1.1
Host: kpi.ua
Connection: close
```

**Відповідь:**

```
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Thu, 19 Nov 2026 12:16:10 GMT
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://kpi.ua/
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

---

### Завдання A.2. Запит без поля `Host` у версії 1.1

**Команда:**

```
printf 'GET / HTTP/1.1\r\nConnection: close\r\n\r\n' | nc kpi.ua 80
```

**Вивід:**

```
HTTP/1.1 400 Bad Request
Server: nginx
Date: Thu, 19 Nov 2026 12:19:44 GMT
Content-Type: text/html
Content-Length: 150
Connection: close

<html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

---

### Завдання A.3. Вплив поля `Host` на відповідь сервера

#### A.3.1. Чуже доменне ім'я в полі `Host`

**Команда:**

```
printf 'GET / HTTP/1.1\r\nHost: openssl.org\r\nConnection: close\r\n\r\n' | nc kpi.ua 80
```

**Вивід:**

```
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Thu, 19 Nov 2026 12:22:27 GMT
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://openssl.org/
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

#### A.3.2. Неіснуюче ім'я в полі `Host`

**Команда:**

```
printf 'GET / HTTP/1.1\r\nHost: opism-pr02.invalid\r\nConnection: close\r\n\r\n' | nc kpi.ua 80
```

**Вивід:**

```
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Thu, 19 Nov 2026 12:25:00 GMT
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://opism-pr02.invalid/
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

#### A.3.3. Запит без поля `Host` у версії 1.0

**Команда:**

```
printf 'GET / HTTP/1.0\r\n\r\n' | nc kpi.ua 80
```

**Вивід:**

```
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Thu, 19 Nov 2026 12:27:38 GMT
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://_/
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

Зведення результатів наведено в **Додатку Д**.

---

### Завдання A.4. Два запити в одному з'єднанні

**Команда:**

```
printf 'GET /opism-pr02-12345 HTTP/1.1\r\nHost: kpi.ua\r\n\r\nGET / HTTP/1.1\r\nHost: kpi.ua\r\nConnection: close\r\n\r\n' | nc -C kpi.ua 80
```

**Вивід:**

```
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Thu, 19 Nov 2026 12:31:54 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
Location: https://kpi.ua/opism-pr02-12345
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Thu, 19 Nov 2026 12:31:54 GMT
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://kpi.ua/
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

**Кількість отриманих відповідей:** 2

**Коди стану отриманих відповідей:** 301,301

---

### Завдання A.5. Запит за допомогою клієнтської програми

**Команда:**

```
curl -v --http1.1 http://kpi.ua/ -o /dev/null
```

**Вивід:**

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Host kpi.ua:80 was resolved.
* IPv6: (none)
* IPv4: 80.91.164.19
*   Trying 80.91.164.19:80...
* Connected to kpi.ua (80.91.164.19) port 80
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0> GET / HTTP/1.1
> Host: kpi.ua
> User-Agent: curl/8.5.0
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< Server: nginx
< Date: Thu, 19 Nov 2026 12:34:48 GMT
< Content-Type: text/html
< Content-Length: 162
< Connection: keep-alive
< Location: https://kpi.ua/
< Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
<
{ [162 bytes data]
100   162  100   162    0     0   1760      0 --:--:-- --:--:-- --:--:--  1760
* Connection #0 to host kpi.ua left intact
```

---

### Завдання A.6. Запит через захищене з'єднання

**Ресурс, на якому виконано завдання:** kpi.ua

**Підстава для використання резервного ресурсу (заповнюють за потреби):** Резервний ресурс не використовувався.

**Команда:**

```
printf 'GET / HTTP/1.1\r\nHost: kpi.ua\r\nConnection: close\r\n\r\n' | openssl s_client -connect kpi.ua:443 -servername kpi.ua -quiet > /root/a6.txt 2>&1
```

**Набраний запит:**

```
GET / HTTP/1.1
Host: kpi.ua
Connection: close
```

**Вивід:**

```
depth=3 C = US, O = Internet Security Research Group, CN = ISRG Root X2
verify return:1
depth=2 C = US, O = ISRG, CN = Root YE
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = YE2
verify return:1
depth=0 CN = kpi.ua
verify return:1
HTTP/1.1 200 OK
Server: nginx
Content-Type: text/html; charset=UTF-8
Content-Length: 96161
Connection: close
Cache-Control: max-age=180, public
Date: Thu, 19 Nov 2026 12:38:35 GMT
Content-language: UK
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Expires: Sun, 19 Nov 1978 05:00:00 GMT
Last-Modified: Thu, 19 Nov 2026 12:38:34 GMT
ETag: "1795091914"
Vary: Cookie
alt-svc: h3-27=":443"; ma=86400, h3-28=":443"; ma=86400, h3-29=":443"; ma=86400, h3=":443"; ma=86400

<!DOCTYPE html>
<html lang="UK" dir="ltr">
  <head>
    <meta charset="utf-8" />
<meta name="description" content="Київський політехнічний інститут імені Ігоря Сікорського працює, щоб створювати суспільство майбутнього на засадах концепції сталого розвитку." />
<meta name="abstract" content="Національний технічний університет України «Київський політехнічний інститут імені Ігоря Сікорського»" />
<meta name="keywords" content="КПІ, Київська політехніка, КПИ, КПІ Сікорського, kpi, rgs, універ, університет, ЗВО, інститут" />
<link rel="shortlink" href="https://kpi.ua/" />
<link rel="canonical" href="https://kpi.ua/" />
<meta name="Generator" content="Drupal 10 (https://www.drupal.org)" />
<meta name="MobileOptimized" content="width" />
<meta name="HandheldFriendly" content="true" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<link rel="icon" href="/kpi.ico" type="image/vnd.microsoft.icon" />

    <title>КПІ ім. Ігоря Сікорського</title>
    <link rel="stylesheet" media="all" href="/files/css/css_o35QIBjmSaoc0s58b1Be9kHzn6p7Ef7Q8wafeNx-hIc.css?delta=0&amp;language=UK&amp;theme=thex&amp;include=eJxtyEEOgCAMBMAPoX0SQajQpIBxQeT3Jnr1ModBP_jcBYmkSDOYaJxpc2DTEt8UtW5OF7SpUqK5hAfodc01dOWvLFQCI9Vh_fTK9LsPPlMrdg" />
<link rel="stylesheet" media="all" href="/files/css/css_SvNgHDYhCPdyycn6mM-Kt_3_dVuhk0hXQwI5Z6n96LI.css?delta=1&amp;language=UK&amp;theme=thex&amp;include=eJxtyEEOgCAMBMAPoX0SQajQpIBxQeT3Jnr1ModBP_jcBYmkSDOYaJxpc2DTEt8UtW5OF7SpUqK5hAfodc01dOWvLFQCI9Vh_fTK9LsPPlMrdg" />

    


<script async src="https://www.googletagmanager.com/gtag/js?id=G-JEVC3CSYBQ"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-JEVC3CSYBQ');
</script>

<script type='text/javascript' src='https://platform-api.sharethis.com/js/sharethis.js#property=616dde684564d200122a7ec1&product=sop' async='async'></script>

  </head>
  <body class="homepage user-guest one-sidebar sidebar-right">
        <a href="#main-content" class="visually-hidden focusable">
      Перейти до основного вмісту
    </a>
    
      <div class="dialog-off-canvas-main-canvas" data-off-canvas-main-canvas>
    <header class="header">
  <div class="container">
    <div class="header-container">
              <div class="site-brand">
            <div class="block-region region-header">
    <div id="block-thex-branding" class="block">
  
    
  <div class="block-content">
  <div class="site-branding">
      <div class="site-logo">
      <a href="/index%2ephp/" rel="home">
        <img src="/files/logo.png" alt="Головна" />
      </a>
    </div><!-- site-logo -->
        <div class="site-name-slogan">
              <div class="site_slogan">
          <a href="/" title="Головна" rel="home">Національний технічний університет України "Київський політехнічний інститут імені Ігоря Сікорського"</a>
        </div>
            
    </div><!-- site-name-slogan -->
  </div><!-- site-branding -->
  </div><!--/.block-content -->
</div><!--/.block -->

  </div>

        </div> <!--/.site-branding -->
                  <div class="header-right">
                  <div class="mobile-menu">
	<span></span>
	<span></span>
	<span></span>
</div>
<div class="primary-menu-wrapper">
	<div class="menu-wrap">
		<div class="close-mobile-menu"><i class="icon-close"></i></div>
		  <div class="block-region region-primary-menu">
    <div id="block-thex-ikonkisocmerezh" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><ul class="social">
<li class="srch"><a href="/searcher" title="Пошук інформації на сайтах мережі КПІ - search"> </a></li>
<li class="fb"><a href="https://www.facebook.com/kpiua1898" title="Офіційна сторінка КПІ в Facebook" target="_blank"> </a></li>
<li class="yt"><a href="https://www.youtube.com/@kpiua" title="Офіційний youtube канал КПІ" target="_blank"> </a></li>
<li class="te"><a href="/telegram" title="Офіційні канали КПІ"> </a></li>
<li class="in"><a href="https://www.linkedin.com/company/kpiua" title="Офіційна сторінка КПІ в Linkedin" target="_blank"> </a></li>
<li class="ig"><a href="https://www.instagram.com/kpi.ua" title="Офіційний Instagram КПІ" target="_blank"> </a></li>
<li class="go"><a href="https://g.page/r/CfNrIZFdTlFHEBA" title="Офіційний сторінка на гугл-мапах КПІ ім.Ігоря Сікорського" target="_blank"> </a></li>
<li class="tt"><a href="https://www.tiktok.com/@kpi.ua" alt="офіційний тікток університету" target="_blank"> </a></li>
<li class="ukl"><a href="/" title="КПІ"> </a></li>
<li class="enl"><a href="/en/" title="KPI"> </a></li>
</ul>
</div>
      
    </div><!--/.block-content -->
</div><!--/.block -->
<div id="block-thex-newslibrary" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><ul class="inews">
<li class="nws"><a href="/news" alt="Новини КПІ" title="Новини КПІ"> </a></li>
<li class="ann"><a href="/boardlist" alt="Анонси КПІ" title="Анонси КПІ"> </a></li>
<li class="roz"><a href="https://schedule.kpi.ua/" title="Розклад для студентів та викладачів" alt="Розклад для студентів та викладачів"> </a></li>
<li class="ec"><a href="https://ecampus.kpi.ua" alt="Електронний кампус" title="Електронний кампус"> </a></li>
<li class="lib"><a href="https://library.kpi.ua" alt="Бібліотека КПІ" title="Бібліотека КПІ"> </a></li>
<li class="ist"><a href="https://istudent.kpi.ua" alt="Центр міжнародної освіти" title="For foreign students"> </a></li>
</ul>
</div>
      
    </div><!--/.block-content -->
</div><!--/.block -->
<div id="block-thex-mainnavigation" class="block">
  
    
  <div class="block-content">
      
<ul id="superfish-main" class="menu sf-menu sf-main sf-horizontal sf-style-none">
  
<li id="main-menu-link-content239e18aa-a37c-43e7-846b-b8027f29692f" class="sf-depth-1 menuparent"><a href="/about" title="Про КПІ ім. Ігоря Сікорського: загальна інформація та новини" class="sf-depth-1 menuparent">Університет</a><ul><li id="main-menu-link-content684c89ca-a4ad-4ed6-bcdf-1c8053490aaf" class="sf-depth-2 menuparent"><a href="/university" title="Загальна інформація про Національний технічний університет України" class="sf-depth-2 menuparent">Університет сьогодні</a><ul><li id="main-menu-link-contentd1b89b6c-82b7-44f7-a09c-c0769fa7f604" class="sf-depth-3 sf-no-children"><a href="/direction" title="Керівні підрозділи КПІ ім.Ігоря Сікорського" class="sf-depth-3">Керівництво</a></li><li id="main-menu-link-content023c8ecb-d00d-42b6-9772-25a99cd3476e" class="sf-depth-3 sf-no-children"><a href="/sb" title="Наглядова рада університету" class="sf-depth-3">Наглядова рада</a></li><li id="main-menu-link-contentae9ef46f-6499-4cf2-8794-85b495869ce2" class="sf-depth-3 sf-no-children"><a href="/labor-conference" title="Конференції трудового колективу КПІ ім. Ігоря Сікорського" class="sf-depth-3">Конференція трудового колективу</a></li><li id="main-menu-link-content4cedfdb3-3eb2-4c26-9d9e-1e8d463253f6" class="sf-depth-3 sf-no-children"><a href="/vr" title="Вчена рада КПІ ім.Ігоря Сікорського" class="sf-depth-3">Вчена рада</a></li><li id="main-menu-link-content01226046-dcba-4b0d-aff7-983d9ab23d49" class="sf-depth-3 sf-no-children"><a href="/structure" title="Структура університету, навчально-наукові підрозділи та інші організації КПІ ім.Ігоря Сікорського" class="sf-depth-3">Структура університету</a></li><li id="main-menu-link-content15da96ea-8b17-4ae8-9e77-11664b4ebbb7" class="sf-depth-3 sf-no-children"><a href="/administration" title="Адміністративні новини ректорату, структурних підрозділів КПІ ім.Ігоря Сікорського" class="sf-depth-3">Адміністративні новини</a></li></ul></li><li id="main-menu-link-contente361b962-2b52-4f74-a953-d6053fa8ed1a" class="sf-depth-2 menuparent"><a href="/information" title="Інформація КПІ ім.Ігоря Сікорського, яка підлягає обов&#039;язковому оприлюдненню" class="sf-depth-2 menuparent">Офіційна інформація</a><ul><li id="main-menu-link-content1cfcb7b9-d42c-49d7-86eb-d10cf6948a44" class="sf-depth-3 sf-no-children"><a href="/public" title="Надання публічної інформації КПІ ім.Ігоря Сікорського" class="sf-depth-3">Доступ до публічної інформації</a></li><li id="main-menu-link-contentbcabe87d-5bdb-413c-8942-aa4f154687da" class="sf-depth-3 sf-no-children"><a href="/statute" title="СТАТУТ КПІ ім. Ігоря Сікорського" class="sf-depth-3">Статут</a></li><li id="main-menu-link-content0470747e-2898-4a8f-85db-d8079734e434" class="sf-depth-3 sf-no-children"><a href="/licence" title="Ліцензії на провадження освітньої діяльності" class="sf-depth-3">Ліцензії</a></li><li id="main-menu-link-content755159ea-35bc-494f-b7f3-1eaa25ab4c57" class="sf-depth-3 sf-no-children"><a href="/strategy" title="Стратегія розвитку КПІ ім. Ігоря Сікорського" class="sf-depth-3">Стратегія розвитку</a></li><li id="main-menu-link-content4ed8ce31-b903-4193-bdf8-8469cf58cf30" class="sf-depth-3 sf-no-children"><a href="/sustainability/strategy" class="sf-depth-3">Стратегія сталого розвитку</a></li><li id="main-menu-link-content90db3877-2d22-44f9-93d6-135e3154c7c7" class="sf-depth-3 sf-no-children"><a href="/policy" title="Політики КПІ ім.Ігоря Сікорського" class="sf-depth-3">Політики університету</a></li><li id="main-menu-link-content227f9e94-8bd7-4590-8036-db4c51b64570" class="sf-depth-3 sf-no-children"><a href="/report" title="Звіти ректора, департаментів та загальноуніверситетських служб КПІ ім.Ігоря Сікорського" class="sf-depth-3">Звітні доповіді</a></li><li id="main-menu-link-content17e1a59f-3120-4f29-9105-945538353a7e" class="sf-depth-3 sf-no-children"><a href="/rector-election" title="ВИБОРИ РЕКТОРА НАЦІОНАЛЬНОГО ТЕХНІЧНОГО УНІВЕРСИТЕТУ УКРАЇНИ «КИЇВСЬКИЙ ПОЛІТЕХНІЧНИЙ ІНСТИТУТ ІМЕНІ ІГОРЯ СІКОРСЬКОГО»" class="sf-depth-3">Вибори ректора</a></li><li id="main-menu-link-contentc800b3f7-7e19-4787-bae0-3b98fc05ea4b" class="sf-depth-3 sf-no-children"><a href="/code" title="Кодекс честі КПІ ім. Ігоря Сікорського" class="sf-depth-3">Кодекс честі</a></li><li id="main-menu-link-content076987d9-baa3-458c-b428-f81871140203" class="sf-depth-3 sf-no-children"><a href="/admin-rule" title="Правила внутрішнього розпорядку КПІ ім. Ігоря Сікорського" class="sf-depth-3">Правила внутрішнього розпорядку</a></li><li id="main-menu-link-content3f8c11e4-3056-4b1d-9c14-a80a8c1dfeab" class="sf-depth-3 sf-no-children"><a href="/collective_agreement" title="Колективний договiр Національного технічного університету України «Київський політехнічний інститут імені Ігоря Сікорського» на річний період" class="sf-depth-3">Колективний договiр</a></li><li id="main-menu-link-content43cd7d0c-ee25-4900-ba87-a3e7b2f5e681" class="sf-depth-3 sf-no-children"><a href="/attribute" title="Герб, прапор, банери, емблеми та інші офіційні атрибути КПІ ім.Ігоря Сікорського" class="sf-depth-3">Атрибути Університету</a></li></ul></li><li id="main-menu-link-content9220e450-46d4-4115-8a5f-1667615a70fc" class="sf-depth-2 menuparent"><a href="/kpi_faculty" title="Інститути та факультети КПІ ім. Ігоря Сікорського" class="sf-depth-2 menuparent">Інститути &amp; факультети</a><ul><li id="main-menu-link-contentac69c771-efeb-42b5-8b42-521235b1668a" class="sf-depth-3 sf-no-children"><a href="/institutes" title="Інститути КПІ ім.Ігоря Сікорського" class="sf-depth-3">Інститути</a></li><li id="main-menu-link-content6f205f92-0027-4577-a364-310ce20b64e2" class="sf-depth-3 sf-no-children"><a href="/faculties" title="Факультети" class="sf-depth-3">Факультети</a></li><li id="main-menu-link-content8de674ea-7146-4334-804d-2a29c721168d" class="sf-depth-3 sf-no-children"><a href="/chairs" title="Кафедри" class="sf-depth-3">Кафедри</a></li></ul></li><li id="main-menu-link-content88acf53a-2311-4155-80f2-8ca8477c7801" class="sf-depth-2 menuparent"><a href="/almamater" title="Історія КПІ ім. Ігоря Сікорського та його підрозділів" class="sf-depth-2 menuparent">Історія</a><ul><li id="main-menu-link-content1179b9a8-f52a-4479-8e65-d8265c37213e" class="sf-depth-3 sf-no-children"><a href="/rectors" title="Ректори КПІ, КІІ, НТУУ КПІ, КПІ ім. Ігоря Сікорського" class="sf-depth-3">Ректори КПІ</a></li><li id="main-menu-link-content9d2bdca5-aae5-476d-88f8-d245f07d08a1" class="sf-depth-3 sf-no-children"><a href="/sikorsky" title="Сікорський Ігор Іванович - всесвітньо відомий авіаконструктор, творець гелікоптерів, студент КПІ" class="sf-depth-3">Сікорський Ігор</a></li><li id="main-menu-link-contenteb37f970-a0b5-4820-b715-aef4dfd9b935" class="sf-depth-3 sf-no-children"><a href="/kpi_people" title="Всесвітньо відомі вчені - засновники науково-педагогічних шкіл КПІ ім. Ігоря Сікорського" class="sf-depth-3">Видатні вчені КПІ</a></li><li id="main-menu-link-contente7653512-0cca-4d72-9148-81ed7323a140" class="sf-depth-3 sf-no-children"><a href="/honorary" title="Почесні професори та доктори КПІ ім. Ігоря Сікорського" class="sf-depth-3">Почесні професори та доктори</a></li><li id="main-menu-link-content54062977-4556-42d5-8c00-1b3eae67e23e" class="sf-depth-3 sf-no-children"><a href="/history-essay-1995" title="Нарис історії: Київський Політехнічний інститут (книга 1995 року)" class="sf-depth-3">Нарис історії</a></li><li id="main-menu-link-contentcdd188a1-2bda-45e0-aba6-d430eef1d9bd" class="sf-depth-3 sf-no-children"><a href="/gallery/history" title="Колекція старих фотографій" class="sf-depth-3">Історичні фотографії</a></li><li id="main-menu-link-content586a86db-d735-4da2-ab17-4bcc7d0ce904" class="sf-depth-3 sf-no-children"><a href="/kp" title="Щотижнева газета «Київський політехнік»" class="sf-depth-3">Газета «Київський політехнік»</a></li></ul></li><li id="main-menu-link-contentad13b5e7-e29a-4976-92c6-25c3af557b6a" class="sf-depth-2 menuparent"><a href="/kpi_links" title="Департамент міжнародного співробітництва КПІ ім. Ігоря Сікорського" class="sf-depth-2 menuparent">Міжнародне співробітництво</a><ul><li id="main-menu-link-content65852708-b82b-4df6-a30f-65e75fa17e60" class="sf-depth-3 sf-no-children"><a href="/partners" title="Співробітництво КПІ ім. Ігоря Сікорського із зарубіжними закладами" class="sf-depth-3">ВНЗ - партнери</a></li><li id="main-menu-link-content89023e03-8050-4782-99cd-332a1a6f693e" class="sf-depth-3 sf-no-children"><a href="/collaboration" title="Найбільш важливі міжнародні проекти, які виконуються в КПІ ім. Ігоря Сікорського" class="sf-depth-3">Міжнародні проекти</a></li><li id="main-menu-link-content03ed3769-a0ad-45f7-89eb-49cac2ae1fc4" class="sf-depth-3 sf-no-children"><a href="/link" title="Міжнародні спільні структури КПІ ім. Ігоря Сікорського з іноземними організаціями та з університетами-партнерами" class="sf-depth-3">Спільні структури</a></li><li id="main-menu-link-content4471e783-ee49-4588-b93a-d5ff95d700b6" class="sf-depth-3 sf-no-children"><a href="/international_department" title="Новини міжнародної кооперації та взаємодії КПІ ім. Ігоря Сікорського" class="sf-depth-3">Новини міжнародного співробітництва</a></li></ul></li><li id="main-menu-link-content70c0939b-6c4d-4c65-a35f-21752133f26f" class="sf-depth-2 menuparent"><a href="/location" title="Територіальне розміщення університету, карта, мапа, схеми доїзду до корпусів КПІ ім. Ігоря Сікорського" class="sf-depth-2 menuparent">Адреса та карта</a><ul><li id="main-menu-link-content6594ada8-43f3-4354-b403-8c3e83371d67" class="sf-depth-3 sf-no-children"><a href="/building" title="Корпуси КПІ ім. Ігоря Сікорського" class="sf-depth-3">Корпуси КПІ</a></li><li id="main-menu-link-contentb72472e5-c767-4752-aaf5-e6bac5c6dd71" class="sf-depth-3 sf-no-children"><a href="/hostel" title="Гуртожитки КПІ ім. Ігоря Сікорського" class="sf-depth-3">Гуртожитки КПІ</a></li><li id="main-menu-link-content81469fa1-adf6-49bc-ae73-eb194a967d5b" class="sf-depth-3 sf-no-children"><a href="/panorama" title="Панорами Університетського містечка, аудиторій, музею" class="sf-depth-3">Панорами</a></li><li id="main-menu-link-content732c67ec-5672-4c3c-9244-356e2db5b4bc" class="sf-depth-3 sf-no-children"><a href="/park" title="Університетський парк" class="sf-depth-3">Парк КПІ</a></li><li id="main-menu-link-content413338c8-6ed8-4f84-9c49-9522d4cf8a54" class="sf-depth-3 sf-no-children"><a href="/resort" title="У розпорядженні спільноти КПІ ім. Ігоря Сікорського є чотири бази відпочинку з максимальними зручностями" class="sf-depth-3">Бази відпочинку</a></li><li id="main-menu-link-content99b7de60-2028-452e-a674-07d4897d1cc5" class="sf-depth-3 sf-no-children"><a href="/map-transport" title="Громадський транспорт в районі КПІ ім. Ігоря Сікорського" class="sf-depth-3">Громадський транспорт</a></li><li id="main-menu-link-content88bbc70d-80cc-4f16-a525-1abd16e2a321" class="sf-depth-3 sf-no-children"><a href="/telephone" title="Гарячі телефони служб університету" class="sf-depth-3">Важливі телефони</a></li><li id="main-menu-link-content98051cee-abe4-4004-b24d-c25f79d43dc0" class="sf-depth-3 sf-no-children"><a href="/mail" title="Корпоративна пошта НТУУ «КПІ»" class="sf-depth-3">Корпоративна пошта</a></li></ul></li><li id="main-menu-link-contentb5d8c053-5635-4a9c-b87d-328ae693b78b" class="sf-depth-2 sf-no-children"><a href="/union" title="Профспiлкові організації КПІ ім. Ігоря Сікорського, професійні спілки університету" class="sf-depth-2">Профспiлкові організації</a></li><li id="main-menu-link-contentb2df79a3-f6eb-4e43-9f8a-16740471d130" class="sf-depth-2 sf-no-children"><a href="/prozorro" class="sf-depth-2">Публічні закупівлі</a></li><li id="main-menu-link-content4e7a5c30-974f-4e12-b97e-d5f7edea3414" class="sf-depth-2 menuparent"><a href="/2025-04-14-anticor" title="Лінія для повідомлень про корупційні правопорушення з боку посадових осіб та працівників КПІ ім. Ігоря Сікорського" class="sf-depth-2 menuparent">Антикорупційні заходи</a><ul><li id="main-menu-link-content9c0d6405-b395-4cdf-b5f3-353397b20883" class="sf-depth-3 sf-no-children"><a href="/about-anticor" title="Повідомлення про прояви корупції в КПІ ім. Ігоря Сікорського" class="sf-depth-3">Повідомлення про прояви корупції</a></li><li id="main-menu-link-contente76402d3-0fab-4653-a3c3-f356f55615e5" class="sf-depth-3 sf-no-children"><a href="/anticor-programm" title="Антикорупційна програма КПІ ім. Ігоря Сікорського" class="sf-depth-3">Антикорупційна програма</a></li><li id="main-menu-link-content7954f7e8-afcd-4da5-b826-890d909e716e" class="sf-depth-3 sf-no-children"><a href="/law-anticor" title="Міжнародні акти та акти антикорупційного законодавства України" class="sf-depth-3">Антикорупційне законодавство</a></li></ul></li><li id="main-menu-link-content9558ab78-b0fd-4cfd-a5d9-ee7bc5a64c45" class="sf-depth-2 sf-no-children"><a href="/jobs" title="Оголошення конкурсу, вакансії в КПІ" class="sf-depth-2">Вакансії в КПІ</a></li><li id="main-menu-link-contentdc9db42f-e424-4f9c-9441-1247c7802dca" class="sf-depth-2 menuparent"><a href="/offices" title="Загальноуніверситетські новини КПІ ім. Ігоря Сікорського" class="sf-depth-2 menuparent">Університетські новини</a><ul><li id="main-menu-link-content1de4902d-733b-4b0f-8414-9e38a8bfdc77" class="sf-depth-3 sf-no-children"><a href="/top" title="КПІ ім. Ігоря Сікорського в рейтингах" class="sf-depth-3">Університет в рейтингах</a></li><li id="main-menu-link-contente85f9e81-f4bf-48ac-9c80-60e66fb1ebbe" class="sf-depth-3 sf-no-children"><a href="/f_session" title="Сесії професорсько-викладацького складу КПІ ім. Ігоря Сікорського" class="sf-depth-3">Сесія професорсько-викладацького складу </a></li></ul></li></ul></li><li id="main-menu-link-content6db97e12-a880-4444-b1d4-d20cb96193ae" class="sf-depth-1 menuparent"><a href="/admission" title="Інформація для вступників в КПІ ім. Ігоря Сікорського" class="sf-depth-1 menuparent">Вступ</a><ul><li id="main-menu-link-content178c4b65-2e32-4f03-a64d-9b6a7363c436" class="sf-depth-2 sf-no-children"><a href="/opendoors" title="Дні відкритих дверей в КПІ ім. Ігоря Сікорського, день відкритих дверей на рівних факультетах та інститутах" class="sf-depth-2">Дні відкритих дверей</a></li><li id="main-menu-link-content0cbdb7c7-907d-4f27-8390-f9c90ebb7ff2" class="sf-depth-2 menuparent"><a href="/work" title="Інформація про приймальну комісію КПІ ім. Ігоря Сікорського" class="sf-depth-2 menuparent">Приймальна комісія</a><ul><li id="main-menu-link-content86daf149-1417-491f-abcd-740781a6543f" class="sf-depth-3 sf-no-children"><a href="/contract" title="Вартість навчання громадян України та прирівнених до них осіб за кошти фізичних та юридичних осіб - контракт" class="sf-depth-3">Вартість навчання</a></li><li id="main-menu-link-contentf201139d-06e4-455d-bedd-3a293b8840fb" class="sf-depth-3 sf-no-children"><a href="/license-plan" title="Ліцензований обсяг прийому та обсяг державного замовлення в КПІ ім. Ігоря Сікорського" class="sf-depth-3">Обсяг прийому</a></li></ul></li><li id="main-menu-link-content9e2f1a94-a9f6-4581-9cca-76350cf83169" class="sf-depth-2 sf-no-children"><a href="/rule-admission" title="Правила прийому до КПІ ім. Ігоря Сікорського" class="sf-depth-2">Правила прийому</a></li><li id="main-menu-link-content4ddd7068-4af6-4ffa-a5ab-047041320466" class="sf-depth-2 sf-no-children"><a href="/applicant" title="Інформація для вступників на ступінь вищої освіти «бакалавр» в КПІ ім. Ігоря Сікорського" class="sf-depth-2">Вступ на бакалаврат</a></li><li id="main-menu-link-content749c0d04-c46b-44a4-918d-ea12abbd3340" class="sf-depth-2 sf-no-children"><a href="/master" title="Інформація для вступників на ступінь вищої освіти «магістр» в КПІ ім. Ігоря Сікорського" class="sf-depth-2">Вступ в магістратуру</a></li><li id="main-menu-link-content87dcee8c-81da-491e-b776-9d2355b35fbb" class="sf-depth-2 sf-no-children"><a href="https://pk.kpi.ua/official-documents/vstup-dlia-veteraniv/" title="Олімпіади для абітурієнтів" class="sf-depth-2 sf-external">Вступ для ветеранів</a></li><li id="main-menu-link-content775c74f7-5e0c-438e-8a39-dc7350041857" class="sf-depth-2 sf-no-children"><a href="https://pk.kpi.ua/donbass-crimea/" title="Освітні центри «Донбас-Україна» та «Крим-Україна»" class="sf-depth-2 sf-external">Для вступників з ТОТ</a></li><li id="main-menu-link-content596ad08b-7fe6-4f61-95e1-af049fd110df" class="sf-depth-2 menuparent"><a href="/phd" title="Інформація для вступників для здобуття ступеня доктора філософії та доктора наук. Аспірантура та докторантура в КПІ ім. Ігоря Сікорського" class="sf-depth-2 menuparent">Аспірантура &amp; Докторантура</a><ul><li id="main-menu-link-content3fd3bf2e-e1e3-4a3b-bfa7-2a5617d9ff09" class="sf-depth-3 sf-no-children"><a href="http://aspirantura.kpi.ua" class="sf-depth-3 sf-external">Відділ аспірантури та докторантури</a></li><li id="main-menu-link-content2d3cd372-0257-4bd6-af58-21767a4c2ce9" class="sf-depth-3 sf-no-children"><a href="http://aspirantura.kpi.ua/?page_id=172" title="Прийом документів на конкурс до аспірантури КПІ ім. Ігоря Сікорського" class="sf-depth-3 sf-external">Умови прийому до аспірантури</a></li><li id="main-menu-link-content8812b78d-320c-403c-bca1-93affc6c23bc" class="sf-depth-3 sf-no-children"><a href="http://aspirantura.kpi.ua/?page_id=304" title="Умови прийому до докторантури КПІ ім. Ігоря Сікорського" class="sf-depth-3 sf-external">Умови прийому до докторантури</a></li></ul></li><li id="main-menu-link-contentbb0e345b-0f56-4abc-b2c0-c78083f6fb4e" class="sf-depth-2 menuparent"><a href="/foreigners" class="sf-depth-2 menuparent">Вступ іноземних громадян</a><ul><li id="main-menu-link-content1c05312f-536b-45e4-90b6-a87f6307e7be" class="sf-depth-3 sf-no-children"><a href="/nation" title="Держава Центрально–Схiдної Європи Україна." class="sf-depth-3">Україна</a></li><li id="main-menu-link-content97b9c1eb-0894-4ba5-9eff-e8a5d16a2375" class="sf-depth-3 sf-no-children"><a href="/kyiv" title="Київ – столиця України" class="sf-depth-3">Київ</a></li><li id="main-menu-link-content46b4eb5c-2d29-4931-aebd-5feb29efc340" class="sf-depth-3 sf-no-children"><a href="/kpi_cmo" title="Центр міжнародної освіти КПІ ім. Ігоря Сікорського" class="sf-depth-3">Центр міжнародної освіти</a></li><li id="main-menu-link-content2897897b-a0ac-40ce-b4eb-cbe192e3a062" class="sf-depth-3 sf-no-children"><a href="https://forms.office.com/r/ABk9u2GKJB" title="Анкета для іноземних громадян, які бажають вступити до КПІ ім. Ігоря Сікорського" class="sf-depth-3 sf-external">Анкета для іноземних громадян</a></li><li id="main-menu-link-content8be6ca5e-9ee0-4460-8f21-63dcf3b447d1" class="sf-depth-3 sf-no-children"><a href="/preparatory-department" title="Підготовче відділення для іноземців" class="sf-depth-3">Підготовче відділення ЦМО</a></li></ul></li><li id="main-menu-link-content43e7c216-8e8f-4df2-94f0-3fad453d08c8" class="sf-depth-2 sf-no-children"><a href="/fdp" title="Доуніверситетська пiдготовка в КПІ ім. Ігоря Сікорського" class="sf-depth-2">Доуніверситетська пiдготовка</a></li><li id="main-menu-link-contenta084096d-bc7d-406a-bea0-6f8e5b5c7e78" class="sf-depth-2 sf-no-children"><a href="/training" title="Післядипломна освіта, підвищення кваліфікації, друга освіта та наступна освіта в КПІ ім. Ігоря Сікорського" class="sf-depth-2">Післядипломна освіта</a></li><li id="main-menu-link-contentf5ffb808-b468-4cb2-b57b-62c60ceb89e7" class="sf-depth-2 sf-no-children"><a href="/admissions" title="Новини приймальної кампанії" class="sf-depth-2">Новини приймальної кампанії</a></li></ul></li><li id="main-menu-link-content92129f0c-fb6a-4f10-b4c2-7cf5af8890e0" class="sf-depth-1 menuparent"><a href="/education" class="sf-depth-1 menuparent">Освіта</a><ul><li id="main-menu-link-contenta3937217-2bb9-4990-98ce-6b35fe1a7a1c" class="sf-depth-2 sf-no-children"><a href="/op" title="Перелік освітніх програм КПІ ім. Ігоря Сікорського" class="sf-depth-2">Освітні програми</a></li><li id="main-menu-link-contentf35a53e5-dc0f-4b02-8a6d-ac2d9bfd23bb" class="sf-depth-2 menuparent"><a href="/regulations" title="Положення про організацію освітнього процесу" class="sf-depth-2 menuparent">Організація навчання</a><ul><li id="main-menu-link-content805bf0d0-f22f-47aa-ba9f-1664c2ace95e" class="sf-depth-3 sf-no-children"><a href="/regulations-2-1" title="Ступенева  система освіти (рівні підготовки)" class="sf-depth-3">Ступенева система освіти</a></li><li id="main-menu-link-contentc7fc7d34-cc58-4a2b-a36a-21ae7a7ccd0c" class="sf-depth-3 sf-no-children"><a href="/taxonomy/term/2623" title="Положення Про Організацію Навчального Процесу В НТУУ “КПІ”: 4. Планування навчального процесу" class="sf-depth-3">Планування навчального процесу</a></li><li id="main-menu-link-content2b441c60-821f-4fb4-9312-d61e4d5da649" class="sf-depth-3 sf-no-children"><a href="/regulations-1-2" class="sf-depth-3">Форми навчання</a></li><li id="main-menu-link-contentf118ed68-a3ac-4161-8764-c8e2f8cb85c8" class="sf-depth-3 sf-no-children"><a href="/forms-of-ep" title="Тимчасове положення про організацію освітнього процесу в НТУУ «КПІ»: 6. Форми організації освітнього процесу " class="sf-depth-3">Форми організації освітнього процесу</a></li></ul></li><li id="main-menu-link-contentf9cdd3b5-519b-4458-99e2-0910b566a4cb" class="sf-depth-2 sf-no-children"><a href="/year" title="Графік навчального процесу в КПІ ім. Ігоря Сікорського на поточний навчальний рік" class="sf-depth-2">Академічний календар</a></li><li id="main-menu-link-contentd7ba288c-38b2-48ed-a788-7d6ef7bc96e6" class="sf-depth-2 sf-no-children"><a href="/edu" title="Вісник НТУУ &quot;КПІ&quot;. Філософія. Психологія. Педагогіка" class="sf-depth-2">Корпоративне середовище</a></li><li id="main-menu-link-contente601c361-b57c-43a1-a7cb-7d9b588da066" class="sf-depth-2 sf-no-children"><a href="https://do.ipo.kpi.ua/" title="Платформа дистанційного навчання &quot;Сікорський&quot;" class="sf-depth-2 sf-external">Дистанційне навчання</a></li><li id="main-menu-link-contentc71ec276-6218-4112-b4a8-cdcc4eb98af2" class="sf-depth-2 sf-no-children"><a href="/iam" title="Інформація про відділ академічної мобільності" class="sf-depth-2">Академічна мобільність</a></li><li id="main-menu-link-contentcae9d45a-6358-4af3-b6d9-cbe70bbe898f" class="sf-depth-2 menuparent"><a href="/vzyavo" title="Внутрішнє забезпечення якості вищої освіти" class="sf-depth-2 menuparent">Внутрішнє забезпечення якості</a><ul><li id="main-menu-link-contentfe5a566f-e582-4f97-bcbc-2b6d525a5be6" class="sf-depth-3 sf-no-children"><a href="/policy/quality-education" title="Політика забезпечення якості вищої освіти" class="sf-depth-3">Політика забезпечення якості ВО</a></li></ul></li><li id="main-menu-link-content85a11e29-bc9c-440d-ba84-5ce4278dbc2d" class="sf-depth-2 sf-no-children"><a href="/m-counsil" title="Методична Рада Київський політехнічний інститут ім. Ігоря Сікорського" class="sf-depth-2">Методична рада</a></li><li id="main-menu-link-content1fcccd1c-2fe5-4441-8a65-849db26a66f5" class="sf-depth-2 sf-no-children"><a href="/education_department" class="sf-depth-2">Новини освіти</a></li></ul></li><li id="main-menu-link-contentdda29401-8b0b-40a4-9f26-782795d4391e" class="sf-depth-1 menuparent"><a href="/science" title="Наука та інновації" class="sf-depth-1 menuparent">Наука</a><ul><li id="main-menu-link-content3d474f24-25f1-44e6-b8d6-d4be18151a73" class="sf-depth-2 sf-no-children"><a href="/science/achievement.html" class="sf-depth-2">Лауреати державних премій</a></li><li id="main-menu-link-content2affba8f-fceb-4091-a662-c111b337619a" class="sf-depth-2 sf-no-children"><a href="/research" title="Перелік напрямів фундаментальних та прикладних наукових досліджень" class="sf-depth-2">Напрями досліджень</a></li><li id="main-menu-link-content2b207242-1ee2-494f-a2b6-bf4d57ca79d8" class="sf-depth-2 sf-no-children"><a href="/science/publication" title="Наукові видання НТУУ КПІ" class="sf-depth-2">Наукові видання</a></li><li id="main-menu-link-content354257ba-c70e-4da2-8cb2-62f54f255435" class="sf-depth-2 menuparent"><a href="/norma" class="sf-depth-2 menuparent">Конкурси НДЧ</a><ul><li id="main-menu-link-content2c51756c-56cb-4aa1-a063-3e29837410db" class="sf-depth-3 sf-no-children"><a href="/teacher-researcher" title="Конкурс на номінації «Викладач-дослідник» та «Молодий викладач-дослідник»" class="sf-depth-3">Конкурс «Викладач-дослідник»</a></li><li id="main-menu-link-content85c7a06b-5d62-4725-ace5-ec15fc3eec3f" class="sf-depth-3 sf-no-children"><a href="/best-textbooks-competition" title="Конкурс на здобуття премій НТУУ «КПІ» за кращі підручники, навчальні посібники та монографії" class="sf-depth-3">Конкурс на кращі підручники</a></li></ul></li><li id="main-menu-link-content37b5aa10-f03c-4572-bc70-c3576ad3c04a" class="sf-depth-2 sf-no-children"><a href="/seminar" title="Конференції, семінари, форуми, з&#039;їзди" class="sf-depth-2">План наукових заходів</a></li><li id="main-menu-link-content0d31b284-45d3-40b9-8549-7bc35abeea7b" class="sf-depth-2 sf-no-children"><a href="/ecoino" title="Інноваційна екосистема Sikorsky Challenge Національного технічного університету України Київський політехнічний інститут (INESC)" class="sf-depth-2">Інноваційна екосистема</a></li><li id="main-menu-link-contentf02b4084-a1ca-4c41-9ef6-e301b6bde8e5" class="sf-depth-2 sf-no-children"><a href="/rmv" title="Рада молодих вчених НТУУ КПІ" class="sf-depth-2">Рада молодих вчених</a></li><li id="main-menu-link-contente2347ab1-f50b-40a7-8f97-d0e22278fbc5" class="sf-depth-2 sf-no-children"><a href="/department" title="Наукові підрозділи НТУУ «КПІ»: НДІ, НЦ, НТЦ, КБ, спеціалізовані відділи" class="sf-depth-2">Наукові підрозділи</a></li><li id="main-menu-link-contentb722aad3-8ea6-4a6e-942a-5fa49a38896c" class="sf-depth-2 menuparent"><a href="/science_department" class="sf-depth-2 menuparent">Новини науки</a><ul><li id="main-menu-link-content4415edc7-28ce-492a-a787-9e34924afc47" class="sf-depth-3 sf-no-children"><a href="/use" title="Рубрика з інформаційними та пізнавальними матеріалами щодо популяризації природничих наук та математики" class="sf-depth-3">Популяризація науки</a></li></ul></li></ul></li><li id="main-menu-link-content6c1a47cc-4ae3-4fe5-bf7f-7878b0b3a06c" class="sf-depth-1 menuparent"><a href="/kpi_students" title="Навчання, відпочинок та проживання студентів в університетському містечку" class="sf-depth-1 menuparent">Студентство</a><ul><li id="main-menu-link-contentc2890117-612b-42f1-afac-bcde894093dd" class="sf-depth-2 menuparent"><a href="/forms-gle" title="Умови навчання та дозвілля в університеті." class="sf-depth-2 menuparent">Освітній процес</a><ul><li id="main-menu-link-content37adbf1c-f099-4bba-badc-25a29faabb47" class="sf-depth-3 sf-no-children"><a href="/vnvr" class="sf-depth-3">Навчання</a></li><li id="main-menu-link-content67e8d4f6-1d8a-4057-a538-a343cadcfb97" class="sf-depth-3 sf-no-children"><a href="/inclusion" title="Підтримка навчання в університеті студентів з особливими освітніми потребами" class="sf-depth-3">Інклюзивне навчання</a></li><li id="main-menu-link-contente43094c8-e82e-462d-bb36-e6bc7ca6cd11" class="sf-depth-3 sf-no-children"><a href="/stipend" title="Стипендії, персональні та академічні стипендії, соціальна допомога, стипендіальне та соціальне забезпечення студентів" class="sf-depth-3">Стипендія</a></li><li id="main-menu-link-contentbec46c2a-50e8-458c-aa6c-cc259f81463d" class="sf-depth-3 sf-no-children"><a href="/vmv" class="sf-depth-3">Військовий облік</a></li></ul></li><li id="main-menu-link-content18778b44-b586-401e-a218-898cf8369643" class="sf-depth-2 menuparent"><a href="/web-studmisto" title="Умови навчання, відпочинку і проживання для студентів, викладачів та співробітників КПІ ім.Ігоря Сікорського" class="sf-depth-2 menuparent">Умови проживання</a><ul><li id="main-menu-link-content96b4def9-8257-4af3-a65b-4422e7c0e653" class="sf-depth-3 sf-no-children"><a href="/web_studmisto" title="Студмісто - Студентське містечко КПІ ім.Ігоря Сікорського, гуртожитки університету, документи з поселення та проживання" class="sf-depth-3">Студмістечко</a></li><li id="main-menu-link-content5e2b4890-1808-4ce0-ac05-29f80a05ad29" class="sf-depth-3 sf-no-children"><a href="https://hostelpay.kpi.ua/" title="Оплата онлайн за послуги Студмістечка КПІ ім. Ігоря Сікорського" class="sf-depth-3 sf-external">Оплата онлайн</a></li><li id="main-menu-link-contentbfb5b1b4-a60c-4a8c-9a2b-bac263614fc5" class="sf-depth-3 sf-no-children"><a href="/vors" title="Центр консолідації студентів (ЦКС)" class="sf-depth-3">Центр консолідації студентів</a></li></ul></li><li id="main-menu-link-content9474022e-3697-4368-863f-96a351736704" class="sf-depth-2 sf-no-children"><a href="/cprk" title="Відділ сприяння працевлаштуванню та професійного розвитку" class="sf-depth-2">Кар&#039;єра</a></li><li id="main-menu-link-content816157c8-4f09-4cdb-afbf-1617eac93fa5" class="sf-depth-2 menuparent"><a href="/web_students-events" title="Інформація про студентські заходи, події та студентів КПІ ім.Ігоря Сікорського" class="sf-depth-2 menuparent">Студентська активність</a><ul><li id="main-menu-link-content58e84010-19c9-4c37-a48f-c3788e030230" class="sf-depth-3 sf-no-children"><a href="/vors" title="Позанавчальний процес" class="sf-depth-3">Гуртки. Олімпіади. Проєкти</a></li><li id="main-menu-link-content96fac4d6-d2a3-4ef4-9026-84e55fb9b93e" class="sf-depth-3 sf-no-children"><a href="/athletics" class="sf-depth-3">Спорт</a></li><li id="main-menu-link-content780054c3-0ed8-4094-92b9-ffc3905cd685" class="sf-depth-3 sf-no-children"><a href="/kpi_culture" title="Студентське дозвілля: культура і мистецтво" class="sf-depth-3">Культура</a></li><li id="main-menu-link-content3d918faf-57a8-4073-b59d-40808f664117" class="sf-depth-3 sf-no-children"><a href="/kpi_events" title="Традиційні заходи та події КПІ" class="sf-depth-3">Традиційні заходи</a></li></ul></li><li id="main-menu-link-content612086e2-5f20-4618-a52c-b1cb51615634" class="sf-depth-2 sf-no-children"><a href="/students-rights" title="Права студентів визначені законом України про Вищу освіту" class="sf-depth-2">Захист прав студентів</a></li><li id="main-menu-link-contente5e1b3eb-202d-4d36-b316-a303ef17970a" class="sf-depth-2 menuparent"><a href="/web_health" title="Медичне обслуговування, відпочинок та допомога студентам" class="sf-depth-2 menuparent">Оздоровлення</a><ul><li id="main-menu-link-contentddb2cecf-9691-4619-9a24-457bb3e6e583" class="sf-depth-3 sf-no-children"><a href="/health" title="Медичне обслуговування та медицина в КПІ" class="sf-depth-3">Медзабезпечення</a></li><li id="main-menu-link-contentafb764e4-6e71-4b1c-a6b3-8924e17e2e84" class="sf-depth-3 sf-no-children"><a href="/web_sss" title="Соціально-психологічна підтримка студентів та аспірантів КПІ" class="sf-depth-3">Соціально-психологічна підтримка</a></li></ul></li><li id="main-menu-link-contentadc700dd-3ff8-4774-80b8-e3d9f8539dbd" class="sf-depth-2 sf-no-children"><a href="/web_organizations" title="Студентські організації та проекти" class="sf-depth-2">Студентське самоврядування</a></li><li id="main-menu-link-contentcf5f85dd-cb2e-4ca3-847f-3cc730841373" class="sf-depth-2 sf-no-children"><a href="/board-net" title="Студентські новини Університету" class="sf-depth-2">Новини студентства</a></li></ul></li>
</ul>

    </div><!--/.block-content -->
</div><!--/.block -->

  </div>

	</div>
</div>
         <!--  end if for primary_menu -->
              </div> <!-- /.header-right -->
        </div><!-- /header-container -->
  </div><!-- /container -->
</header><!-- /header -->

  <div class="highlighted block-section">
  <aside class="container" role="complementary">
      <div class="block-region region-highlighted">
    <div class="views-element-container block" id="block-thex-views-block-10-slider-main-block-1-2">
  
    
  <div class="block-content">
      <div><div class="js-view-dom-id-3b0589ce74c79e6a1b0bd15b8d9d04c4d0b7d4cfdad8bc3af459d400d458a587">
  
  
  

  
  
  

    <div class="skin-default">
    
    <div id="views_slideshow_cycle_main_10_slider_main-block_1" class="views_slideshow_cycle_main views_slideshow_main">
    <div id="views_slideshow_cycle_teaser_section_10_slider_main-block_1"  class="views_slideshow_cycle_teaser_section">
     <div id="views_slideshow_cycle_div_10_slider_main-block_1_0"  class="views_slideshow_cycle_slide views_slideshow_slide views-row-1 views-row-odd">
      <div  class="views-row views-row-0 views-row-odd views-row-first">
  <div class="views-field views-field-node-image"><div class="field-content">  <a href="/enter" hreflang="UK"><img loading="lazy" src="/files/styles/mini/public/images-story/2026-eqmi.jpg?itok=paBCRH59" width="1600" height="550" alt="Зимовий вступ у КПІ" title="Зимовий вступ у КПІ" class="image-field" />

</a>
</div></div>
</div>

  </div>

     <div id="views_slideshow_cycle_div_10_slider_main-block_1_1"  class="views_slideshow_cycle_slide views_slideshow_slide views-row-2 views_slideshow_cycle_hidden views-row-even">
      <div  class="views-row views-row-1 views-row-even">
  <div class="views-field views-field-node-image"><div class="field-content">  <a href="/node/21709" hreflang="UK"><img loading="lazy" src="/files/styles/mini/public/images-story/2026-11-27.jpg?itok=m-tKAcqp" width="1600" height="550" alt="KPI Career beAhead Expo 2026" title="KPI Career beAhead Expo 2026" class="image-field" />

</a>
</div></div>
</div>

  </div>

     <div id="views_slideshow_cycle_div_10_slider_main-block_1_2"  class="views_slideshow_cycle_slide views_slideshow_slide views-row-3 views_slideshow_cycle_hidden views-row-odd">
      <div  class="views-row views-row-2 views-row-odd">
  <div class="views-field views-field-node-image"><div class="field-content">  <a href="/node/21600" hreflang="UK"><img loading="lazy" src="/files/styles/mini/public/images-story/2027.jpg?itok=BWW9aIF0" width="1600" height="550" alt="Фінал Sikorsky Challenge 2026" title="Фінал Sikorsky Challenge 2026" class="image-field" />

</a>
</div></div>
</div>

  </div>

     <div id="views_slideshow_cycle_div_10_slider_main-block_1_3"  class="views_slideshow_cycle_slide views_slideshow_slide views-row-4 views_slideshow_cycle_hidden views-row-even">
      <div  class="views-row views-row-3 views-row-even">
  <div class="views-field views-field-node-image"><div class="field-content">  <a href="/seminar" hreflang="UK"><img loading="lazy" src="/files/styles/mini/public/images-story/2024-seminar.jpg?itok=NLWmTrPE" width="1600" height="550" alt="План наукових та науково-технічних заходів" title="План наукових та науково-технічних заходів" class="image-field" />

</a>
</div></div>
</div>

  </div>

     <div id="views_slideshow_cycle_div_10_slider_main-block_1_4"  class="views_slideshow_cycle_slide views_slideshow_slide views-row-5 views_slideshow_cycle_hidden views-row-odd">
      <div  class="views-row views-row-4 views-row-odd views-row-last">
  <div class="views-field views-field-node-image"><div class="field-content">  <a href="/2025-publish" hreflang="UK"><img loading="lazy" src="/files/styles/mini/public/images-story/2025-publish.jpg?itok=xGrnmPcW" width="1600" height="550" alt="Видавничий дім КПІ ім. Ігоря Сікорського" title="Видавничий дім КПІ ім. Ігоря Сікорського" class="image-field" />

</a>
</div></div>
</div>

  </div>

  </div>

</div>


        </div>

    

  
  

  
  
</div>
</div>

    </div><!--/.block-content -->
</div><!--/.block -->

  </div>

  </aside>
</div>


        <div class="main-wrapper">
  <div class="container">
    <a id="main-content" tabindex="-1"></a>
    <div class="main-container">
      <main id="main" class="main-content">
                <div class="node-content">
            <div class="block-region region-content">
    <div data-drupal-messages-fallback class="hidden"></div><div id="block-thex-newsforfront" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><h2 class="allnews">Новини | <a href="/news" class="all">Всі новини</a></h2>
</div>
      
    </div><!--/.block-content -->
</div><!--/.block -->
<div class="views-element-container block" id="block-thex-views-block-allpage-block-1">
  
    
  <div class="block-content">
      <div><div class="frontpagenews js-view-dom-id-2452e9d46f76a84b55d11e37126defbcae0a595bbad49014e1d27b7659951b7a">
  
  
  

  
  
  

  <div class="item-list">
  
  <ul>

          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/mriia-5" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-mriia_1.jpg?itok=qKusPLfW" width="360" height="215" alt="П’ятий інженерний конкурс для школярів «Збудуй свою Мрію»" title="П’ятий інженерний конкурс для школярів «Збудуй свою Мрію»" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/mriia-5" hreflang="UK">🎥 П’ятий інженерний конкурс для школярів «Збудуй свою Мрію»</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-18T22:00:01Z">2026-11-19</time>
</div></div></li>
          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/2026-11-cz" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-11-18-science.jpg?itok=1rmj8a2O" width="360" height="215" alt="делегація з Чехії" title="делегація з Чехії" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/2026-11-cz" hreflang="UK">Нові можливості для українсько-чеської співпраці в науці, інженерії та технологіях</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-18T13:00:00Z">2026-11-18</time>
</div></div></li>
          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/node/21716" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-11-18.jpg?itok=hFxC-shI" width="360" height="215" alt="План управління дослідницькими даними в проєктах &amp;laquo;Горизонт Європа" title="План управління дослідницькими даними в проєктах &amp;laquo;Горизонт Європа" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/node/21716" hreflang="UK">🌍 План управління дослідницькими даними в проєктах &quot;Горизонт Європа&quot;</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-18T13:00:00Z">2026-11-18</time>
</div></div></li>
          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/2026-Berkeley" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-berkeley.jpg?itok=bTf4gLqd" width="360" height="215" alt="візит у США" title="візит у США" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/2026-Berkeley" hreflang="UK">Співпраця із Каліфорнійським університетом у Берклі</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-17T08:00:00Z">2026-11-17</time>
</div></div></li>
          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/2026-kp37-mmsa" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-starovoit.jpg?itok=dumIx95A" width="360" height="215" alt="Тетяна Старовойт" title="Тетяна Старовойт" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/2026-kp37-mmsa" hreflang="UK">Тетяна Старовойт. Інтелектуальні моделі стійкості: як наукові розробки КПІ допомагають захищати інженерні мережі міст</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-16T11:00:00Z">2026-11-16</time>
</div></div></li>
          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/2026-orcid-kbis" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-orcid-cbis.jpg?itok=70wilMzj" width="360" height="215" alt="У KPI ID з&#039;явився вхід через ORCID" title="У KPI ID з&#039;явився вхід через ORCID" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/2026-orcid-kbis" hreflang="UK">У KPI ID з&#039;явився вхід через ORCID</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-16T10:00:00Z">2026-11-16</time>
</div></div></li>
          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/2026-kp37-aemk" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-boichenko_0.jpg?itok=6jFVWAsP" width="360" height="215" alt="Бойченко, Сергій Валерійович, 1968" title="Бойченко, Сергій Валерійович, 1968" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/2026-kp37-aemk" hreflang="UK">Сергій Бойченко: &quot;Якщо хочеш бачити якісні зміни в українській науці, маєш ставати учасником цих змін&quot;</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-16T08:00:00Z">2026-11-16</time>
</div></div></li>
          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/2026-sqe" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-sqe.jpg?itok=xmgj_PZB" width="360" height="215" alt="Від оцінки до можливостей: як змінити оцінювання професійної діяльності педагогів" title="Від оцінки до можливостей: як змінити оцінювання професійної діяльності педагогів" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/2026-sqe" hreflang="UK">🎥 Експертна сесія «Від оцінки до можливостей: як змінити оцінювання професійної діяльності педагогів»</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-14T13:00:00Z">2026-11-14</time>
</div></div></li>
          <li><div class="views-field views-field-node-image"><div class="field-content">  <a href="/2026-3d-cz" hreflang="UK"><img loading="lazy" src="/files/styles/attention/public/images-story/2026-3d.jpg?itok=PQBmTXVX" width="360" height="215" alt="Партнерство із чеською компанією Prusa Research" title="Prusa Research — відомий світовий розробник і виробник 3D-принтерів" class="image-field" />

</a>
</div></div><div class="views-field views-field-title"><span class="field-content"><a href="/2026-3d-cz" hreflang="UK">Партнерство із чеською компанією Prusa Research</a></span></div><div class="views-field views-field-field-end"><div class="field-content"><time datetime="2026-11-13T15:00:00Z">2026-11-13</time>
</div></div></li>
    
  </ul>

</div>

    

  
  

  
  
</div>
</div>

    </div><!--/.block-content -->
</div><!--/.block -->
<div id="block-thex-content" class="block">
  
    
  <div class="block-content">
      <div class="views-element-container"><div class="js-view-dom-id-ef0fb24dae149e045f1bd51d61f652e0870cdcaca28ecbbd7117c10125004b40">
  
  
  

  
  
  

      <div class="views-row"><div class="views-field views-field-body"><div class="field-content"><h1>Місія КПІ ім. Ігоря Сікорського</h1>
<blockquote><p>Реалізовувати науково-освітній та інноваційний потенціал Університету для досягнення цілей сталого (sustainable) розвитку суспільства, високотехнологічної трансформації держави та зміцнення ї ї обороноздатності, формування якісного людського капіталу для відновлення та стійкого (resilient) розвитку України, створення засад поставання соціально-відповідальних фахівців, готових вирішувати глобальні проблеми та змінювати світ. </p></blockquote>
</div></div></div>

    

  
  

  
  
</div>
</div>

    </div><!--/.block-content -->
</div><!--/.block -->

  </div>

        </div>
              </main>
              <aside id="sidebar-right" class="sidebar block-section" role="complementary">
    <div class="block-region region-sidebar-second">
    <div class="views-element-container block" id="block-thex-views-block-announcements-block-1">
  
      <h2 class="block-title">Анонси</h2>
    
  <div class="block-content">
      <div><div class="js-view-dom-id-b4752005bdac412d57e513af187262df83541de0bd6603cce3de85fa8b9d8ae1">
  
  
  

  
  
  

      <div class="views-row"><div class="views-field views-field-title"><span class="field-content"><a href="/enter" hreflang="UK">🥇 Зимовий вступ у КПІ</a></span></div></div>
    <div class="views-row"><div class="views-field views-field-title"><span class="field-content"><a href="/node/21717" hreflang="UK">⚡ Відкрита студентська олімпіада з теорії електронних кіл “ТЕК’2026”</a></span></div></div>
    <div class="views-row"><div class="views-field views-field-title"><span class="field-content"><a href="/node/21714" hreflang="UK">💛💙 Родинна субота у Державному Політехнічному музеї</a></span></div></div>
    <div class="views-row"><div class="views-field views-field-title"><span class="field-content"><a href="/node/21710" hreflang="UK">🏓 Турнір з настільного тенісу для співробітників!</a></span></div></div>
    <div class="views-row"><div class="views-field views-field-title"><span class="field-content"><a href="/node/21709" hreflang="UK">📅📍 KPI Career beAhead Expo 2026. Autumn: познайомся з топовими роботодавцями на ринку</a></span></div></div>
    <div class="views-row"><div class="views-field views-field-title"><span class="field-content"><a href="/inform" hreflang="UK">✔️ Долучайтесь до Всеукраїнської інформаційно-освітньої кампанії «Місяць цифрової грамотності»</a></span></div></div>
    <div class="views-row"><div class="views-field views-field-title"><span class="field-content"><a href="/node/21704" hreflang="UK">🈴 Відкритий турнір із шьоґі, присвячений Дню шьоґі </a></span></div></div>

    

  
  

  
  
</div>
</div>

    </div><!--/.block-content -->
</div><!--/.block -->
<div id="block-thex-studyinenglish" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><p><a href="https://charity.kpi.ua/"><img src="/files/charity.jpg" alt="charitable giving platform" title="charitable giving platform" width="100%"></a><br>
<br><br>
<a href="/about-anticor"><img src="/files/anticor.jpg" alt="повідомити про корупцію в університеті" title="Маєте інформацію щодо проявів корупції? Повідомте нам про це." width="100%"></a></p>
</div>
      
    </div><!--/.block-content -->
</div><!--/.block -->

  </div>

</aside>        </div><!--/main-container -->
  </div><!--/container -->
</div><!--/main-wrapper -->
  <footer class="footer-top footer">
  <div class="container">
    <div class="footer-top-container">
        <div class="block-region region-footer-top">
    <div id="block-thex-footertop2" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><div class="vfront">
<div class="kpin">
<div class="3col">
<iframe width="100%" height="270" src="https://www.youtube.com/embed/rUaTbTn4PEU" title="Відкриття сонячної електростанції в ФІОТ КПІ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe></div>
<div class="3col">
<iframe width="100%" height="270" src="https://www.youtube.com/embed/vfoBUohxqFA" title="КПІшники — серед переможців Student Tech Challenge 2026 від Huawei Ukraine!" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe></div>
<div class="3col">
<iframe width="100%" height="270" src="https://www.youtube.com/embed/pMglQmo2AeE" title="Радіодиктант національної єдності 2026 в КПІ ім. Ігоря Сікорського" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe></div>
<div class="3col">
<iframe width="100%" height="270" src="https://www.youtube.com/embed/5WxxCRn-CIM" title="Спартакіада КПІ 2026: Змагання з настільного тенісу" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe></div>
<div class="3col">
<iframe width="100%" height="270" src="https://www.youtube.com/embed/UGM0MUbxEcc" title="ML-хакатон «Код плазми» у ФМФ КПІ ім. Ігоря Сікорського" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe></div>
<div class="3col">
<iframe width="100%" height="270" src="https://www.youtube.com/embed/mq6QNro0QHE" title="Engineering Project Sprint 2026 у КПІ ім. Ігоря Сікорського" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe></div>
</div>
</div>
</div>
      
    </div><!--/.block-content -->
</div><!--/.block -->
<div id="block-thex-footertop" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><div class="ifront">
<ul class="kpii">
<li class="faculty"><a href="/kpi_faculty" title="Факультети та інститути">Факультети та інститути</a></li>
<li class="op"><a href="https://osvita.kpi.ua/op" title="Освітні програми і спеціальності">Освітні програми і спеціальності</a></li>
<li class="vstup"><a href="https://vstup.kpi.ua/" title="Інформація про вступ в КПІ">Інформація про вступ в КПІ</a></li>
<li class="science"><a href="/science" title="Наукові заходи і дослідження">Наукові заходи і дослідження</a></li>
<li class="practice"><a href="https://robota.kpi.ua" title="Працевлаштування">Працевлаштування</a></li>
<li class="am"><a href="https://mobilnist.kpi.ua/" title="Академічна мобільність">Академічна мобільність</a></li>
<li class="sd"><a href="https://www.sikorsky-distance.org/" title="Sicorsky Distance">Sicorsky Distance</a></li>
<li class="roz"><a href="https://rozklad.kpi.ua" title="Розклад">Розклад</a></li>
<li class="cult"><a href="/culture" title="Культура і творчість в КПІ">Культура і творчість в КПІ</a></li>
<li class="stm"><a href="https://studmisto.kpi.ua" title="Студмісто">Студмістечко</a></li>
<li class="sport"><a href="/sport" title>Спорт і оздоровлення</a></li>
<li class="sss"><a href="https://sc.kpi.ua/" title="Студентська рада КПІ - це ком'юніті згуртованих та ініціативних студентів">Студентська рада КПІ</a></li>
</ul>
</div>
</div>
      
    </div><!--/.block-content -->
</div><!--/.block -->
<div id="block-thex-footertop3" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><div class="region region-projects">
<div class="target-b">
<h2>Корисна інформація</h2>
<ul class="target">
<li class="sh"><a href="/pupil" title="школярам">Корисна інформація для школярів</a></li>
<li class="vs"><a href="https://pk.kpi.ua" title="вступникам">Корисна інформація для вступників</a></li>
<li class="fr"><a href="/freshmen" title="першокурсникам">Корисна інформація першокурсників</a></li>
<li class="st"><a href="/4students" title="студентам">Корисна інформація для студентів</a></li>
<li class="as"><a href="https://aspirantura.kpi.ua" title="аспірантам">Корисна інформація для аспірантів</a></li>
<li class="dc"><a href="https://aspirantura.kpi.ua" title="докторантам">Корисна інформація для докторантів</a></li>
<li class="vy"><a href="/alumnus" title="випускникам">Корисна інформація для випускників</a></li>
<li class="sp"><a href="/tutor" title="викладачам та співробітникам">Корисна інформація для співробітників</a></li>
<li class="is"><a href="https://istudent.kpi.ua" title="іноземним студентам">Корисна інформація для іноземних студентів</a></li>
</ul>
</div>
<div class="projects">
<h2>Проекти та ініціативи<br>КПІ ім.Ігоря Сікорського</h2>
<ul class="project">
<li class="stp"><a href="https://science.kpi.ua/" title="Наукові проекти">Наукові проекти</a></li>
<li class="stgr"><a href="https://dnvr.kpi.ua/s-t-e-a-m/" title="Студентські гуртки">Студентські гуртки</a></li>
</ul>
</div>
<ul class="project">
<li class="sc"><a href="/sikorsky-challenge" title="sikorsky-challenge">Sikorsky Challenge</a></li>
<li class="ec"><a href="http://login.kpi.ua/" title>Електронний кампус</a></li>
<li class="fest"><a href="/opendoors" title="КПІАбітFest">КПІАбітFest</a></li>
<li class="talks"><a href="https://www.youtube.com/watch?v=69w_RQCQzVQ&amp;list=PLj5NByUcaAvYuV3V_GStE1x_-NYV7lEjp" title="KPItalks" target="_blank">KPItalks</a></li>
<li class="fair"><a href="https://careerfair.kpi.ua/" title="Ярмарок вакансій">Ярмарок вакансій</a></li>
<li class="iy"><a href="https://www.youtube.com/watch?v=GNBWP0LjBLI&amp;list=PLj5NByUcaAvb1qx21HPlBEJriX02GwVOH" title="Історії успіху" target="_blank">Історії успіху</a></li>
<li class="sh"><a href="https://www.youtube.com/channel/UC0-PrcKDVWPZYOKInmoS4zQ" title="Школа кар'єрного зростання" target="_blank">Школа кар'єрного зростання</a></li>
<li class="art"><a href="http://technoart.kpi.ua/" title="ФЕСТИВАЛЬ ТЕХНОАРТКПІ">ФЕСТИВАЛЬ ТЕХНОАРТКПІ</a></li>
</ul>
</div></div>
      
    </div><!--/.block-content -->
</div><!--/.block -->

  </div>

    </div><!-- /footer-top-container -->
  </div><!-- /container -->
</footer><!-- /footer-top -->
  <footer class="footer-blocks footer">
  <div class="container">
    <div class="footer-blocks-container">
              <div class="footer-block footer-one">
            <div class="block-region region-footer-one">
    <nav role="navigation" aria-labelledby="block-thex-structure-menu" id="block-thex-structure" class="block block-menu">
      
  <h2 class="block-title" id="block-thex-structure-menu">Структура</h2>
  

        
              <ul class="menu">
                    <li class="menu-item menu-item-level-1">
        <a href="/rectorate" data-drupal-link-system-path="taxonomy/term/354">Ректорат</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://rada.kpi.ua">Вчена рада</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/office" title="Департаменти КПІ ім.Ігоря Сікорського" data-drupal-link-system-path="term/8">Департаменти</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/institutes" title="Інститути КПІ" data-drupal-link-system-path="term/275">Інститути</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/faculties" data-drupal-link-system-path="term/274">Факультети</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/chairs" title="Кафедри КПІ ім.Ігоря Сікорського" data-drupal-link-system-path="term/299">Кафедри</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/subdivisions" data-drupal-link-system-path="term/25">Інші підрозділи та організації</a>
              </li>
        </ul>
  


  </nav>
<nav role="navigation" aria-labelledby="block-thex-organizations-menu" id="block-thex-organizations" class="block block-menu">
      
  <h2 class="block-title" id="block-thex-organizations-menu">Організації</h2>
  

        
              <ul class="menu">
                    <li class="menu-item menu-item-level-1">
        <a href="http://museum.kpi.ua/" title="Державний політехнічний музей ">Державний політехнічний музей</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://kt.kpi.ua/" title="НТО “КПІ-Телеком”">ЦТ “КПІ-Телеком”</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://sport.kpi.ua/" title="ентр фізичного виховання та спорту КПІ ім. Ігоря Сікорського Центр фізичного виховання та спорту КПІ ім. Ігоря Сікорського">Спорткомплекс КПІ</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://studmisto.kpi.ua/" title="Студентське містечко">Студмісто</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="http://uajc.kpi.ua/" title="Українсько-Японський центр КПІ ім. Ігоря Сікорського">Українсько-Японський центр</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://robota.kpi.ua/" title="Вакансії для студентів і випускників КПІ, пропозиції для працедавців">Центр розвитку кар&#039;єри</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://io.kpi.ua/" title="Журнал &quot;Наукові вісті КПІ&quot;">Офіс інновацій</a>
              </li>
        </ul>
  


  </nav>

  </div>

        </div>
                    <div class="footer-block footer-two">
            <div class="block-region region-footer-two">
    <nav role="navigation" aria-labelledby="block-thex-mozhlivostiosviti-menu" id="block-thex-mozhlivostiosviti" class="block block-menu">
      
  <h2 class="block-title" id="block-thex-mozhlivostiosviti-menu">Можливості освіти</h2>
  

        
              <ul class="menu">
                    <li class="menu-item menu-item-level-1">
        <a href="https://www.sikorsky-distance.org/" title="Дистанційне навчання з використанням інформаційних технологій в КПІ">Дистанційна освіта</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="http://mobilnist.kpi.ua/" title="Стипендіальні програми, подвійний диплом, національна мобільність, волонтерство">Академічна мобільність</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="http://ipo.kpi.ua/druga-vishha" title="Друга і наступна вища освіта в КПІ">Перепідготовка (друга вища освіта)</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="http://ipo.kpi.ua/povyshenie_kvalif" title="Підвищення кваліфікації в КПІ">Підвищення кваліфікації</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://istudent.kpi.ua/" title="Центр міжнародної освіти">Центр міжнародної освіти</a>
              </li>
        </ul>
  


  </nav>
<nav role="navigation" aria-labelledby="block-thex-informaciyniresursi-menu" id="block-thex-informaciyniresursi" class="block block-menu">
      
  <h2 class="block-title" id="block-thex-informaciyniresursi-menu">Інформаційні ресурси</h2>
  

        
              <ul class="menu">
                    <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/weblinks" title="Сайти мережі КПІ" data-drupal-link-system-path="node/7921">Сайти мережі КПІ</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/journal" title="Наукові видання НТУУ КПІ ім.Ігоря Сікорського" data-drupal-link-system-path="taxonomy/term/301">Наукові видання</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/u_conferences" title="Наукові конференції КПІ ім.Ігоря Сікорського" data-drupal-link-system-path="term/147">Наукові конференції</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://report.kpi.ua/" title=" Завершені науково-дослідні роботи КПІ ім. Ігоря Сікорського">Науково-дослідні роботи</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="http://ela.kpi.ua/" title="Електронний архів КПІ ім.Ігоря Сікорського">Електронний архів</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/kp" title="Щотижнева газета &quot;Київський політехнік&quot;" data-drupal-link-system-path="taxonomy/term/148">Газета &quot;Київський політехнік&quot;</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="http://scinews.kpi.ua/" title="Міжнародний науково-технічний журнал «KPI Science News»">Журнал &quot;Наукові вісті КПІ&quot;</a>
              </li>
        </ul>
  


  </nav>

  </div>

        </div>
                    <div class="footer-block footer-three">
            <div class="block-region region-footer-three">
    <nav role="navigation" aria-labelledby="block-thex-forusers-menu" id="block-thex-forusers" class="block block-menu">
      
  <h2 class="block-title" id="block-thex-forusers-menu">Для користувачів</h2>
  

        
              <ul class="menu">
                    <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/pupil" title="Інформація для школярів" data-drupal-link-system-path="node/7238">Школярам</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/application" data-drupal-link-system-path="node/2426">Вступникам</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://pk.kpi.ua/donbass-crimea/" title="Абітурієнтам з тимчасово окупованих територій">Особам з тимчасово окупованих територій</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/freshmen" title="Інформація для перщокурсника і його/її батьків та рідних" data-drupal-link-system-path="node/1640">Першокурсникам</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/4students" title="Інформація для студентів" data-drupal-link-system-path="node/7285">Студентам</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/alumnus" data-drupal-link-system-path="node/7239">Випускникам</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/tutor" title="Корисні матеріали для НПП та співробітників університету" data-drupal-link-system-path="node/7127">Співробітникам</a>
              </li>
        </ul>
  


  </nav>
<nav role="navigation" aria-labelledby="block-thex-populyarniposhukovizapiti-menu" id="block-thex-populyarniposhukovizapiti" class="block block-menu">
      
  <h2 class="block-title" id="block-thex-populyarniposhukovizapiti-menu">Актуальне</h2>
  

        
              <ul class="menu">
                    <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/sc" title="Sikorsky Challenge" data-drupal-link-system-path="taxonomy/term/348">Sikorsky Challenge</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/researchers" title="Результати конкурсів «Викладач-дослідник», «Молодий викладач-дослідник» та умови участі" data-drupal-link-system-path="taxonomy/term/161">Викладачі-дослідники</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/academic-integrity" title="Академічна доброчесність в університеті - нормативно-правові та регламентуючі документи" data-drupal-link-system-path="node/18066">Академічна доброчесність</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="https://osvita.kpi.ua/op" title="Спеціальності та освітні програми (спеціалізації) ">Спеціальності та освітні програми</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/war" title="Агресія російської федерації проти України" data-drupal-link-system-path="taxonomy/term/292">Війна</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/clust-space" title="Смартукриття для студентів CLUST SPACE" data-drupal-link-system-path="taxonomy/term/2619">CLUST SPACE</a>
              </li>
                <li class="menu-item menu-item-level-1">
        <a href="/index%2ephp/sustainability/goals" title="Глобальні цілі сталого розвитку" data-drupal-link-system-path="taxonomy/term/1893">Цілі сталого розвитку</a>
              </li>
        </ul>
  


  </nav>

  </div>

        </div>
                    <div class="footer-block footer-four">
            <div class="block-region region-footer-four">
    <div class="views-element-container block" id="block-thex-views-block-slayder-baneri-block-1">
  
    
  <div class="block-content">
      <div><div class="js-view-dom-id-644e2f5244c17dd8d6c2d42eff4266f6f2b7a0b20b7eb5d4527d3f780e72f524">
  
  
  

  
  
  

    <div class="skin-default">
    
    <div id="views_slideshow_cycle_main_slayder_baneri-block_1" class="views_slideshow_cycle_main views_slideshow_main">
    <div id="views_slideshow_cycle_teaser_section_slayder_baneri-block_1"  class="views_slideshow_cycle_teaser_section">
     <div id="views_slideshow_cycle_div_slayder_baneri-block_1_0"  class="views_slideshow_cycle_slide views_slideshow_slide views-row-1 views-row-odd">
      <div  class="views-row views-row-0 views-row-odd views-row-first">
  <div class="views-field views-field-node-image"><div class="field-content"><a href="https://solom.kyivcity.gov.ua/">  <img loading="lazy" src="/files/images-page/solor_0.jpg" width="429" height="206" alt="Солом&#039;янська РДА в місті Києві" title="Солом&#039;янська РДА в місті Києві" class="image-field" />

</a></div></div>
</div>

  </div>

     <div id="views_slideshow_cycle_div_slayder_baneri-block_1_1"  class="views_slideshow_cycle_slide views_slideshow_slide views-row-2 views_slideshow_cycle_hidden views-row-even">
      <div  class="views-row views-row-1 views-row-even views-row-last">
  <div class="views-field views-field-node-image"><div class="field-content"><a href="https://mon.gov.ua">  <img loading="lazy" src="/files/images-page/mon.jpg" width="429" height="206" alt="Банер МОН України" title="Банер МОН України" class="image-field" />

</a></div></div>
</div>

  </div>

  </div>

</div>


        </div>

    

  
  

  
  
</div>
</div>

    </div><!--/.block-content -->
</div><!--/.block -->
<div id="block-thex-banner" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><p><a href="https://prozorro.gov.ua/search/plan?text=02070921" target="_blank"><img src="/files/images-page/prozorro.png" width="100%" alt="Плани закупівель КПІ ім.Ігоря Сікорського"></a></p>
<p><a href="https://publish.kpi.ua/"><img src="/files/publish.jpg" width="100%" title="Видавничий дім КПІ ім. Ігоря Сікорського"></a></p>
<p><a href="https://hotelsikorsky.kpi.ua/"><img src="/files/hotelsikorsky.jpg" width="100%" title="Готель SIKORSKY - це зручний готель біля прекрасного парку"></a></p>
<p><a href="/testdm"><img src="/files/2026-daf.jpg" width="100%" title="Складайте екзамени TestDaF, TestAS, onSET, dMAT у Києві – без зайвих турбот та витрат на подорожі за кордон!"></a></p>
</div>
      
    </div><!--/.block-content -->
</div><!--/.block -->

  </div>

        </div>
          </div><!-- /footer-top-container -->
  </div><!-- /container -->
</footer><!-- /footer-top -->
  <footer class="footer-bottom-blocks footer">
  <div class="container">
    <div class="footer-bottom-blocks-container">
                    <div class="footer-bottom-block footer-bottom-block-right">
            <div class="block-region region-footer-bottom-right">
    <div id="block-thex-footermessage" class="block">
  
    
  <div class="block-content">
      
            <div class="field field--name-body field--type-text-with-summary field--label-hidden field-item"><p><span itemscope itemtype="http://schema.org/EducationalOrganization"><span itemprop="name"><a href="https://kpi.ua">Національний технічний університет України "Київський політехнічний інститут імені Ігоря Сікорського"</a></span><br><br>
<a href="/contact">Контактні дані</a> &nbsp; <a href="/information" alt="Інформація, яка підлягає обов'язковому оприлюдненню">Обов'язкова інформація</a> &nbsp; <a href="https://brand.kpi.ua/">Бренд КПІ</a> &nbsp; <a href="/site">Про сайт</a> &nbsp; <a href="/sitemap">Карта сайту</a>  &nbsp; <a href="/node">Нові матеріали</a><br><span itemprop="address" itemscope itemtype="http://schema.org/PostalAddress"><a href="/location">Адреса: <span itemprop="addressRegion">Україна</span>,  <span itemprop="postalCode">03056</span>, м.<span itemprop="addressLocality">Київ</span>-56, <span itemprop="streetAddress">Берестейський проспект (Перемоги), 37</span> / Мапа кампусу</a>, <a href="mailto:mail@kpi.ua" title="університетська поштова скринька">📧</a> <br></span></span></p>
<p>Телефон загального відділу: <span itemprop="telephone">+38 044 204 8282</span>, довiдка телефонної станцiї: <span itemprop="telephone">+38 044 204 9494</span>, <a href="https://contact.kpi.ua/">телефонний довідник</a><br><br>
Контакти Департаменту безпеки: +38 044 204 2071, +38 097 373 5387, <a href="https://t.me/dbkpi_bot" title="Телеграм-бот для швидкого реагування на звернення студентів">Бот швидкого реагування</a><br>
<br><br>
⚠️ <a href="/2022-emergency-procedure">Порядок дій в умовах загрози надзвичайної ситуації</a><br>💡 <a href="/points-of-invincibility">Пункти незламності</a><br>🔥 Слідкуйте за повідомленнями в <a href="https://t.me/rektoratkpi">телеграм-каналі «Ректорат»</a><br>
<br><br>
1998-2026 ©  Вдячні за ваші <a href="mailto:lab-inform@kpi.ua">пропозиції і зауваження стосовно сайту 📧</a></p>
</div>
      
    </div><!--/.block-content -->
</div><!--/.block -->

  </div>

        </div>
          </div><!-- /footer-bottom-blocks-container -->
  </div><!-- /container -->
</footer><!-- /footer-bottom-blocks -->
<footer class="footer-bottom footer">
  <div class="container">
    <div class="footer-bottom-container">
      <div class="website-copyright">
        &copy; 2026 КПІ ім. Ігоря Сікорського, All rights reserved.
      </div>
          </div><!-- /footer-bottom-container -->
  </div><!-- /container -->
</footer><!-- /footer-bottom -->
  <div class="scrolltop"><i class="icon-arrow-up"></i></div>

  </div>

    
    <script type="application/json" data-drupal-selector="drupal-settings-json">{"path":{"baseUrl":"\/","pathPrefix":"","currentPath":"home","currentPathIsAdmin":false,"isFront":true,"currentLanguage":"UK"},"pluralDelimiter":"\u0003","suppressDeprecationErrors":true,"viewsSlideshowCycle":{"#views_slideshow_cycle_main_10_slider_main-block_1":{"num_divs":5,"id_prefix":"#views_slideshow_cycle_main_","div_prefix":"#views_slideshow_cycle_div_","vss_id":"10_slider_main-block_1","effect":"fade","transition_advanced":0,"timeout":5000,"speed":700,"delay":0,"sync":1,"random":0,"pause":1,"pause_on_click":0,"action_advanced":0,"start_paused":0,"remember_slide":0,"remember_slide_days":1,"pause_in_middle":0,"pause_when_hidden":0,"pause_when_hidden_type":"full","amount_allowed_visible":"","nowrap":0,"fixed_height":1,"items_per_slide":1,"items_per_slide_first":0,"items_per_slide_first_number":1,"wait_for_image_load":1,"wait_for_image_load_timeout":3000,"cleartype":0,"cleartypenobg":0,"advanced_options":"{}","advanced_options_choices":0,"advanced_options_entry":""},"#views_slideshow_cycle_main_slayder_baneri-block_1":{"num_divs":2,"id_prefix":"#views_slideshow_cycle_main_","div_prefix":"#views_slideshow_cycle_div_","vss_id":"slayder_baneri-block_1","effect":"fade","transition_advanced":1,"timeout":5000,"speed":700,"delay":0,"sync":1,"random":0,"pause":1,"pause_on_click":0,"action_advanced":0,"start_paused":0,"remember_slide":0,"remember_slide_days":1,"pause_in_middle":0,"pause_when_hidden":0,"pause_when_hidden_type":"full","amount_allowed_visible":"","nowrap":0,"fixed_height":1,"items_per_slide":1,"items_per_slide_first":0,"items_per_slide_first_number":1,"wait_for_image_load":1,"wait_for_image_load_timeout":3000,"cleartype":0,"cleartypenobg":0,"advanced_options":"{}","advanced_options_choices":0,"advanced_options_entry":""}},"viewsSlideshow":{"10_slider_main-block_1":{"methods":{"goToSlide":["viewsSlideshowSlideCounter","viewsSlideshowPager","viewsSlideshowCycle"],"nextSlide":["viewsSlideshowSlideCounter","viewsSlideshowPager","viewsSlideshowCycle"],"pause":["viewsSlideshowControls","viewsSlideshowCycle"],"play":["viewsSlideshowControls","viewsSlideshowCycle"],"previousSlide":["viewsSlideshowSlideCounter","viewsSlideshowPager","viewsSlideshowCycle"],"transitionBegin":["viewsSlideshowSlideCounter","viewsSlideshowPager"],"transitionEnd":[]},"paused":0},"slayder_baneri-block_1":{"methods":{"goToSlide":["viewsSlideshowSlideCounter","viewsSlideshowPager","viewsSlideshowCycle"],"nextSlide":["viewsSlideshowSlideCounter","viewsSlideshowPager","viewsSlideshowCycle"],"pause":["viewsSlideshowControls","viewsSlideshowCycle"],"play":["viewsSlideshowControls","viewsSlideshowCycle"],"previousSlide":["viewsSlideshowSlideCounter","viewsSlideshowPager","viewsSlideshowCycle"],"transitionBegin":["viewsSlideshowSlideCounter","viewsSlideshowPager"],"transitionEnd":[]},"paused":0}},"superfish":{"superfish-main":{"id":"superfish-main","sf":{"animation":{"opacity":"show","height":"show"},"speed":"fast","autoArrows":false,"dropShadows":false},"plugins":{"smallscreen":{"mode":"window_width","breakpoint":1092,"expandText":"\u0420\u043e\u0437\u043a\u0440\u0438\u0442\u0438","collapseText":"\u0417\u0433\u043e\u0440\u043d\u0443\u0442\u0438","title":"\u0413\u043e\u043b\u043e\u0432\u043d\u0435 \u043c\u0435\u043d\u044e"},"supposition":true,"supersubs":true}}},"user":{"uid":0,"permissionsHash":"50e95c2192d024699f431630984132c1a547f4629a81f6b57c5e460b141b9df9"}}</script>
<script src="/files/js/js_LEP0NOZ74dmXkq8qkpZrKv8HzEPub1nQV3VGGHg3D-A.js?scope=footer&amp;delta=0&amp;language=UK&amp;theme=thex&amp;include=eJx1jlEOgzAMQy_EVGm32CkqKKHNlCWsLjBuPzHEDyt_tp9lGdNIeWAkh0M1-M98spkyayEtVY5XK4KQibTON4WpwxUdDVzYtCmJPi6Kda3cUFZhjc3MtMBDuCckW9zzPVFe91OP_dS5snAfqXjWwc7MhzUIHSM_c1WB6f2CVdMvmjSIMg"></script>

  </body>
</html>

```

---

## Частина B. Розбір полів заголовка

Розбирається відповідь, отримана в завданні A.1.

**Загальна кількість полів заголовка у відповіді:** 7

| № | Поле заголовка | Значення | Призначення (власне формулювання) | Походження: сервер / проміжний вузол / не визначено | Обґрунтування |
|---|---|---|---|---|---|
| 1 | Server | nginx | Вказує програмне забезпечення, яке обробило HTTP-запит. | Не визначено | nginx може працювати як вебсервер або як проміжний вузол. |
| 2 | Date | Thu, 19 Nov 2026 12:16:10 GMT | Вказує дату та час формування відповіді. | Не визначено | За отриманим виводом неможливо встановити, який саме вузол сформував це поле. |
| 3 | Content-Type | text/html | Вказує тип вмісту отриманої відповіді. | Не визначено | Поле описує HTML-сторінку, але не дозволяє встановити її джерело. |
| 4 | Content-Length | 162 | Вказує розмір тіла відповіді в байтах. | Не визначено | Значення характеризує обсяг отриманих даних, але не визначає походження поля. |
| 5 | Connection | close | Повідомляє про закриття з'єднання після передавання відповіді. | Не визначено | Поле стосується поточного з'єднання, але роль вузла невідома. |
| 6 | Location | https://kpi.ua/ | Вказує адресу, на яку перенаправляється клієнт. | Не визначено | Перенаправлення може виконуватися як кінцевим сервером, так і проміжним вузлом. |
| 7 | Strict-Transport-Security | max-age=31536000; includeSubDomains; preload | Повідомляє про політику використання захищеного HTTPS-з'єднання. | Не визначено | Неможливо встановити, на якому вузлі налаштовано цю політику. |

> Рядок наводять на кожне поле, яке реально надійшло. Зайві рядки вилучають, за потреби додають нові. Поле, походження якого встановити не вдалося, зазначають із позначкою «не визначено» та поясненням утруднення.

---

## Частина D. Висновки

Обсяг — 150–300 слів. Висновки спираються на власні спостереження.

**D.1.** Що з поведінки сервера виявилося неочевидним або несподіваним. Конкретно, з посиланням на рядок виводу.

Під час виконання практичної роботи для мене виявилося несподіваним те, що сервер прийняв запити із чужим та навіть не існуючим доменним іменем у полі Host. У завданні А.3.2 було отримано відповідь `HTTP/1.1 301 Moved Permanently` із заголовком `Location: https://opism-pr02.invalid/`. Це свідчить про те, що сервер формує адресу перенаправлення на основі отриманого значення поля Host. Також цікавим виявилося те, що у версії HTTP/1.0 сервер прийняв запит без поля Host, тоді як у версії HTTP/1.1 повернув помилку `400 Bad Request`.

**D.2.** Яке з полів заголовка викликало найбільше утруднення при визначенні походження (частина B) та з якої причини.

Найбільше утруднення викликало визначення походження поля `Server: nginx`. Це поле вказує на програмне забезпечення, яке обробляє HTTP-запити, однак nginx може виконувати функції як вебсервера, так і проміжного вузла. Отримані експериментальні дані не дозволяють однозначно встановити, яку саме функцію він виконує в цьому випадку. Саме тому в таблиці частини B походження цього поля зазначено як «не визначено».

**D.3.** Яке питання залишилося без відповіді після виконання роботи.

Після виконання практичної роботи залишилося питання, чому сервер приймає запит із неіснуючим доменним іменем у полі Host та формує відповідну адресу перенаправлення, не перевіряючи її доступність. Також хотілося б детальніше дослідити, за якими ознаками можна визначити наявність проміжних вузлів між клієнтом і кінцевим сервером, якщо отримані поля заголовка не містять однозначної інформації про їх походження.

---

## Контрольні питання

**1.** У завданні A.1 сервер не надсилав відповіді, доки не було введено порожній рядок. Чим це зумовлено?

Це зумовлено структурою HTTP-запиту. Порожній рядок позначає завершення заголовків запиту та відокремлює їх від тіла повідомлення. До отримання цього рядка сервер очікує продовження заголовків, оскільки запит ще не завершено. Після введення порожнього рядка сервер обробляє запит і надсилає відповідь.

**2.** Порівняйте результати завдань A.1, A.2 та A.3.1–A.3.3 (таблиця Додатка Д). За яких значень поля `Host` і за якої версії протоколу сервер обслуговує запит, а за яких — ні? Яку задачу розв'язує поле `Host`? Відповідь має посилатися на конкретні рядки ваших виводів.

У завданнях А.1, А.3.1 та А.3.2 сервер повернув відповідь `301 Moved Permanently` незалежно від того, чи було в полі Host зазначено власний, чужий або неіснуючий домен. При цьому значення поля Location змінювалося відповідно до значення Host.
У завданні А.2 за відсутності поля Host у версії HTTP/1.1 сервер повернув помилку `400 Bad Request`. У завданні А.3.3 за відсутності цього поля у версії HTTP/1.0 сервер повернув `301 Moved Permanently` із заголовком `Location: https://_/`.
Поле Host дозволяє серверу визначити, до якого доменного імені звертається клієнт. Це особливо важливо, коли на одній IP-адресі розміщено декілька вебсайтів.

**3.** Скільки відповідей надійшло у завданні A.4 і з якими кодами стану? Чи залежить відповідь сервера на порту 80 від запитаного шляху — і що це говорить про роль цього сервера? Якщо надійшла одна відповідь, знайдіть у ній поле заголовка, яке це пояснює, або зазначте, що такого поля немає. Якщо надійшло дві — що це означає для клієнтської програми, яка завантажує сторінку з великою кількістю вкладених ресурсів?

У завданні А.4 отримано дві відповіді з однаковим кодом стану `301 Moved Permanently`. Код стану не залежав від запитаного шляху, однак значення поля Location змінювалося відповідно до нього. Це свідчить про те, що сервер на порту 80 виконує перенаправлення HTTP-запитів на HTTPS.
У першій відповіді зазначено `Connection: keep-alive`, а в другій — `Connection: close`. Отримання двох відповідей в одному з'єднанні свідчить про можливість передавання кількох запитів без повторного встановлення TCP-з'єднання. Це дозволяє клієнтській програмі зменшити витрати часу на завантаження вебсторінок із великою кількістю вкладених ресурсів.

**4.** Які поля заголовка програма `curl` додала самостійно (завдання A.5)? Ці поля не є обов'язковими — сервер відповів і без них у завданні A.1. З якою метою їх додано?

У завданні А.5 програма curl автоматично додала два необов'язкові поля заголовка: `User-Agent: curl/8.5.0` та `Accept: */*`.
Поле User-Agent повідомляє серверу назву та версію клієнтської програми, яка надсилає запит. Поле Accept зі значенням */* вказує на готовність клієнта приймати дані будь-якого типу.
Ці поля не є обов'язковими, що підтверджується результатом завдання А.1, у якому сервер успішно відповів на запит без них.

**5.** За якими ознаками у вашому виводі виявляється присутність проміжного вузла? Якщо таких ознак не виявлено, поясніть, що з цього випливає.

В отриманих результатах не виявлено однозначних ознак присутності проміжного вузла. Поле `Server: nginx` може свідчити про використання як вебсервера, так і проксі-сервера, однак цього недостатньо для визначення його ролі.
Відсутність явних ознак проміжного вузла не означає, що він відсутній. Це лише свідчить про те, що на основі отриманих експериментальних даних неможливо достовірно встановити його наявність.

**6.** Три рядки, про які не йшлося на лекції, наведено в **Додатку В**.

---

## Додаток В. Відповіді на питання 6

| № | Рядок виводу | Джерело (номер завдання) |
|---|---|---|
| 1 | `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload` | А.1 |
| 2 | `X-Content-Type-Options: nosniff` | А.6 |
| 3 | `X-Frame-Options: SAMEORIGIN` | А.6 |

---

## Додаток Д. Зведення результатів завдання A.3

**Вузол, з яким установлювалося з'єднання (у всіх пробах однаковий):** kpi.ua, порт 80

| Проба | Значення поля `Host` | Версія | Код стану | Обсяг тіла відповіді | Збігається з A.1 (так / ні) |
|---|---|---|---|---|---|
| A.1 (вихідна) | kpi.ua | 1.1 | 301 | 162 байти | — |
| A.2 | поле відсутнє | 1.1 | 400 | 150 байтів | Ні |
| A.3.1 | openssl.org | 1.1 | 301 | 162 байти | Ні |
| A.3.2 | opism-pr02.invalid | 1.1 | 301 | 162 байти | Ні|
| A.3.3 | поле відсутнє | 1.0 | 301 | 162 байти | Ні |

**Висновок за таблицею (2–4 речення):** що саме змінювалося у запиті від проби до проби і як на це реагував сервер.

Під час виконання завдань змінювалися значення поля Host, його наявність та версія протоколу HTTP. За відсутності поля Host у версії HTTP/1.1 сервер повернув помилку 400, тоді як в інших випадках було отримано код 301. Значення поля Location змінювалося залежно від зазначеного доменного імені. Отримані результати свідчать про важливість поля Host для обробки HTTP-запитів та формування адреси перенаправлення.

---

## Декларування використання технологій штучного інтелекту

Для цієї роботи встановлено **рівень Р3 — ШІ як співвиконавець**.

Виводи команд частини A не можуть бути згенеровані та мають бути отримані внаслідок фактичного виконання команд.

**Чи використовувалися технології ШІ під час виконання роботи:** так 

| № | Інструмент (назва, версія) | Етап роботи | Дослівний текст запиту (промпту) | Як використано результат |
|---|---|---|---|---|
| 1 | ChatGPT (GPT-6) | Виконання експериментальної частини | виконуємо 2 | Уточнено послідовність виконання завдань та синтаксис команд. |
| 2 | ChatGPT (GPT-6) | Аналіз HTTP-відповідей | Скриншоти результатів експериментів (без текстового запиту) | Отримано пояснення щодо кодів стану та особливостей полів HTTP-заголовка. |
| 3 | ChatGPT (GPT-6) | Контрольні питання та висновки | 5; 6; далі | Отримано допомогу з аналізом експериментальних даних і формулюванням висновків. |

Усі команди виконувалися особисто в середовищі Killercoda Ubuntu Playground. Штучний інтелект використовувався для пояснення отриманих результатів, оформлення звіту та формулювання висновків і відповідей на контрольні питання.

**Підтвердження:** усі виводи команд, наведені в частині A, отримано внаслідок фактичного виконання команд на зазначеному індивідуальному домені.

---

*ОПІСМ (ОК-13) · Практична робота № 2 · бланк звіту*
