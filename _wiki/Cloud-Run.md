---
layout: wiki 
title: Cloud Run
tags: ["Cloud"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [Cloud Build](#cloud-build)
- [Container Registry](#container-registry)
- [기타](#기타)
    - [인증](#인증)
    - [배포](#배포)
    - [`/bin/sh` 에러](#binsh-에러)
    - [Custom Domain](#custom-domain)
    - [Serverless VPC Connector](#serverless-vpc-connector)

<!-- /TOC -->

가이드[^fn-guide]에 따라 진행. Dockerfile을 만드는데 까지는 동일하다.

[^fn-guide]: <https://cloud.google.com/run/docs/quickstarts/build-and-deploy>

# Cloud Build
그러나, 가이드에서는 로컬에서 docker를 build 하지 않고 다음과 같이 Cloud Build에 등록하고 배포한다. 이 경우 소스코드가 tgz로 압축되어 GCS에 올라가고 빌드 서버에서 빌드된 후에 Container Registry에 자동으로 올라간다. 이렇게 하지 않을 경우 직접 빌드해서 `docker push`로 직접 올려야 한다.

```console
# 개인 계정으로 로그인
$ gcloud auth login

# 설정
$ gcloud config set run/platform managed
$ gcloud config set run/region asia-northeast3

# 빌드 & 배포
$ gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld
$ gcloud run deploy --image gcr.io/PROJECT_ID/helloworld
```

전송된 Dockerfile을 Kaniko로 빌드한다. `--tag` 지정으로 `docker build`가 수행된다. Cloud Build will run a remote `docker build -t $TAG .`[^fn-help]

[^fn-help]: `gcloud builds submit --help`

- `gcr.io`는 이미지를 미국의 데이터 센터에서 호스팅한다.
- `us.gcr.io`는 이미지를 미국의 데이터 센터에서, `gcr.io`에서 호스팅하는 이미지와 별도의 스토리지 버킷에 호스팅한다.
- `eu.gcr.io`는 유럽 연합 내 이미지를 호스팅한다.
- `asia.gcr.io`는 이미지를 아시아의 데이터 센터에서 호스팅한다.

# Container Registry

Container Registry에 있는 이미지는 GCE, GKE, Cloud Run에 각각 편리하게 선택하여 배포 가능하다. GCE는 모든 포트가 맵핑되어 있으며(맥에서는 some reasons로 docker에서 불가[^fn-some]), GKE는 직접 K8s 기본 클러스터까지 생성해준다.

[^fn-some]: <https://stackoverflow.com/questions/49323225/expose-all-ports-for-a-docker-image/49323975#comment114361345_49323975>

# 기타
## 인증

인증을 걸어두면,
```
$ curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" SERVICE_URL
```
이 형태로 호출이 가능하다. 매 번 토큰을 받는건 시간이 걸리므로 한 번만 받아와서 얼마간 사용도 가능하다.

## 배포

80으로 접속할 수 있고 별도 dns까지 제공한다.  
<https://helloflask-qdg4vdmju1-du.a.run.app/> 이런 형태

## `/bin/sh` 에러

예전에는 `flask`를 실행하면 `/bin/sh`을 찾을 수 없다는 에러가 발생했는데 내부에서 쉘 스크립트를 구동하는 모든 작업이 실행되지 않는듯 했다. 지금은 문제 없다.

```dockerfile
FROM python:3.9-alpine

# We copy just the requirements.txt first to leverage Docker cache
COPY ./requirements.txt /app/requirements.txt

RUN apk add --update netcat-openbsd && rm -rf /var/cache/apk/*

WORKDIR /app

RUN pip3 install -r requirements.txt

# We copy whole directory for easy maintenance.
COPY . /app

ENV AUTHLIB_INSECURE_TRANSPORT=1
ENV FLASK_DEBUG=1

CMD /usr/local/bin/flask run --host='0.0.0.0' --port=$PORT
```

[^fn-workaround]: <https://stackoverflow.com/a/64084917>

Flask를 그대로 실행해도 실행은 되지만 일정 시간이 지난 후,
```
Error: Server Error
The server encountered an error and could not complete your request.
Please try again in 30 seconds.
```
이런 오류가 발생했다. 아마도 리소스 부족으로 인한 오류로 보이는데 로그를 뒤져도 특별한게 없고 원인은 찾지 못했다.

## Custom Domain

Cloud Domains 또는 AWS Route53에서 발급한 도메인을 활용해 verify domain 인증을 거친 후 CNAME으로 연동 가능하다. 다만 아직 Seoul은 지원하지 않고, Tokyo로 배포하여 연동했는데 속도가 2배 늦다. flask hello world가 Seoul은 0.3s, Tokyo는 0.6s가 소요된다.

## Serverless VPC Connector

VPC 연동을 하면 해당 VPC 대역에 속한 서버와 통신이 가능하다. Cloud Run은 외부 IP로 동작하지만 VPC가 Cloud VPN을 이용해 on-prem 통신이 된다면 Cloud Run이 on-prem 시스템과 직접 통신이 가능하다. 또한 Ingress를 Allow internal traffic only로 설정하면 해당 VPC 대역 내에서만 Cloud Run 접속이 가능하고 이외에는 403 오류가 뜬다.
