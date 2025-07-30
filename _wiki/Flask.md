---
layout: wiki 
title: Flask
tags:  ["Languages & Framework"]
last_modified_at: 2024/02/02 14:19:01
---

<!-- TOC -->

- [설정](#설정)
  - [두 번 실행](#두-번-실행)
  - [Docker](#docker)
- [실행](#실행)
- [`runme.sh`](#runmesh)

<!-- /TOC -->

# 설정
Flask의 가장 까다로운 점 중 하나는 devel/prod 설정이 어렵다는 점이다. 분명 제대로 한 것 같은데 적용되지 않는 경우가 많다. 이유는 `flask run`으로 실행시 이미, 
```
* Environment: production
```
으로 구동된 이후이기 때문에 이후에 아무리 `app.config.update`로 `FLASK_ENV` 설정을 해도 반영되지 않는다. 가능한 방법은, 
```
$ FLASK_ENV=development flask run
```
로 시작하는 것이다.

## 두 번 실행
`development` 환경일때는 두 번씩 실행되는 경우가 있어 까다롭다. Why does running the Flask dev server run itself twice?[^fn-flask]

[^fn-flask]: <https://stackoverflow.com/q/25504149>

`FLASK_DEBUG=1`만 해두어도 debugger가 동작하여 auto reloading은 동작한다.

## Docker
docker 내에서 flask를 구동할때는 0.0.0.0으로 해야 한다. 마찬가지로 환경 설정으로는 이미 구동된 상태에서 적용되지 않는 경우가 있기 때문에 CLI에서 미리 지정한다.

```console
$ flask run --host='0.0.0.0'
```

# 실행
Dockerfile
```dockerfile
FROM ubuntu:latest

MAINTAINER Sang Park <likejazz@gmail.com>

RUN apt update -y && \
	apt install -y python3 python3-pip

# We copy just the requirements.txt first to leverage Docker cache
COPY ./requirements.txt /app/requirements.txt

WORKDIR /app

RUN pip3 install -r requirements.txt

# We copy whole directory for easy maintenance.
COPY . /app

ENTRYPOINT AUTHLIB_INSECURE_TRANSPORT=1 FLASK_DEBUG=1 flask run --host='0.0.0.0'
```

로컬에서 실행할때 다음과 같이 runme.sh를 작성하여 매 번 빌드하는 형태로 활용.
```bash
#!/bin/bash

# Stop & Remove
docker stop oauth2-container
docker rm oauth2-container

# Build
docker build -t oauth2 .

# Run
docker run -d \
  --name oauth2-container \
  -p 5001:5000 \
  -v "${PWD}":/app \
  -e PYTHONUNBUFFERED=1 \
  oauth2
docker logs -f oauth2-container
```

# `runme.sh`
```shell
GUNICORN_CMD_ARGS=" \
  --bind=0.0.0.0:8888 \
  --workers=2 \
  --worker-class gevent \
  --reload \
  --reload-engine poll \
  --access-logformat '%({X-Forwarded-For}i)s %(l)s %(u)s %(t)s \"%(r)s\" %(s)s %(b)s \"%(f)s\" \"%(a)s\"' \
" \
gunicorn main:app
```

flask, gunicorn, gevent
