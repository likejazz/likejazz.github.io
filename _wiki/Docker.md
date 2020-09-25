---
layout: wiki 
title: Docker
last-modified: 2020/09/24 20:56:05
---

<!-- TOC -->

- [기본](#기본)
    - [Dockerfile](#dockerfile)
    - [명령](#명령)
    - [Apache Hello World](#apache-hello-world)
    - [스크립트](#스크립트)
    - [Push Docker Image to ECR](#push-docker-image-to-ecr)
    - [Push the Docker image to Container Registry](#push-the-docker-image-to-container-registry)
- [CMD vs. ENTRYPOINT](#cmd-vs-entrypoint)
    - [Keep Docker Containers Running](#keep-docker-containers-running)
- [Books](#books)
    - [도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문 <sub>2018, 2019</sub>](#도커쿠버네티스를-활용한-컨테이너-개발-실전-입문-2018-2019)

<!-- /TOC -->

# 기본
## Dockerfile
```docker
FROM hayd/ubuntu-deno:1.3.0
MAINTAINER skpark1224@hyundai.com

WORKDIR /www
CMD deno run --allow-net --allow-read hello.ts
```

Go는 빌드 후에 바이너리만 별도로 담을 수 있기 때문에[^fn-go] multi-stage builds를 사용하면 docker를 위한 최적의 언어 같다.

[^fn-go]: <https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/tree/master/hello-app>

```docker
FROM golang:1.8-alpine
ADD . /go/src/hello-app
RUN go install hello-app

FROM alpine:latest
COPY --from=0 /go/bin/hello-app .
ENV PORT 8080
CMD ["./hello-app"]
```
## 명령
```console
# Stop & Remove
docker stop aas-www-container
docker rm aas-www-container

# Build
docker build -t aas-www-image .

# Run
docker run -d --name aas-www-container -p 80:8123 -v /home/gcp-user/www:/www aas-www-image
docker logs -f aas-www-container
```

## Apache Hello World
```docker
FROM ubuntu:18.04

# Install dependencies
RUN apt-get update && \
 apt-get -y install apache2

# Install apache and write hello world message
RUN echo 'Hello World!' > /var/www/html/index.html

# Configure apache
RUN echo '. /etc/apache2/envvars' > /root/run_apache.sh && \
 echo 'mkdir -p /var/run/apache2' >> /root/run_apache.sh && \
 echo 'mkdir -p /var/lock/apache2' >> /root/run_apache.sh && \ 
 echo '/usr/sbin/apache2 -D FOREGROUND' >> /root/run_apache.sh && \ 
 chmod 755 /root/run_apache.sh

EXPOSE 80

CMD /root/run_apache.sh
```

## 스크립트
Gist 정리
- [update-docker.sh](https://gist.github.com/likejazz/85c54f4c6b69e60cb7b75f806659153d)  
업데이트가 있을 경우 자동으로 이미지를 받아서 교체
- [dockerize.sh](https://gist.github.com/likejazz/ba41d83fc94dbb75b982f4e37dc008b6)  
도커 이미지를 빌드해서 배포

## Push Docker Image to ECR
```bash
readonly VERSION=$(date '+%y.%m.%d')
readonly ECR_REGISTRY=0996xxx.dkr.ecr.ap-northeast-2.amazonaws.com/edith/edith-xxxx
readonly AWS_OTP=$(aws ecr get-login-password --region ap-northeast-2)

echo "${AWS_OTP}" | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker tag edith/edith-xxxx:latest ${ECR_REGISTRY}:"${VERSION}"
docker push ${ECR_REGISTRY}:"${VERSION}"
```

## Push the Docker image to Container Registry
```bash
$ gcloud auth configure-docker
$ docker push gcr.io/edith-xxx/hello-app:v1
```
GCP는 docker login 필요 없이 gcloud 인증으로 바로 진행된다.

# CMD vs. ENTRYPOINT
항상 헷갈린다. 주로 `CMD`로 처리했으며, `docker run` 이후에 실행된다.

- `CMD`: when container runs with a command, e.g., `docker run -it <image> /bin/bash`, CMD is ignored and bash interpreter runs instead.
- `ENTRYPOINT`: Shell form of ENTRYPOINT ignores any CMD or docker run command line arguments.

[RUN, CMD, ENTRYPOINT의 관계를 잘 정리한 글](http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/)

```
# Dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/cat"]
```
and
```
$ docker build -t=cat .
```
then you can see:
```
$ docker run cat /etc/passwd
#                ^^^^^^^^^^^
#                    CMD
#            ^^^      
#            image (tag)- using the default ENTRYPOINT
```
[Link](https://stackoverflow.com/a/21558992/3513266)

아래는 php 어플리케이션을 구동하기 위해 몇 가지 수정한 Dockerfile이다.
```dockerfile
FROM php:7-apache
LABEL maintainer="kaon.park@xxx"

# 기본 포트가 80으로 설정되어 있어 sed로 교체한다.
ENV CONTAINER_PORT 18080
EXPOSE 18080
RUN sed -i "s/80/$CONTAINER_PORT/g" /etc/apache2/sites-available/000-default.conf /etc/apache2/ports.conf

# CMD is the command the container executes by default when you launch the built image.
# A Dockerfile can only have one CMD.
# CMD docker-php-entrypoint apache2-foreground

# 개발시에는 VOLUME 설정을 하고 실서비스는 파일을 복사한다.
COPY ./demo/index.php /var/www/html/
```

## Keep Docker Containers Running
```
FROM amazonlinux:latest
...
CMD tail -f /dev/null
```

# Books
## 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문 <sub>2018, 2019</sub>
- 도커 컨테이너 배포
- ~~스웜을 이용한 실전 애플리케이션 개발~~
    - 필요 없음
- 쿠버네티스 입문/클러스터 구축  
    - GKE에서 진행
- 쿠버네티스 실전
    - kubectl, helm등 다양한 활용 방법 소개, rolling update 전략, service mesh Istio 언급.
- 컨테이너 운영
    - log 관리, GCP의 stackdriver logging 소개(현재는 Cloud Logging)
- 가벼운 도커 이미지 만들기
    - scratch, busybox, alpine 까지 소개한다.

중간중간 컬럼을 통해 저자의 생각을 읽을 수 있어서 좋다.