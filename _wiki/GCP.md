---
layout: wiki 
title: GCP
last-modified: 2020/07/28 14:15:02
---

<!-- TOC -->

- [SSH 접속](#ssh-접속)
- [설치](#설치)
    - [Ubuntu](#ubuntu)
    - [Deep Learning Image](#deep-learning-image)
    - [Deep Learning VM](#deep-learning-vm)
    - [디스크 마운트](#디스크-마운트)
- [설정](#설정)
- [운영](#운영)
- [ML Ops](#ml-ops)
    - [Anaconda](#anaconda)
    - [데이터](#데이터)

<!-- /TOC -->

# SSH 접속
gcloud를 이용하는 방법은 너무 불편하여 ssh로 바로 접속이 가능하도록 셋팅했다.

`cat ~/.ssh/id_rsa_gcp.pub`를 Metadata > SSH Keys에 등록했다. 해당 프로젝트 모든 서버에서 ssh 접속 가능. network interface details에서 내 IP에 대해 all allow 처리로 편리하게 사용한다.

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

XGBoost, CatBoost는 GPU 버전이 바로 설치되지만, LightGBM는 별도 옵션으로 설치(소스 컴파일됨)해야 한다. pytorch는 지난 3월에 릴리즈된 1.4.0이 이미 설치되어 있다. CuDF를 설치하고 싶었으나 conda로만 설치가 가능하고, 설치 진행이 안되서 실패.

## Deep Learning VM
RAPIDS XGBoost를 experimental 버전으로 바로 지원한다. conda 설치에 시달릴 필요가 없다. 특히 CuDF를 바로 사용할 수 있어 이 이미지를 택했다. PyCharm에서 Remote Python으로 Python Console로 실험. Jupyter 보다 더 편하다. 다만 큰 파일은 오래 걸리므로 `Show Variables`는 turn off.

아래는 XGBoost를 `gpu_hist`로 학습하는 모습이다.
<img src="https://user-images.githubusercontent.com/1250095/88621747-35125d80-d0dc-11ea-8ed8-a0d5832dfd2d.png" width="70%">

## 디스크 마운트
SSD Persistent Disk는 이미 마운트 되어 있다. 아래는 LocalSSD의 경우인데, 이 instance는 stop이 안되기 때문에 비용 절감을 할 수 없다.

```console
$ lsblk
$ sudo mkfs.ext4 -F /dev/sdb
$ sudo mkdir -p /ml-experiments
$ sudo mount /dev/sdb /ml-experiments
$ sudo chmod a+w /ml-experiments
$ df -h
```

# 설정
```
$ vi ~/.bash_aliases
alias ll='ls -al'
```

# 운영
```
# 조회
$ gcloud compute instances list --project=xxx

# 시작/중지
$ gcloud compute instances start stark-seoul --zone=asia-northeast3-c --project=xxx  # 시작
$ gcloud compute instances stop stark-seoul --zone=asia-northeast3-c --project=xxx  # 중지
```

# ML Ops
ML Engine: [Serving scikit-learn, XGBoost tutorial](https://cloud.google.com/blog/products/gcp/serving-real-time-scikit-learn-and-xgboost-predictions)

## Anaconda
conda가 설치된 상태에서 추가 설치 및 업그레이드가 필요하다. 특히 RAPIDS는 conda로만 설치된다. C++14 및 CUDA 제약 사항으로 인해[^fn-conda] 그런데, `$ conda update --all --verbose` 조차 제대로 실행 안됨. 채널에서 정보를 가져오는데 상당한 제약 사항이 있다. 다음과 같이 수정 필요.

```console
$ conda config --show-sources
==> /home/gcp-user/.condarc <==
channel_priority: flexible
channels:
  - conda-forge
  - defaults
```

채널 설정을 `flexbile`로 두는게 핵심이다. 이 사소한 설정으로 인해 `Solving environment:`가 hang up되어 이후 과정이 진행 안됨. 대부분의 문서에서,

```console
$ conda config --set channel_priority strict
```
를 가이드 하지만 실제로는,
```console
$ conda config --set channel_priority flexible
```
에서 동작했다. 추가 확인이 필요하다. 

느리고, 설정에 sensitive한 설치로 conda 설치에 대한 신뢰가 많이 떨어져 있다. 다행히 Deep Learning VM으로 이미 RAPIDS가 설치된 이미지로 부팅할 수 있다.

[^fn-conda]: <https://medium.com/rapids-ai/rapids-0-7-release-drops-pip-packages-47fc966e9472>

## 데이터
Google Storage에 올려두고 로컬에 없을 경우 다운로드 하도록 구성.
```python
from google.cloud import storage
client = storage.Client()
```
추후 BigQuery로 gstorage에 있는 JSON 분석 과정 추가.