---
layout: wiki 
title: GCP
last-modified: 2020/09/17 03:32:05
---

<!-- TOC -->

- [설치](#설치)
    - [Ubuntu](#ubuntu)
    - [Deep Learning Image](#deep-learning-image)
    - [Deep Learning VM](#deep-learning-vm)
    - [Container Optimized OS](#container-optimized-os)
- [설정](#설정)
    - [SSH 접속](#ssh-접속)
    - [서버 설정](#서버-설정)
    - [네트워크](#네트워크)
    - [CLI](#cli)
- [운영](#운영)
    - [인증](#인증)

<!-- /TOC -->

# 설치
## Ubuntu
Ubuntu 20.04 LTS, SSD(Boot) 10G  
`Management, security, disks, networking, sole tenancy`에서 Disks, `+ Add new disk`, `Type: Local SSD scratch disk (maximum 24)` 빠른 파일 로딩을 위해 Local SSDs x 1 (NVME, 385 GB) 지정. 그러나, Local SSD는 instance stop이 안된다. 비용 절감을 할 수 없다. 

아래는 conda 및 gcc, cmake 별도 설치하는 과정:
```
# Anaconda
$ wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh

# ==> For changes to take effect, close and re-open your current shell. <==

# Update to a newer version of Anaconda
conda update -n base -c defaults conda

# Developer Tools
$ sudo apt install gcc cmake
```

## Deep Learning Image
`Deep Learning Image: PyTorch 1.4.0 and fastai m51` 설치

T4의 생성 limit이 처음에 1로 되어 있는데, all quotas에서 요청을 하면 5분 이내에 바로 처리해준다. 반영되는데는 최대 15분 소요. 처음 접속할때 CUDA 최초 설치 필요. anaconda는 이미 설치되어 있다. nvtop 설치[^fn-nvtop] apk에는 없어서 소스 컴파일 설치했다.

```
$ sudo apt install cmake libncurses5-dev libncursesw5-dev
$ git clone https://github.com/Syllo/nvtop.git
$ mkdir -p nvtop/build && cd nvtop/build
$ cmake ..
$ make
$ sudo make install
```

[^fn-nvtop]: <https://github.com/Syllo/nvtop#nvtop-build>

XGBoost, CatBoost는 GPU 버전이 바로 설치되지만, LightGBM는 별도 옵션으로 설치(소스 컴파일됨)해야 한다. pytorch는 지난 3월에 릴리즈된 1.4.0이 이미 설치되어 있다. cuDF를 설치하고 싶었으나 conda로만 설치가 가능하고, 설치 진행이 안되서 실패.

## Deep Learning VM
RAPIDS XGBoost를 experimental 버전으로 바로 지원한다. conda 설치에 시달릴 필요가 없다. 특히 cuDF를 바로 사용할 수 있어 이 이미지를 택했다. PyCharm에서 Remote Python으로 Python Console로 실험. Jupyter 보다 더 편하다. 다만 큰 데이터는 오래 걸리므로 `Show Variables`는 turn off.

PyTorch 설치:
```console
$ conda install pytorch torchvision cudatoolkit=10.0 -c pytorch
```
CUDA 10.0이라서 PyTorch는 1.4.0 버전이 설치된다. pip 설치는 CUDA 버전이 맞지 않다며 실행되지 않음.

python 3.7.3이 설치되어 있으니 `$ conda update -all` 진행. 그러나 cuDF를 사용하려면 numba가 0.48.0이어야 한다. 주의. `$ conda install -c numba numba=0.48.0`로 아래 버전 지정 설치. (어느새 pytorch는 1.3.1로 내려가 있다. 그러나 conda로 cudatoolkit=10.0 하면 1.4.1이 다시 설치된다.)

cuDF가 0.7이라서(최신은 0.14) 기능 제약이 많다. conda로 업데이트를 시도하면 당연히 잘 안된다.

## Container Optimized OS
추가 디스크 공간이 `/mnt/stateful_partition/home`로 할당되어 있는데 `/home`과 동일하다. `$ readlink . -f`에서는 각각의 디렉토리 명이 표시된다.

`yum`, `apt` 모두 지원하지 않고 `toolbox`를 실행해서 설치[^fn-toolbox] 여러가지 유용한 도구 제공

[^fn-toolbox]: <https://cloud.google.com/container-optimized-os/docs/how-to/toolbox>

k8s에서도 이 OS를 활용한다고.

# 설정

## SSH 접속
gcloud를 이용하는 방법은 너무 불편하여 ssh로 바로 접속이 가능하도록 셋팅. `cat ~/.ssh/id_rsa_gcp.pub`를 Metadata > SSH Keys에 등록했다. 해당 프로젝트내 모든 서버에서 ssh 접속 가능. 접속 관련은 아래 네트워크 섹션 참고.

## 서버 설정
alias, 시간, 로케일 설정
```
$ vi ~/.bash_aliases
alias ll='ls -al'

# Set up a clean UTF-8 environment
sudo bash -c 'echo "LANG=en_US.utf-8
LC_ALL=en_US.utf-8" > /etc/environment'
 
# Set the Timezone to KST
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```

디스크의 경우 LocalSSD가 가장 빠르지만 별도로 마운트 해야 할뿐 아니라 stop이 안되기 때문에 비용 절감을 할 수도 없다.

## 네트워크
VPC networks - Firewall에서 내 IP에 대해 allow all 처리로 편하게 이용. 태그를 지정하면 해당 태그에만 룰이 적용되도록 설정 가능. network logging도 가능하다.

## CLI
gcloud cli 설정 조회
```console
$ gcloud config list
[compute]
region = asia-northeast3
zone = asia-northeast3-a
[core]
account = xxx@email.com
disable_usage_reporting = False
project = edith-xxx

Your active configuration is: [default]
```
default project도 지정. 매 번 cli에서 project id를 입력하지 않아도 된다. 이외 `$ export PROJECT_ID=xxx` 방법도 있음.

# 운영
```
# 조회
$ gcloud compute instances list --project=xxx

# 시작/중지
$ gcloud compute instances start stark-seoul --zone=asia-northeast3-c --project=xxx  # 시작
$ gcloud compute instances stop stark-seoul --zone=asia-northeast3-c --project=xxx  # 중지
```

## 인증
인증 차이 숙지
```
$ gcloud auth list
```

```
$ gcloud auth application-default login
```

디폴트 프로젝트 변경. 편하게 사용하기 위해 꼭 필요하다.
```
$ gcloud config set project XXX
```