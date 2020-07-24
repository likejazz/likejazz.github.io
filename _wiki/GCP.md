---
layout: wiki 
title: GCP
last-modified: 2020/07/24 17:06:11
---

<!-- TOC -->

- [SSH 접속](#ssh-접속)
- [설치](#설치)
    - [Ubuntu](#ubuntu)
    - [Deep Learning](#deep-learning)
    - [디스크 마운트](#디스크-마운트)
- [운영](#운영)
- [ML Ops](#ml-ops)

<!-- /TOC -->

# SSH 접속
gcloud를 이용하는 방법은 너무 불편하여 ssh로 바로 접속이 가능하도록 셋팅하다.

compute engine에서 ssh keys에 gcp-user,
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSwZ37DGZfsOmT1zj40zT9uPnbNi5prL8fVEebvwWW9hYsbZ7ctcS9MXuUW3rUoDxlCyxIYBwqDqqbKTiUua94B1OpSzvsWwSOQ/... gcp-user
```
`cat ~/.ssh/id_rsa_gcp.pub`를 등록했다. ~~그러나 다른 서버에 등록한 설정도 동일 프로젝트에서 ssh 접속이 되는 현상이 있음. 동일한 centos에서 발생하는 것으로 보아 버그 같다.~~ Metadata > SSH Keys에 등록되어 있다. network interface details에서 내 IP에 대해 all allow 처리로 편리하게 사용한다.

# 설치
## Ubuntu
Ubuntu 20.04 LTS, SSD(Boot) 10G  
`Management, security, disks, networking, sole tenancy`에서 Disks, `+ Add new disk`, `Type: Local SSD scratch disk (maximum 24)` 빠른 파일 로딩을 위해 Local SSDs x 1 (NVME, 385 GB) 지정. 그러나, Local SSD는 instance stop이 안된다. 비용 절감을 할 수 없다.

```
# Anaconda
$ wget https://repo.anaconda.com/archive/Anaconda3-2020.02-Linux-x86_64.sh

# ==> For changes to take effect, close and re-open your current shell. <==

# Update to a newer version of Anaconda
conda update -n base -c defaults conda

# Developer Tools
$ sudo apt install gcc cmake
```

## Deep Learning

`Deep Learning Image: PyTorch 1.4.0 and fastai m51` 설치

~~Seoul 리전에서는 Tesla T4 인스턴스를 만들 수 없다. does not have enough resources available to fulfill the request 라고. 그래서 Tokyo에 생성했다.~~ 우리쪽 quota 부족으로 보인다. all quotas에서 요청을 하면 5분 이내에 바로 처리해준다. 반영되는데는 최대 15분 소요.

처음 접속할때 CUDA 최초 설치 필요. anaconda는 이미 설치되어 있다. nvtop 설치[^fn-nvtop] apk에는 없어서 소스 컴파일 설치했다.

[^fn-nvtop]: <https://github.com/Syllo/nvtop#nvtop-build>

XGBoost, CatBoost는 GPU 버전이 바로 설치되지만, LightGBM는 별도 옵션으로 설치(소스 컴파일됨)해야 한다. pytorch는 지난 3월에 릴리즈된 1.4.0이 이미 설치되어 있다.

## 디스크 마운트
SSD Persistent Disk는 이미 마운트 되어 있다. 아래는 LocalSSD의 경우인데, 이 instances는 stop이 안되기 때문에 비용 절감을 할 수 없다.

```console
$ lsblk
$ sudo mkfs.ext4 -F /dev/sdb
$ sudo mkdir -p /ml-experiments
$ sudo mount /dev/sdb /ml-experiments
$ sudo chmod a+w /ml-experiments
$ df -h
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
ML Engine: Serving scikit-learn, XGBoost tutorial[^fn-serv]

[^fn-serv]:<https://cloud.google.com/blog/products/gcp/serving-real-time-scikit-learn-and-xgboost-predictions>