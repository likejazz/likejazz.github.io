---
layout: wiki 
title: CUDA
tags: ["MLOps & HPC"]
last_modified_at: 2024/06/20 17:24:29
---

- [드라이버 설치](#드라이버-설치)
  - [Uninstall](#uninstall)
  - [CUDA Toolkit](#cuda-toolkit)
  - [nvidia-container(Docker) 설치](#nvidia-containerdocker-설치)
  - [트러블슈팅](#트러블슈팅)
  - [버전 인식](#버전-인식)
- [Dockerfile](#dockerfile)
- [BLAS](#blas)

# 드라이버 설치

## Uninstall

```
# To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-12.4/bin
# To uninstall the NVIDIA Driver, run nvidia-uninstall

$ sudo reboot
```
uninstall 하지 않으면 기존 커널 모듈이 로드되어 있다고 다른 버전 CUDA 설치가 되지 않는다고 로그에 출력된다.

cuda 12.2는 [커널 라이센스 문제](https://forums.developer.nvidia.com/t/linux-6-7-3-545-29-06-550-40-07-error-modpost-gpl-incompatible-module-nvidia-ko-uses-gpl-only-symbol-rcu-read-lock/280908/37)로 우분투 22.04에 설치되지 않는다.


## CUDA Toolkit
<https://developer.nvidia.com/cuda-toolkit-archive> runfile(local) 방식이 가장 간단
cuda_12.5.0_555.42.02_linux.run (Jun 2024)

```bash
$ sudo apt install build-essential
$ wget ...
$ chmod +x cuda_12.5.0_555.42.02_linux.run
$ sudo ./cuda_12.5.0_555.42.02_linux.run
```

CUDA Toolkit은 드라이버와 개발툴 모두 설치해준다.

## nvidia-container(Docker) 설치
Docker내에서 apt 실행시 hash 에러가 발생하므로 apt 주소 다음과 같이 변경:
```bash
$ sudo vi /etc/apt/sources.list
:%s/kr.archive.ubuntu.com/mirror.kakao.com/
$ sudo apt update
```

[Install Docker with NVIDIA support](/wiki/Docker/#install-docker-with-nvidia-support) 참고

host에 cuda가 12.4가 설치되어 있다면 docker에서 cuda 12.5 버전은 구동되지 않는다. 하위 버전만 구동 가능하며 cuda와 driver는 host의 버전으로 인식된다.

## 트러블슈팅

CUDA를 설치하면 nvidia driver 버전은 맞춰서 함께 올라간다.

`nvidia-smi`실행시,
```
NVML: Driver/library version mismatch
```
갑자기 드라이버 오류 발생, 리부팅으로 해결[^fn-mismatch]

[^fn-mismatch]: <https://stackoverflow.com/a/45319156/3513266>

---

아래는 더 이상 쓰이지 않는 모든 언인스톨 과정:

```
$ sudo apt-get purge nvidia*
$ sudo apt-get autoremove

$ sudo systemctl isolate multi-user.target
$ sudo modprobe -r nvidia-drm

$ sudo reboot
```

```bash
$ sudo ubuntu-drivers list --gpgpu
$ sudo ubuntu-drivers install --gpgpu nvidia:550-server
$ sudo reboot
```

재부팅 후 `nvidia-smi`가 또 안되는 오류가 발생하여 [가이드](https://forums.developer.nvidia.com/t/nvidia-smi-has-failed-because-it-couldnt-communicate-with-the-nvidia-driver-make-sure-that-the-latest-nvidia-driver-is-installed-and-running/197141/6)에 따라 기존 드라이버 모두 언인스톨 후 cuda만 실행했다.

---

```
$ docker exec -it ollama nvidia-smi
Failed to initialize NVML: Unknown Error
```

docker에서 gpu 인식이 안되는 문제가 발생. docker를 재시작해야 해결이 된다.

## 버전 인식
docker에서는 host의 Driver / CUDA 버전을 docker가 그대로 따라간다. 당연히 nvcc는 이미지 내 설치된 버전을 따른다. 자기보다 상위 버전의 docker는 실행이 되지 않는다. 하위 버전의 docker에서도 현재 host의 CUDA 버전이 출력된다.

docker에서는 host가 12.5이면 이미지는 어떤 버전을 사용하던 nvidia-smi에서는 12.5로 고정되서 보인다.  
K8s에서는 12.2 이미지로 실행하면 nvidia-smi에서 12.2로 인식되고 12.5 이미지로 실행하면 12.5로 인식된다.  

# Dockerfile
CUDA 기반 Dockerfile:
```docker
FROM nvcr.io/nvidia/cuda:12.4.0-devel-ubuntu22.04

# Set the Timezone to KST
ENV TZ=Asia/Seoul

# Avoid question during docker build.
ARG DEBIAN_FRONTEND=noninteractive

# Change apt repo to kakao due to a hash error.
RUN sed -i "s/archive.ubuntu.com/mirror.kakao.com/g" /etc/apt/sources.list

# Install additonal packages.
RUN apt-get update && \
    apt-get install -y --no-install-recommends locales locales-all tzdata && \
    locale-gen en_US en_US.UTF-8 && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
    locale && \
    apt-get install -y --no-install-recommends \
          cmake \
          vim \
          silversearcher-ag \
          file \
          screen \
          wget \
          git \
          python3-dev \
          python3-pip \
          tree && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

# Default `python` runs `python3`
RUN echo 'alias python=python3' >> ~/.bashrc

# Default workdir is `/home/xxxx` on the host.
WORKDIR /xxxx
```

llama.cpp 빌드:
```bash
$ cmake .. -DLLAMA_CUDA=on -DLLAMA_CUDA_F16=1 -DCMAKE_CUDA_ARCHITECTURES=89
$ cmake --build . --config Release --parallel 8
```

huggingface-cli(HF_TOKEN 필요):
```bash
$ pip install -U "huggingface_hub[cli]"
$ huggingface-cli download google/gemma-2b-it
```

# BLAS
[cuBLAS가 가장 빠르다](https://siboehm.com/articles/22/CUDA-MMM). 슈트라센 알고리즘으로 $$O(n^{2.37188})$$으로 가능함에도 불구하고.