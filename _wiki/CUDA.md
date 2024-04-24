---
layout: wiki 
title: CUDA
tags: ["MLOps & HPC"]
last_modified_at: 2024/04/22 14:43:55
---

- [CUDA](#cuda)
  - [Driver Update](#driver-update)
  - [CUDA Driver Update](#cuda-driver-update)
  - [nvidia-container(Docker) 설치](#nvidia-containerdocker-설치)
  - [트러블슈팅](#트러블슈팅)
  - [Dockerfile](#dockerfile)
- [BLAS](#blas)

# CUDA
## Driver Update
```bash
$ sudo ubuntu-drivers list --gpgpu
$ sudo ubuntu-drivers install --gpgpu nvidia:550-server
$ sudo reboot
```

재부팅 후 `nvidia-smi`가 또 안되는 오류가 발생하여 [가이드](https://forums.developer.nvidia.com/t/nvidia-smi-has-failed-because-it-couldnt-communicate-with-the-nvidia-driver-make-sure-that-the-latest-nvidia-driver-is-installed-and-running/197141/6)에 따라 기존 드라이버 모두 언인스톨 후 cuda만 실행했다.

## CUDA Driver Update
CUDA Toolkit 12.4 (Apr 2024)  
<https://developer.nvidia.com/cuda-toolkit-archive> runfile(local) 방식이 가장 간단

```bash
$ sudo apt install build-essential
$ wget https://developer.download.nvidia.com/compute/cuda/12.4.1/local_installers/cuda_12.4.1_550.54.15_linux.run
$ chmod +x cuda_12.4.1_550.54.15_linux.run
$ sudo ./cuda_12.4.1_550.54.15_linux.run
```

CUDA Installer가 drvier까지 같이 설치해주는 것으로 보이는데 확인 필요

언인스톨은 다음과 같다.

```
# To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-12.4/bin
# To uninstall the NVIDIA Driver, run nvidia-uninstall
$ sudo apt-get purge nvidia*
$ sudo apt-get autoremove

$ sudo systemctl isolate multi-user.target
$ sudo modprobe -r nvidia-drm

$ sudo reboot
```
## nvidia-container(Docker) 설치
apt에서 hash 에러가 발생하므로 apt 주소 변경:
```bash
$ sudo vi /etc/apt/sources.list
:%s/kr.archive.ubuntu.com/mirror.kakao.com/
$ sudo apt update
```

[Install Docker with NVIDIA support](/wiki/Docker) 참고

## 트러블슈팅

CUDA를 설치하면 nvidia driver 버전은 맞춰서 함께 올라간다.

`nvidia-smi`실행시,
```
NVML: Driver/library version mismatch
```
갑자기 드라이버 오류 발생, 리부팅으로 해결[^fn-mismatch]

[^fn-mismatch]: <https://stackoverflow.com/a/45319156/3513266>

## Dockerfile
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