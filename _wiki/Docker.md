---
layout: wiki 
title: Docker
last-modified: 2020/10/10 02:49:06
---

<!-- TOC -->

- [기본](#기본)
    - [Dockerfile](#dockerfile)
        - [multi-stage builds](#multi-stage-builds)
        - [컨테이너를 실행하는 스크립트](#컨테이너를-실행하는-스크립트)
        - [Host Kernel 공유](#host-kernel-공유)
    - [스크립트](#스크립트)
    - [AWS](#aws)
    - [GCP](#gcp)
- [CMD vs. ENTRYPOINT](#cmd-vs-entrypoint)
    - [Keep Docker Containers Running](#keep-docker-containers-running)
- [Docker Resource Constraints](#docker-resource-constraints)
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

### multi-stage builds
Go는 빌드 후에 바이너리만 별도로 담을 수 있기 때문에[^fn-go] multi-stage builds를 사용하면 docker를 위한 최적의 언어 같다.

[^fn-go]: <https://github.com/GoogleCloudPlatform/kubernetes-engine-samples/tree/master/hello-app>

```docker
FROM golang:alpine
ADD . /go/src/hello-app
RUN go install hello-app

FROM alpine:latest
COPY --from=0 /go/bin/hello-app .
ENV PORT 8080
CMD ["./hello-app"]
```

### 컨테이너를 실행하는 스크립트
바깥에서 docker를 실행하기 위한 명령, 컨테이너를 새롭게 빌드하고 포트 맵핑, 디렉토리 맵핑(hot reload를 위해)을 처리했다.

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

### Host Kernel 공유
```console
$ docker run -it ubuntu:latest
root@97039839a4a6:/# uname -sr
Linux 4.19.76-linuxkit

$ docker run -it centos:latest
[root@071ff5d65f0f /]# uname -sr
Linux 4.19.76-linuxkit
```
어느 OS를 실행하던 커널은 동일하다. 컨테이너는 커널을 포함하지 않으며 OS 커널을 사용하면서 Kernel Space는 공유하기 때문이다. 초창기 리눅스에서만 컨테이너가 구동되었던 이유이기도 하다.

따라서, 아무 OS에서든 일단 실행 파일을 빌드하면 scratch에서도 잘 실행이 된다.
```docker
FROM scratch
ADD hello /
CMD ["/hello"]
```

hello 바이너리는 다음과 같이 만들 수 있다. 그런데 c++ static 빌드라서 실행파일만 2.3MB에 달한다. (alpine은 OS 전체도 5.5MB 밖에 안된다) c인 경우 0.8MB. static 빌드가 아닌 경우 해당 OS가 아니면 실행이 안된다.

```console
$ docker run -it -v $PWD:/build ubuntu:latest
$ apt update && apt install -y build-essential
$ cd /build && g++ -o hello -static hello.cc
```

결국 컨테이너는 Host 커널은 함께 사용하면서 유저 프로세스를 별도로 격리하는 역할을 한다.

## 스크립트
Gist 정리
- [update-docker.sh](https://gist.github.com/likejazz/85c54f4c6b69e60cb7b75f806659153d)  
업데이트가 있을 경우 자동으로 이미지를 받아서 교체
- [dockerize.sh](https://gist.github.com/likejazz/ba41d83fc94dbb75b982f4e37dc008b6)  
도커 이미지를 빌드해서 배포

## AWS
Push Docker Image to ECR
```bash
readonly VERSION=$(date '+%y.%m.%d')
readonly ECR_REGISTRY=0996xxx.dkr.ecr.ap-northeast-2.amazonaws.com/edith/edith-xxxx
readonly AWS_OTP=$(aws ecr get-login-password --region ap-northeast-2)

echo "${AWS_OTP}" | docker login --username AWS --password-stdin ${ECR_REGISTRY}
docker tag edith/edith-xxxx:latest ${ECR_REGISTRY}:"${VERSION}"
docker push ${ECR_REGISTRY}:"${VERSION}"
```

## GCP
Push the Docker image to Container Registry
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

# Docker Resource Constraints
docker는 기본적으로 자원을 제한하지 않지만 cpu/memory 리소스를 제한할 수 있다. macOS는 기본적으로 제한한 상태에서 구동된다.

docker의 memory resource를 8g까지 늘려 잡고 실제로 pandas로 메모리를 점유해봐도 host 머신의 메모리 점유는 늘지 않는다. 단순히 표현을 안할뿐인지, 아니면 가상 메모리를 활용하는지(이렇게 되면 성능이 떨어질텐데) 확인 필요. 애초에 docker를 실행할때는 일정 부분 메모리를 점유한다. 8.7g 점유하고 있던 메모리가 docker 종료시 3.7g가 된다. 메모리를 얼마를 잡든 항상 그 정도는 차지하는 듯 하다. 약간이라도 메모리를 더 확보하기 위해서는 안쓸때는 docker를 종료할 필요가 있다.

docker 내에서 CPULoadGenerator[^fn-cpu]로 테스트 해보니 host 메모리도 더 이상 늘지 않고 CPU는 나눠 쓴다. 꼭 같은 코어를 쓰는것도 아니다. 해당 코어가 일을 하고 있다면 다른 코어를 사용한다. 6개를 점유하고 있어도 6개가 유휴 상태가 아니라면 600% 일을 할 수 없고, host CPU가 4개만 남은 상태로 테스트 결과 490% 정도 밖에 성능을 낼 수 없었다. (다른 코어를 100%에서 80%까지 성능을 떨어트려 90% 여유분 확보) 

<img width="80%" src="https://user-images.githubusercontent.com/1250095/94359426-12072880-00e2-11eb-867d-9825c554153b.png">

총 합은 코어 갯수 * 100%로 동일. 즉, docker내에서는 100% 일 한다고 표시하지만 실제로는 100% 성능을 못내는 상태.

[^fn-cpu]: <https://github.com/GaetanoCarlucci/CPULoadGenerator/tree/Python3/>

```console
$ ./CPULoadGenerator.py -l 1.0 -d 20 -c 0 -c 1 -c 2 -c 3
$ yes > /dev/null &
```

이렇게 간단하게[^fn-yes] 스트레스 테스트를 할 수 있다. 맥에서는 이걸로 테스트. 내가 실험한건 docker의 부하가 host에 어떤 영향을 끼치는지 이고, ctop과 스트레스 전용 이미지를 이용해 docker 내부의 cpu 제약을 실험한 글[^fn-cpu-limit]도 참고.

[^fn-yes]: <https://osxdaily.com/2012/10/02/stress-test-mac-cpu/>
[^fn-cpu-limit]: <https://thorsten-hans.com/docker-container-cpu-limits-explained>

# Books
## 도커/쿠버네티스를 활용한 컨테이너 개발 실전 입문 <sub>2018, 2019</sub>
- 도커 컨테이너 배포
- ~~스웜을 이용한 실전 애플리케이션 개발~~ 필요 없음
- 쿠버네티스 입문/클러스터 구축  
    - GKE에서 진행
- 쿠버네티스 실전
    - kubectl, helm등 다양한 활용 방법 소개, rolling update 전략, service mesh Istio 언급.
- 컨테이너 운영
    - log 관리, GCP의 stackdriver logging 소개(현재는 Cloud Logging)
- 가벼운 도커 이미지 만들기
    - scratch, busybox, alpine 까지 소개한다.
