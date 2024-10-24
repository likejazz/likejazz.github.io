---
layout: wiki 
title: GCP AI
tags: ["Cloud"]
last_modified_at: 2024/10/24 00:35:51
---

<!-- TOC -->

- [Vertex AI](#vertex-ai)
  - [AI Platform](#ai-platform)
    - [Models 및 Local Predict](#models-및-local-predict)
    - [AutoML(Vertex AI)](#automlvertex-ai)
  - [AI Platform \> Notebook](#ai-platform--notebook)
- [Recommendations AI](#recommendations-ai)
- [TPU](#tpu)
- [Colab](#colab)

<!-- /TOC -->

# Vertex AI

ml-engine → AI Platform → Vertex AI 순으로 진화

Vertex AI > Workbench > Managed Notebooks에서 관리형 GPU(T4, V100, A100 모두 가능)로 노트북을 사용할 수 있다. `apt`는 막혀 있지만 `pip`는 가능하다. tf, pytorch, xgboost는 이미 설치된 이미지를 제공한다.

관리형이므로 콘솔도 주피터에서 직접 접근한다.

## AI Platform
NOTICE: 기존 AI Platform을 정리했던 내용이다. 지금은 대부분 DEPRECATED 되고, Vertex AI로 연결된다.

<img width="20%" src="https://user-images.githubusercontent.com/1250095/116340926-f51f0900-a81a-11eb-860a-b3f3adfd2aad.png" border="1">

- AI Hub는 다양한 pre training 모델을 제공한다. 
- Data Labeling은 레이블링 툴 제공. Explosion의 Prodigy와 유사할 것으로 보임.
- ~~Notebooks~~Workbench는 jupyter notebook을 제공한다. 한국은 지원하지 않았지만 2021년 기준 Vertex AI가 등장하면서 train/predict 통합 파이프라인을 제공하고 Notebooks도 서울을 지원한다.
- 과거 AI Platform > Pipelines는 완전히 동일한 Kubeflow Pipelines였는데, Vertex AI에서는 비슷하나 조금 다른 형태다.
- Jobs는 training을 지원하는데, built-in은 BERT를 택해서 학습할 수 있고, custom은 ML framework을 선택할 수 있다. (scikit-learn, TensorFlow, XGBoost 중 택 일)
- Models는 prediction 서비스로 사실상 가장 유용한 서비스다. ~~마찬가지로 아직 한국은 지원하지 않아서 모델을 한국 이외 리전에 업로드해야 한다.~~ Vertex AI가 되면서 서울도 지원한다. input-type이 json인데, 특이한 형태로 받고 있어서 `gcloud ai-platform predict --help` 도움말로 확인 필요. new line으로 입력값을 구분한다. 바로 모델을 업로드 하니 잘 안되는데, `local predict` 기능이 있어서 여기서 먼저 잘 되는지 확인할 수 있다. 매우 편리. 실제로 로컬 모듈을 활용하기 때문에, tf도 설치되어 있어야 한다.
  - Vertex AI에서는 Models는 Training 작업을 하는 것이고, predict는 Endpoints에서 진행된다. Endpoints는 하나 이상의 Models가 필요하다.

### Models 및 Local Predict
xgboost를 sckit-learn interface로 했다. 이 경우 framework이 XGBOOST인지 SCIKIT_LEARN인지도 애매하다. (xgboost가 직접 구동되어야 하니 당연히 XGBOOST이다.) xgboost DMatrix로 모두 변환. 기존에 XGBClassifier()가 자동으로 wrapping해서 처리해주는 것들이 많았는데, 전부 직접 해야 해서 매우 번거롭다. 그러나, 덕분에 early stopping 처리도 하고, LabelEncoder()도 직접 구현할 수 있었다. 이제 ai-platform에서 잘 동작한다. 다만, python/java만 제공해서 웹 프론트를 python으로 변경해야 한다. 다시 해보니 기존 scikit-learn wrapper도 잘 동작한다. 결과를 보면 `multi:softprob`로 되어 있다. 그래서 n개 중에 확률 높은 하나를 직접 처리해주는 작업이 필요함. 기존에 wrapper가 이 작업을 해줬다. Local Predict는 다음과 같이 실행한다.

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

### AutoML(Vertex AI)
[Building a fraud detection model with AutoML](https://codelabs.developers.google.com/vertex-automl-tabular)을 보면 BigQuery에 있는 데이터를 이용해 데이터셋을 만들고, 학습하고, 모델을 만들어 서빙하는 과정을 진행한다. Datasets→Models→Endpoints 순서.
- BigQuery 데이터가 us 리전에 있어 서울 리전으로 데이터셋을 만들면 가져올 수 없다. us-central로 데이터셋을 지정해서 가져옴. 
- node hours를 1로 했을때 학습에 2시간 30분이 걸렸다.
- feature importance를 확인할 수 있다. 백서를 보면 SHAP을 쓴 것으로 보인다.
- `projects/YOUR-PROJECT-NUMBER/locations/us-central1/endpoints/YOUR-ENDPOINT-ID` 2개의 숫자 기입 필요. 주피터 노트북에서 인증 없이 호출 가능하다. 숫자만 맞추면 아무나 호출할 수 있는데, Brute-Force Attack이나 보안은 어떤식으로 강화했을까.

## AI Platform > Notebook
과거 노트북 제공 방식, GCE를 자동으로 생성해준다. (요즘 Vertex AI에는 Managed Notebook이 있다) 

```
======================================
Welcome to the Google Deep Learning VM
======================================

Version: pytorch-gpu.1-9.m79
Based on: Debian GNU/Linux 10 (buster) (GNU/Linux 4.19.0-17-cloud-amd64 x86_64\n)
```
이와 같은 MOTD로 과거 Deep Learning VM과 동일한 것으로 보이며, Seoul 리전도 추가로 오픈했다.

<img src="https://user-images.githubusercontent.com/1250095/93212937-38ea6400-f79e-11ea-971f-3febfc7d9f7d.png" width="100%">

<img src="https://user-images.githubusercontent.com/1250095/102859486-4f741b80-446f-11eb-828f-e9aea6e0a96a.png" width="80%">

BigQuery 연동이 가능하다.

`%%bigquery` 매직 키워드 바로 사용 가능. 처음에 storage warning이 발생해서 콘솔에서 `$ pip install google-cloud-bigquery-storage` 커널 재시작만 한 번 해줌.
```
%%bigquery df
select temp, count(temp) from aas
group by temp
order by temp asc
```

# Recommendations AI
AWS와 유사하게 다음 3가지 기능을 제공한다.
1. Others you may like(Typically used on product detail pages)
1. Frequently bought together(Commonly displayed after add-to-cart events)
1. Recommended for you(Typically used on home page)

2021년 4월 기준 세 가지 모델 타입을 지원하며, CTR 또는 CVR로 성능을 평가한다. 모델 생성 이후에는 JavaScript pixel을 심어서 피드백을 받아 A/B 테스트를 진행한다.

# TPU
21년 하반기 현재 tpu-vm 생성은 아직 preview이고, tpu-node는 정식으로 지원. 각 대륙별로 1군데씩 지원하는 정도(미국만 `us-central1-[a-c]` 지원)

엄밀히 따지면 tpu-node는 managed이고 TPUs에서 별도 관리. 이에 대응되는 vm이 VM instances에 추가되는 구조다. 실제로도 executor처럼 vm에서 connect 후 실행된다. tpu-node만 삭제하면 vm에서 더 이상 실행되지 않는다.

# Colab
Datalore는 UI가 미려하고 특히 DataSpell(아직 Remote Python 불가)처럼 auto complete를 지원해서 편하지만 일부 노트북이 제대로 보이지 않는 경우가 있다. 특히 로그 출력이 안보인다. Datalore의 문제인지, 특정 패키지의 문제인지 확실하지 않음. 그러나 Colab은 유명하다 보니 대부분 별도로 처리해준다. 특히 GCE VM 연결이 가능(21년 10월에 추가된 기능)하기 때문에 마켓[^fn-market]에서 원하는 사양을 구매해서 연결할 수 있다. GPU가 가능하고 세션이 계속 유지된다. Seoul 리전은 아직 보이지 않음. Vertex AI의 Workbench에서 지원하지 않음.

[^fn-market]: <https://console.cloud.google.com/marketplace/product/colab-marketplace-image-public/colab>

Transformer Chatbot[^fn-chatbot] 예제의 경우,
- 기본 제공 GPU(K80): 첫 epoch 40s, 그 다음 부터는 30s
- 기본 제공 GPU(T4): 첫 epoch 21s, 그 다음 부터는 11s
- V100 구매: 첫 epoch 13s, 그 다음 부터는 6s

[^fn-chatbot]: <https://github.com/ukairia777/tensorflow-transformer>