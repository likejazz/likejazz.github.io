---
layout: wiki 
title: GCP
last-modified: 2020/08/05 14:22:08
---

<!-- TOC -->

- [SSH 접속](#ssh-접속)
- [설치](#설치)
    - [Ubuntu](#ubuntu)
    - [Deep Learning Image](#deep-learning-image)
    - [Deep Learning VM](#deep-learning-vm)
- [설정](#설정)
    - [디스크 마운트](#디스크-마운트)
- [운영](#운영)
- [MLOps](#mlops)
    - [Anaconda](#anaconda)
- [Google Cloud Storage](#google-cloud-storage)
    - [CLI](#cli)
- [BigQuery](#bigquery)
    - [권한](#권한)
    - [Jupyter Notebook](#jupyter-notebook)

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

# 설정
```
$ vi ~/.bash_aliases
alias ll='ls -al'
```

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

# 운영
```
# 조회
$ gcloud compute instances list --project=xxx

# 시작/중지
$ gcloud compute instances start stark-seoul --zone=asia-northeast3-c --project=xxx  # 시작
$ gcloud compute instances stop stark-seoul --zone=asia-northeast3-c --project=xxx  # 중지
```

# MLOps
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

conda 버전을 낮추면 된다는 얘기가 있어 따라해봤으나 여전히 안된다. 특히 4.6.x에서는 아예 killed 되어 버렸다. 여전히 RAPIDS conda 설치는 실패.

```console
$ conda install conda=4.6.14
$ wget https://repo.anaconda.com/pkgs/misc/conda-execs/conda-4.7.5-linux-64.exe
$ ./conda-4.7.5-linux-64.exe install -p /opt/conda conda=4.7.5
$ conda install -c rapidsai -c nvidia -c conda-forge -c defaults rapids=0.14 python=3.7 cudatoolkit=10.1
```

# Google Cloud Storage
Cloud Storage에 올려두고 로컬에 파일이 없을 경우 다운로드 하도록 구성.
```python
import os
from google.cloud import storage

BUCKET = "bucket"
PATH = '/ml-experiments'
FILE = 'sample.csv'

client = storage.Client()
if not os.path.exists(PATH + '/' + FILE):
    bucket = client.get_bucket(BUCKET)
    blob = bucket.blob(FILE)
    blob.download_to_filename(PATH + '/' + blob.name)

    print('Downloaded to {}'.format(PATH + '/' + blob.name))
```

## CLI
hive에서 gcs까지 과정
```console
# 원격 서버
$ ssh xxx@10.12.109.xxx

# 분석 결과 CSV export
$ hive -e "INSERT OVERWRITE DIRECTORY '/user/xxx/output6.csv'
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
select * from ignxxx.adaptive_advice_system_data_refined_6 limit 1000"
 
# CSV 결과 조회
$ hdfs dfs -ls output6.csv
 
# 로컬로 가져오기
$ hdfs dfs -get /user/xxx/output6.csv

# append header required.

# 맥북에서 streaming으로 GCS 업로드(5.3G 약 5분 소요, 로컬 디스크 공간 필요 없음)
$ scp xxx@10.12.109.xxx:output6.csv/000000_0 /dev/stdout | gsutil cp - gs://stark-xxx/output6.csv
```

원래 회사 유선망으로 100MB/s가 나오는데, 50MB/s 정도였고 remote server에서 가져오는 동안 멈춰있는 것으로 보인다.

# BigQuery
hive에서 `limit 1000000`으로 ~~csv download 후,~~ 다음과 같이 데이터가 csv 포맷이 되도록 쿼리 한다.

```sql
INSERT OVERWRITE DIRECTORY '/user/xxx/output.csv' 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
select * from table_xxx limit 1000000  -- 1M에서 650MB
```

`/user/xxx/output.csv`를 클릭하여 file browser에서 열고 download. 이후 최상단에 header만 추가해서 gcs에 upload. bigquery에서 create native table from gcs로 schema는 auto detect. header rows to skip은 1. invalid columns가 많을 경우 error count를 충분히 늘려주면 도움이 된다. bigquery dataset은 위치를 default로 한다. seoul(asia-northeast3)로 강제 지정했더니 gcs에서 import시 `Cannot read and write in different locations: source: asia, destination: asia-northeast3` 오류 발생. raw file은 gsutil을 이용해 gcs로 업로드 하는데, 사내 유선망은 100MB/s가 나와서 5.3G도 어렵지 않게 업로드 완료.

## 권한
BigQuery 쿼리 결과를 로컬 pandas로 내려서 분석 시도. Data Studio는 사용법도 다르고 무엇보다 data source connection 오류가 있어서 데이터를 부르지도 못했다. 로컬 분석은 가이드[^fn-guide]를 참고했다.

[^fn-guide]: <https://cloud.google.com/bigquery/docs/bigquery-storage-python-pandas#pip> 
```console
# Linux
$ pip install --upgrade google-cloud-bigquery[bqstorage,pandas]
# OSX
$ pip install --upgrade google-cloud-bigquery
# AttributeError: module 'grpc.experimental.aio' has no attribute 'Call' 오류로 인해
$ pip install --upgrade grpcio
```
conda는 여전히 설치되지 않음. 인증 문제가 있는데 vm 내에서 `$ gcloud auth application-default login`로 직접 처리.

```python
>>> %load_ext google.cloud.bigquery
>>> %%bigquery df --use_bqstorage_api
select * from ds.1m where vin = 'KMTHA81BBxxx'
>>> type(df)
pandas.core.frame.DataFrame
```
쿼리 결과가 dataframe에 맵핑된다. invalid value 때문에 모든 컬럼이 STRING이 되었다.

## Jupyter Notebook
```python
>>> %%bigquery df --use_bqstorage_api
select temperature, count(temperature) from ds.10m 
group by temperature having 
temperature != '-40.0' and
temperature != '\\N'
>>> df = df.sort_values(by='temperature')
>>> df.plot.bar(x='temperature', y='f0_', figsize=(20,10))
```
<img src="https://user-images.githubusercontent.com/1250095/89176000-c3e31680-d5c3-11ea-9d73-9a47ba9aefc5.jpg" width="80%">
