---
layout: wiki 
title: Serverless AI
last-modified: 2020/12/29 13:45:26
---

<!-- TOC -->

- [AWS Personalize](#aws-personalize)
- [GCP Recommendations AI](#gcp-recommendations-ai)
- [AWS SageMaker](#aws-sagemaker)
- [GCP AI Platform](#gcp-ai-platform)
    - [Local Predict](#local-predict)
- [GCP GCE](#gcp-gce)
    - [Conda](#conda)
    - [Notebook](#notebook)

<!-- /TOC -->

# AWS Personalize
[가이드](https://docs.aws.amazon.com/personalize/latest/dg/gs-prerequisites.html)에 따라 진행

IAM 생성 권한이 없어서 Dataset 생성에서 더 이상 진행되지 않음.

MovieLens 예제는 rating 컬럼을 제거하는데, 그렇다면 Get Recommendations에서 무작위로 보여주는게 아닌가?  
원래 MovieLens 데이터는 rating, user, item 정보 모두 포함되어 있음

나 같으면 10% 정도를 A/B 테스트 용도로 할당하고 피드백을 받아 1/10 보다 CTR이 높은 경우 대체하는 방식으로 구현할 것 같다. 여기서도 SDK 또는 JS 라이브러리로 사용자 이벤트를 추적하는 것 같다.

# GCP Recommendations AI
GCP의 경우도 거의 유사하다. 모델 생성 이후에는 JavaScript pixel을 심어서 피드백을 받아 A/B 테스트를 진행한다. 이를 통해 CTR 등을 측정한다.

# AWS SageMaker
학습 부터 배포 까지 전체 데이터 플로우를 관리한다. 심지어 IDE 까지 제공  
모델 경량화 서빙을 위한 SageMaker Neo

# GCP AI Platform

ml-engine에서 이름이 바꼈다. [공식 가이드 문서](https://cloud.google.com/ai-platform/prediction/docs/getting-started-scikit-xgboost)가 가장 정확. 예전 블로그는 outdated.

us-central1만 지원해서 모델을 이쪽으로 업로드 하도록 별도 생성. input-type이 json인데, 특이한 형태로 받고 있어서 `gcloud ai-platform predict --help` 도움말로 확인 필요. new line으로 입력값을 구분한다. 바로 모델을 업로드 하니 잘 안되는데, `local predict` 기능이 있어서 여기서 먼저 잘 되는지 확인할 수 있다. 매우 편리. 실제로 로컬 모듈을 활용하기 때문에, tf도 설치되어 있어야 한다.

xgboost를 sckit-learn interface로 했다. 이 경우 framework이 XGBOOST인지 SCIKIT_LEARN인지도 애매하다. (xgboost가 직접 구동되어야 하니 당연히 XGBOOST이다.) xgboost DMatrix로 모두 변환. 기존에 XGBClassifier()가 자동으로 wrapping해서 처리해주는 것들이 많았는데, 전부 직접 해야 해서 매우 번거롭다. 그러나, 덕분에 early stopping 처리도 하고, LabelEncoder()도 직접 구현할 수 있었다. 이제 ai-platform에서 잘 동작한다. 다만, python/java만 제공해서 웹 프론트를 python으로 변경해야 한다. 다시 해보니 기존 scikit-learn wrapper도 잘 동작한다. 결과를 보면 `multi:softprob`로 되어 있다. 그래서 n개 중에 확률 높은 하나를 직접 처리해주는 작업이 필요함. 기존에 wrapper가 이 작업을 해줬다.

## Local Predict
```console
$ gcloud ai-platform local predict --model-dir=. \
--json-instances=./input.json --framework=XGBOOST
```

```bash
# input.json
[46042,276,32.5,28.5,22.5,22.5,10,1,1,1]
[3.8306e+04, 9.1500e+02, 2.8000e+01, 3.0500e+01, 6.5000e+01,1.4000e+01, 0.0000e+00, 1.0000e+00, 1.0000e+00, 0.0000e+00]
```

AI-Platform에서는 입력값을 다음과 같이 한다.
```json
{"instances":[[46042,276,32.5,28.5,22.5,22.5,10,1,1,1]]}
```

# GCP GCE
GCE 설치에 관한 내용이지만, ML과 관련이 있어서 여기에 정리
## Conda
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

## Notebook
노트북을 설치한 인스턴스를 생성해준다. 아직 Seoul은 없음. 인스턴스를 생성하고 JupyterLab을 직접 설치하는 것과 뭐가 다르냐고 할 수 있지만 구글 인증을 포함한 많은 부분이 이미 처리되어 있어 매우 편리하다. 

- Python 2 and 3 지정
- 비용 최소를 위해 1 CPU 별도 지정
- asia-northeast1-b (Tokyo)

<img src="https://user-images.githubusercontent.com/1250095/93212937-38ea6400-f79e-11ea-971f-3febfc7d9f7d.png" width="100%">

`%%bigquery` 매직 키워드 바로 사용 가능. 처음에 storage warning이 발생해서 콘솔에서 `$ pip install google-cloud-bigquery-storage` 커널 재시작만 한 번 해줌.
```
%%bigquery df
select temp, count(temp) from aas
group by temp
order by temp asc
```
노트북은 public 접속 가능하나 구글 인증으로 확인이 되어야 접속 가능하다.