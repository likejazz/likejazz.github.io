---
layout: wiki 
title: FastCGI
tags: ["Network Programming"]
last_modified_at: 2024/05/04 12:21:50
---

<!-- TOC -->

- [CGI](#cgi)
- [WSGI](#wsgi)
  - [ASGI](#asgi)
  - [Gunicorn](#gunicorn)
  - [FastCGI](#fastcgi)
    - [How to connect to PHP-FPM directly](#how-to-connect-to-php-fpm-directly)

<!-- /TOC -->

# CGI
"one new process per request" model makes CGI programs very simple to implement, but limits efficiency and scalability.

# WSGI
[^fn-wsgi]A WSGI server wraps a back-end process using one or more protocols:

- FastCGI: calling a server process.
- SCGI: a simpler FastCGI  
a fast and simple interface between web servers and application servers. It is similar to FastCGI but is designed to be easier to implement. SCGI는 이제 거의 쓰이지 않는것 같다.
- AJP: a Java FastCGI seems to be a simple and fast binary protocol. It is used in Tomcat, Jetty, and more. WSGI는 프로토콜은 정의하지 않기 때문에 대부분 HTTP, 일부는 Unix Socket을 사용하나 AJP는 바이너리 프로토콜이다.
- mod_python: runs pre-loaded code per request - uses lots of RAM.

[^fn-wsgi]: <https://stackoverflow.com/a/29949739>

SCGI is a language-neutral means of connecting a front-end web server and a web application. WSGI is a Python-specific interface standard for web applications.

WSGI 구현으로 uWSGI와 Gunicorn이 있다.

## ASGI
In 2019, WSGI was superseded by ASGI (Asynchronous Server Gateway Interface), used by frameworks like FastAPI on servers like Uvicorn, which is much faster.

## Gunicorn
AWS의 ELB를 사용하고 flask로 개발, gunicorn으로 바로 연동할 수 있다. static files를 서빙하지 않는다면 nginx를 굳이 사용하지 않아도 된다. WSGI PEP 3333을 보면 Python programmatic API definition이며, flask는 이미 대응하고 있다.

```bash
$ GUNICORN_CMD_ARGS=" \
  --bind=0.0.0.0:8860 \
  --workers=4 \
  --worker-class gevent \
  --reload \
  --reload-engine poll \
  --reload-extra-file templates/ \
  --access-logfile ../logs/access.log \
  --access-logformat '%({X-Forwarded-For}i)s %(l)s %(u)s %(t)s \"%(r)s\" %(s)s %(b)s \"%(f)s\" \"%(a)s\"' \
  --error-logfile ../logs/error.log \
" \
gunicorn serve-www:app
```

기존에 flask로는 다음과 같이 구동했다. 파이썬 코드상에서 따로 `app.run()` 하지 않고 CLI에서 구동:

```bash
$ FLASK_DEBUG=1 \
python -m flask \
  --app serve-www run \
  --host 0.0.0.0 \
  --port 8860
```

## FastCGI
1990년대 중반 [Mark Brown](https://www.linkedin.com/in/mark-brown-32a01b11/)이 Open Market(보스턴의 eCommerce 스타트업이었음) 재직시 개발. 원래 NSAPI 대응. 이즈음 Apache 모듈이 등장하며 mod_perl, mod_php 등과 경쟁.

httpd.conf에서 mod_php 설정 예제:
```
LoadModule php_module /usr/local/opt/php/lib/httpd/modules/libphp.so

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

FastCGI는 별도 프로세스에서 항상 구동되며 일종의 WAS처럼 동작한다.

### How to connect to PHP-FPM directly
```
$ brew install php fcgi
```
리눅스와 달리 맥에서는 저기에 `php-fpm`과 `cgi-fcgi`가 모두 설치된다. www 디렉토리에서 `php-fpm` 실행 후,
```
$ SCRIPT_FILENAME=index.php \
REQUEST_METHOD=GET \
cgi-fcgi -bind -connect 127.0.0.1:9000
```
CLI에서 FastCGI 동작 여부를 확인할 수 있다.

Debian 계열에서는 `apt-get install libfcgi0ldbl`으로 가능하다고 하나 테스트 해보지 않음. CentOS 계열(Amazon Linux 2 포함)에서는 yum에서 패키지를 찾을 수 없었다.
