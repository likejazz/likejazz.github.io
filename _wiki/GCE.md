---
layout: wiki 
title: GCE
tags: ["Cloud"]
last_modified_at: 2022/06/29 15:44:13
---

<!-- TOC -->

- [Compute Engine](#compute-engine)
  - [Ubuntu](#ubuntu)
  - [Container Optimized OS](#container-optimized-os)
  - [Deep Learning on Linux](#deep-learning-on-linux)
    - [Deep Learning Image](#deep-learning-image)
    - [Deep Learning VM](#deep-learning-vm)
- [설정](#설정)
  - [SSH 접속](#ssh-접속)
  - [서버 설정](#서버-설정)
  - [네트워크](#네트워크)
  - [CLI](#cli)
    - [환경 설정](#환경-설정)
- [기타](#기타)

<!-- /TOC -->

# Compute Engine
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

개인적으로 ubuntu를 선호하지만 GCP는 debian이 default다.

## Container Optimized OS
추가 디스크 공간이 `/mnt/stateful_partition/home`로 할당되어 있는데 `/home`과 동일하다. `$ readlink . -f`에서는 각각의 디렉토리 명이 표시된다.

`yum`, `apt` 모두 지원하지 않고 `toolbox`를 실행해서 설치[^fn-toolbox] 여러가지 유용한 도구 제공. k8s에서도 이 OS(약칭 cos)를 활용한다.

[^fn-toolbox]: <https://cloud.google.com/container-optimized-os/docs/how-to/toolbox>

```console
# Inside the toolbox shell
USER@cos-dev ~ $ toolbox
root@cos-dev:~# apt-get update && apt-get install -y htop psmisc
root@cos-dev:~# htop
root@cos-dev:~# pstree -p
root@cos-dev:~# exit
```

## Deep Learning on Linux
GCE에서 제공하는 딥러닝 이미지. 예전에는 Deep Learning Image와 Deep Learning VM이 별도로 존재했다. 지금도 Deep Learning VM은 남아 있지만 최근에는 관리가 안되는 것으로 보인다.

지금은 GCE 생성시 이 이미지를 통해 GPU 이미지를 생성할 수 있다. 만약 GPU 머신을 직접 구성한다면 이 이미지를 택한다.

### Deep Learning Image

**지금은 안보인다. Deep Learning VM은 그대로 있으나 이미지는 Deep Learning on Linux로 변경**

`Deep Learning Image: PyTorch 1.4.0 and fastai m51` 설치(이제 PyTorch 1.6.0도 지원한다)

T4의 생성 limit이 처음에 1로 되어 있는데, all quotas에서 요청을 하면 5분 이내에 바로 처리해준다. 반영되는데는 최대 15분 소요. 처음 접속할때 CUDA 최초 설치 필요. anaconda는 이미 설치되어 있다. 

XGBoost, CatBoost는 GPU 버전이 바로 설치되지만, LightGBM는 별도 옵션으로 설치(소스 컴파일됨)해야 한다. pytorch는 지난 3월에 릴리즈된 1.4.0이 이미 설치되어 있다. cuDF를 설치하고 싶었으나 conda로만 설치가 가능하고, 설치 진행이 안되서 실패. RAPIDS는 설치가 가장 어렵다.

### Deep Learning VM
Google Click to Deploy  
Seoul도 지원하고(K80만 보인다) Enable access to JupyterLab via URL instead of SSH도 지원한다. 최근 관리가 안되는 것으로 보인다. 예전에는 AI Platform에서도 동일 이미지로 보였으나 최근 Vertex AI에서는 별도로 관리하는듯.

RAPIDS XGBoost를 experimental 버전으로 바로 지원한다. conda 설치에 시달릴 필요가 없다. 특히 cuDF를 바로 사용할 수 있어 이 이미지를 택했다. PyCharm에서 Remote Python으로 Python Console로 실험. Jupyter 보다 편하다. 다만 큰 데이터는 오래 걸리므로 `Show Variables`는 turn off.

PyTorch 설치:
```console
$ conda install pytorch torchvision cudatoolkit=10.0 -c pytorch
```
CUDA 10.0이라서 PyTorch는 1.4.0 버전이 설치된다. pip 설치는 CUDA 버전이 맞지 않다며 실행되지 않음(이제 CUDA 11과 PyTorch 1.6도 있다)

python 3.7.3이 설치되어 있으니 `$ conda update -all` 진행. 그러나 cuDF를 사용하려면 numba가 0.48.0이어야 한다. 주의. `$ conda install -c numba numba=0.48.0`로 아래 버전 지정 설치. (어느새 pytorch는 1.3.1로 내려가 있다. 그러나 conda로 cudatoolkit=10.0 하면 1.4.1이 다시 설치된다.)

cuDF가 0.7이라서(최신은 0.14) 기능 제약이 많다. conda로 업데이트를 시도하면 당연히 잘 안된다.

# 설정

## SSH 접속
gcloud를 이용하는 방법은 너무 불편하여 ssh로 바로 접속이 가능하도록 셋팅. `cat ~/.ssh/id_rsa_gcp.pub`를 Metadata > SSH Keys에 등록했다. 해당 프로젝트내 모든 서버에 ssh 접속 가능하다. 정석대로는 gcloud를 이용해 ssh 접속이 가능하다.

```
$ gcloud compute ssh gcp-user@"NAME" --zone "asia-northeast3-c"
```

마찬가지로 Metadata > SSH Keys에 자동으로 등록하면서 접속이 진행된다. 맨 처음에 다음과 같은 메시지가 나온다.
```
Updating project ssh metadata...⠧
```
사용자 계정은 gcp-user로 로그인된다. 모두 sudo 권한을 갖고 있기 때문에 사실상 계정명은 큰 의미가 없다. default-allow-ssh로 모든 IP에 대해 ssh 접속 허용이 프로젝트 생성시 기본으로 되어 있는거 같다. 보안상 삭제가 필요하다.

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

방화벽 설정에서 소스와 접속 포트만 다음과 같이 나열할 수 있다.

```
gcloud compute firewall-rules list --format="table(
                name,
                sourceRanges.list():label=SRC_RANGES,
                allowed[].map().firewall_rule().list():label=ALLOW
            )"
```

## CLI
### 환경 설정
설정 조회
```console
$ gcloud config list
[compute]
region = asia-northeast3
zone = asia-northeast3-b
[core]
account = xxx@email.com
disable_usage_reporting = False
project = edith-xxx

Your active configuration is: [default]
```

```bash
$ gcloud config set project edith-xx
```
이처럼 default project 지정. 매 번 cli에서 project id를 입력하지 않아도 된다. 이외 `$ export PROJECT_ID=xxx` 방법도 있다.

설정은 `~/.config/gcloud` 위치에 저장된다. SQLite 포맷이며, 개별 조회는 할 수 없다.

# 기타
인증 정보는 다음과 같다.
1. Google Cloud SDK  
```
$ gcloud auth list
```
gcloud가 사용하는 계정 정보. GCP 콘솔에서 사용하는 계정 정보와 동일하다.
1. Google Auth Library
```
$ gcloud auth application-default login
```