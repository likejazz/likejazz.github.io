---
layout: wiki 
title: Docker
tags:  ["Infrastructure"]
last_modified_at: 2024/09/02 16:22:36
---

- [기본](#기본)
  - [multi-stage builds](#multi-stage-builds)
  - [컨테이너를 실행하는 스크립트](#컨테이너를-실행하는-스크립트)
  - [Host Kernel 공유](#host-kernel-공유)
  - [root 사용자](#root-사용자)
- [enroot](#enroot)
- [스크립트](#스크립트)
  - [AWS](#aws)
  - [GCP](#gcp)
    - [Accessing private Google Container Registry from docker](#accessing-private-google-container-registry-from-docker)
- [CMD vs. ENTRYPOINT](#cmd-vs-entrypoint)
  - [sshd 시작](#sshd-시작)
- [Docker Resource Constraints](#docker-resource-constraints)
  - [Mac](#mac)
  - [Linux \& enroot](#linux--enroot)
- [GPU 지원](#gpu-지원)
  - [Install Docker with NVIDIA support](#install-docker-with-nvidia-support)
- [Docker Attach](#docker-attach)
- [Dockerfile](#dockerfile)
- [Docker without root](#docker-without-root)
- [Change apt repo to kakao due to a hash error](#change-apt-repo-to-kakao-due-to-a-hash-error)

# 기본
## multi-stage builds
Go는 빌드 후에 바이너리만 별도로 담을 수 있기 때문에[^fn-go] multi-stage builds를 사용하면 docker를 위한 최적의 언어다.

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

## 컨테이너를 실행하는 스크립트
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

디렉토리 맵핑을 지정할때는 상대 경로가 아니라 **절대 경로**로 해야 한다.

## Host Kernel 공유
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

참고로 alpine은 docker를 위한 최적의 OS로, 팩키지 매니지까지 제공하면서도 용량이 작기 때문에 매우 유용하다.
```console
$ apk add htop
```

## root 사용자
docker의 기본 사용자가 root이다 보니 여러 문제가 있다. jupyter notebook도 root로 실행되고, 다른 유저로 실행할 수 있는 옵션이 없음.

# enroot
enroot는 루트 권한 필요없이 userspace에서 동작하는, NVIDIA에서 만든 컨테이너 런타임이다. 컨테이너 내에 job을 실행해 배포(slurm등)할 때 사용한다. 원래 docker가 chroot의 영향을 많이 받았는데, enroot는 권한이 필요 없는 enhanced chroot로 볼 수 있다. docker가 데몬이나 프로세스 중심이라면 enroot는 독립된 환경 구성에 좀 더 촛점을 맞춘 느낌. chroot는 가이드[^fn-chroot]에 따라 `$ sudo chroot /home/ubuntu/chroot-jail/ /bin/bash`로 간단히 테스트해 볼 수 있으며, 완전히 격리된 환경을 제공한다. original author는 bill joy.

[^fn-chroot]: <https://www.journaldev.com/38044/chroot-command-in-linux>

```
$ enroot import docker://centos # docker 이미지 받아와 sqsh 생성
$ enroot create centos.sqsh     # 로컬에 파일 추출
$ enroot start centos           # 실행
```

원래 docker는 root가 기본이고 파일시스템 쓰기도 가능하지만 enroot는 일반 사용자와 read-only를 기본으로 실행된다. 따라서 root 사용자, 파일시스템 쓰기가 가능하려면 다음과 같이 실행한다.
```
$ enroot start --root --rw centos
```

docker는 자칫 프로세스가 종료되면 파일의 현재 상태가 모두 날아간다.

참고로, 로컬 이미지를 불러올때는 다음과 같이 `dockerd`로 지정한다.

```
$ docker import dockerd://pytorch-custom
```

extracting에는 다소 시간이 소요된다.

설치된 이미지는 `$ enroot list`로 확인 가능하며, 파일 시스템 자체는 `~/.local/share/enroot/` 아래로 설치된다.

# 스크립트
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

### Accessing private Google Container Registry from docker
docker-credential-gcr가 이미 설치되어 있기 때문에 다음과 같이 설정만 해주면 해당 프로젝트에서는 바로 접근 가능하다.
```
$ docker-credential-gcr configure-docker
```

COS에서는 다음과 같이 `runme.sh`를 만들어 GCR에서 바로 가져올 수 있다.
```bash
#!/bin/bash

# Stop & Remove
docker stop oauth2-container
docker rm oauth2-container

# Pull
docker pull asia.gcr.io/xxx/xxx:latest

# Run
docker run -d \
  --name oauth2-container \
  -p 5143:5000 \
  -e PYTHONUNBUFFERED=1 \
  asia.gcr.io/xxx/xxx:latest
docker logs -f oauth2-container
```

# CMD vs. ENTRYPOINT
`CMD`는 다음과 같이 `docker run -it <image> /bin/bash`로 CMD is ignored and bash interpreter runs instead. `ENTRYPOINT`는 `bash`로 대체할 수 없다.
```docker
FROM alpine:latest

ENTRYPOINT echo "Hello, 2"
```

단, 해당 이미지를 이용해 다시 `ENTRYPOINT`를 정의한 경우에는 기존 `ENTRYPOINT`가 무시되고 새로 정의한 `ENTRYPOINT`가 실행된다.
```docker
FROM alpine-hello-2:latest

ENTRYPOINT echo "Hello, 3"
```

```
$ docker run -it alpine-hello-3 /bin/sh
Hello, 3
```

`/bin/sh`는 무시되고 새로 정의한 `ENTRYPOINT`가 실행된다.

Keep Docker Containers Running:
```docker
FROM amazonlinux:latest

CMD tail -f /dev/null
```

`CMD`이므로 외부 컴맨드로 대체 가능하다.

## sshd 시작
Dockerfile 내부에서 `RUN service ssh start`해도 실행되지 않는다. Docker는 프로세스를 구동할 수 없기 때문. 마지막에 `CMD /usr/sbin/sshd -D`로 foreground로 실행해야 한다. 여러 프로세스는 supervisord를 사용 `CMD ["/usr/bin/supervisord"]`, `CMD`는 `/bin/bash`로 ignored 가능하다.

이전 이미지가 ENTRYPOINT가 설정되어 있다면 이마저도 어렵다. 이때는 일단 실행한 후에 따로 접속해서 ssh 실행[^fn-container]

[^fn-container]: <https://gist.github.com/likejazz/b9e3eadb39e941ddda77f11a3bc8fb6c>

# Docker Resource Constraints
## Mac
docker는 기본적으로 자원을 제한하지 않지만 cpu/memory 리소스를 제한할 수 있다. macOS는 기본적으로 제한한 상태에서 구동된다.

docker의 memory resource를 8g까지 늘려 잡고 실제로 pandas로 메모리를 점유해봐도 mac에서는 host 머신의 메모리 점유는 늘지 않는다. 애초에 mac은 docker를 실행할때 일정 부분 메모리를 점유한다.

docker 내에서 CPULoadGenerator[^fn-cpu]로 테스트 해보니 mac에서는 host 메모리도 더 이상 늘지 않고 CPU는 나눠 쓴다. 꼭 같은 코어를 쓰는것도 아니다. 해당 코어가 일을 하고 있다면 다른 코어를 사용한다. 6개를 점유하고 있어도 6개가 유휴 상태가 아니라면 600% 일을 할 수 없고, host CPU가 4개만 남은 상태로 테스트 결과 490% 정도 밖에 성능을 낼 수 없었다. (다른 코어를 100%에서 80%까지 성능을 떨어트려 90% 여유분 확보)

<img width="80%" src="https://user-images.githubusercontent.com/1250095/94359426-12072880-00e2-11eb-867d-9825c554153b.png">

총 합은 '코어 갯수 $$\times$$ 100%'로 동일. 즉, docker내에서는 100% 일 한다고 표시하지만 실제로는 100% 성능을 못내는 상태. mac에서는 Linux와 달리 hypervisor로 인해 다소 특이한 형태로 동작한다.

[^fn-cpu]: <https://github.com/GaetanoCarlucci/CPULoadGenerator/tree/Python3/>

```console
$ ./CPULoadGenerator.py -l 1.0 -d 20 -c 0 -c 1 -c 2 -c 3
$ yes > /dev/null &
```

`yes`로 간단하게[^fn-yes] 스트레스 테스트도 할 수 있다. mac에서는 docker내 프로세스가 보이지 않으며, 각 컨테이너의 사용량을 확인할 때는 ctop[^fn-ctop]이 유용하다. 동일한 이름으로 cgroup 중심인 ctop 우분투 패키지가 있기 때문에 유의.

[^fn-yes]: <https://osxdaily.com/2012/10/02/stress-test-mac-cpu/>
[^fn-ctop]: <https://github.com/bcicen/ctop>

## Linux & enroot
Linux & enroot 환경에서 동일하게 CPU 부하 실험을 한 결과는 다음과 같다.
<img src="https://user-images.githubusercontent.com/1250095/168588637-ca0a3d65-6fb9-4391-a65c-87bfb4e04247.png" width="70%">

enroot 내에서 CPU 2번 코어에 100% 부하를 주자 mac과 달리 host에서 정확히 해당 코어의 부하가 올라가는 것을 확인할 수 있다. enroot 이미지는 PyTorch를 약간 수정했으며 별다른 작업을 하지 않았다. host에서 enroot 내에서 실행되는 명령이 표시되며(프로세스가 `ps`에서도 보인다) 정확하게 부하가 증가하는 것을 확인할 수 있다. enroot의 특징이라기 보다는 Linux에서는 docker의 containerd도 동일하게 동작한다. 프로세스가 `ps`에서 보이고, CPU, 메모리도 내외부가 동일하게 동작한다. enroot 내에서는 바깥의 다른 프로세스가 모두 nobody로 표시되며, 파일 시스템에는 접근할 수 없다.

# GPU 지원
docker 19.03 부터 nvidia-docker2가 deprecated 되고 GPU를 지원한다.
```
$ docker run --gpus all [container]
```
`--gpus` 옵션을 부여해야 `nvidia-smi` 바이너리가 컨테이너에 포함되며, 옵션을 부여하지 않은 경우 GPU 시스템에서도 `nvidia-smi`를 실행할 수 없다. 또한 CPU 시스템은 옵션을 줄 수 없기 때문에 마찬가지로 `nvidia-smi`가 없다고 나온다.

`nvidia-smi`는 docker 이미지 외부 프로세스이므로 enroot에서는 소유자가 nobody로 표시되며, docker에서는 root로 표시된다.

## Install Docker with NVIDIA support
snap으로 설치했더니 libnvidia-ml.so를 찾을 수 없다며 `docker --gpus all`이 구동되지 않는다. 이후에도 snap 설치는 계속 실패하고 있다.

[nvidia-docker 설치 가이드](https://gist.github.com/Merwanski/dd2c928d5190d4e0a3f3b9b5766d7049#file-install_docker_with_nvidia-sh). (Jetson에서는 이 가이드 필요 없이 JetPack으로 설치된다.)

이 가이드는 먼저 get.docker.com으로 apt를 설치하고 이후 nvidia-docker2를 설치하는 방식이다. 별 다른 문제 없이 잘 설치된다. 나중에 aws 인증 오류가 발생한다면 `$ docker logout public.ecr.aws`

```
$ docker run --rm --gpus all ubuntu nvidia-smi
```

# Docker Attach

attach로 연결:

```bash
$ docker attach e4413eeff75e
```

`docker attach`에서 세션을 유지한 상태에서 빠져나올 때 `CTRL+P,Q`를 눌러야 한다. `screen`에서는 `CTRL+A,D`였다. `CTRL+D`로 빠져나오면 세션을 종료한다.

# Dockerfile

[#1-3. Dockerfile for L4T PyTorch (ssh version)](/wiki/Private-Links)

Timezone 설정, 기본 패키지 설치, nvtop, pip, ssh 접속까지 설정한 Dockerfile Template. 컨테이너 내부에는 데몬을 실행해두어도 모든 프로세스가 종료된 상태로 패키징 되기 때문에 ssh 데몬은 runme시 접속해서 직접 실행하는 형태로 구현했다.

# Docker without root
```shell
$ sudo usermod -a -G docker $USER
$ sudo chmod 666 /var/run/docker.sock
```

# Change apt repo to kakao due to a hash error
```dockerfile
# Change apt repo to kakao due to a hash error.
RUN sed -i "s/archive.ubuntu.com/mirror.kakao.com/g" /etc/apt/sources.list
```