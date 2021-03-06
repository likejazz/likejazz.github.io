---
layout: wiki 
title: GCP
last_modified_at: 2021/06/25 22:02:18
---

<!-- TOC -->

- [설치](#설치)
    - [Ubuntu](#ubuntu)
    - [Deep Learning Image](#deep-learning-image)
    - [Deep Learning VM](#deep-learning-vm)
    - [AI Platform](#ai-platform)
    - [Container Optimized OS](#container-optimized-os)
- [설정](#설정)
    - [SSH 접속](#ssh-접속)
    - [서버 설정](#서버-설정)
    - [네트워크](#네트워크)
    - [CLI](#cli)
        - [환경 설정](#환경-설정)
    - [Cloud Domains](#cloud-domains)
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

개인적으로 ubuntu를 가장 선호하지만 GCP는 debian을 미는 것 같다.

## Deep Learning Image
`Deep Learning Image: PyTorch 1.4.0 and fastai m51` 설치  
(이제 PyTorch 1.6.0도 지원한다)

T4의 생성 limit이 처음에 1로 되어 있는데, all quotas에서 요청을 하면 5분 이내에 바로 처리해준다. 반영되는데는 최대 15분 소요. 처음 접속할때 CUDA 최초 설치 필요. anaconda는 이미 설치되어 있다. htop과 유사한 nvtop 설치[^fn-nvtop] apk에는 없어서 소스 컴파일 설치했다.

```console
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
Solution provided by Google Click to Deploy  
여긴 Seoul도 지원하고 Enable access to JupyterLab via URL instead of SSH. (Beta) 도 Beta이긴 하지만 지원한다. 사실상 이걸 쓰면 AI Platform을 사용하지 않아도 되는거 아닌지?

RAPIDS XGBoost를 experimental 버전으로 바로 지원한다. conda 설치에 시달릴 필요가 없다. 특히 cuDF를 바로 사용할 수 있어 이 이미지를 택했다. PyCharm에서 Remote Python으로 Python Console로 실험. Jupyter 보다 더 편하다. 다만 큰 데이터는 오래 걸리므로 `Show Variables`는 turn off.

PyTorch 설치:
```console
$ conda install pytorch torchvision cudatoolkit=10.0 -c pytorch
```
CUDA 10.0이라서 PyTorch는 1.4.0 버전이 설치된다. pip 설치는 CUDA 버전이 맞지 않다며 실행되지 않음(이제 CUDA 11과 PyTorch 1.6도 있다)

python 3.7.3이 설치되어 있으니 `$ conda update -all` 진행. 그러나 cuDF를 사용하려면 numba가 0.48.0이어야 한다. 주의. `$ conda install -c numba numba=0.48.0`로 아래 버전 지정 설치. (어느새 pytorch는 1.3.1로 내려가 있다. 그러나 conda로 cudatoolkit=10.0 하면 1.4.1이 다시 설치된다.)

cuDF가 0.7이라서(최신은 0.14) 기능 제약이 많다. conda로 업데이트를 시도하면 당연히 잘 안된다.

## AI Platform
여기는 AI Platform에서 따로 인스턴스를 생성할 수 있다. 주피터 노트북까지 함께 설치되는 형태이며, Kaggle, PyTorch등 Deep Learning VM 처럼 다양한 팩키지를 이미 지원한다. PyTorch는 1.6.0이 최신 버전. BigQuery 연동도 되어 있다. 아쉽게도 Seoul이 아직 없어서 Tokyo로 설정

<img src="https://user-images.githubusercontent.com/1250095/102859486-4f741b80-446f-11eb-828f-e9aea6e0a96a.png" width="80%">

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

# 설정

## SSH 접속
gcloud를 이용하는 방법은 너무 불편하여 ssh로 바로 접속이 가능하도록 셋팅. `cat ~/.ssh/id_rsa_gcp.pub`를 Metadata > SSH Keys에 등록했다. 해당 프로젝트내 모든 서버에 ssh 접속 가능하다. 정석대로는 gcloud를 이용해 ssh 접속이 가능하다.

```
$ gcloud compute ssh gcp-user@"NAME" --zone "asia-northeast3-c"
```

마찬가지로 Metadata > SSH Keys에 자동으로 등록하면서 접속이 진행된다. 사용자 계정은 gcp-user로 로그인된다. 모두 sudo 권한을 갖고 있기 때문에 사실상 계정명은 큰 의미가 없다.

default-allow-ssh로 모든 IP에 대해 ssh 접속 허용이 프로젝트 생성시 기본으로 되어 있는거 같다. 보안상 삭제.

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

로컬 콘솔 외에도 Cloud Shell을 실행하는 방법이 있다. 모든 도구가 설치되어 있으며, 환경 차이 없이 가상 서버 터미널을 브라우저에서 표준 방식으로 사용할 수 있어 편리하다.

## Cloud Domains
Google Cloud Domains(Beta)는 프로젝트 기반이고 도메인을 다른 프로젝트로 이동 불가[^fn-gdomains] export 하게 되면 더 이상 관리 안되며 [Google Domains](https://domains.google.com/registrar)에서만 확인 가능. 여기는 계정 기반으로 프로젝트도 없는 별도 서비스다. 우리나라에서는 지원 안된다고 나오지만 My domains에서 확인 가능. 여기서 exported 도메인을 찾아 완전히 삭제할 수 있다.

[^fn-gdomains]: <https://stackoverflow.com/a/65962034/3513266>

# 운영
```
# 조회
$ gcloud compute instances list

# 시작/중지
$ gcloud compute instances start stark-seoul  # 시작
$ gcloud compute instances stop stark-seoul  # 중지
```

## 인증
인증 차이 숙지
```
$ gcloud auth list
```

```
$ gcloud auth application-default login
```
