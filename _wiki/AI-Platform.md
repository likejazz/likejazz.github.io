---
layout: wiki 
title: AI Platform
last-modified: 2020/09/05 01:50:42
---

<!-- TOC -->

- [가이드](#가이드)

<!-- /TOC -->

# 가이드
ml-engine에서 이름이 바꼈다. [공식 가이드 문서](https://cloud.google.com/ai-platform/prediction/docs/getting-started-scikit-xgboost)가 가장 정확. 예전 블로그는 outdated.

us-central1만 지원해서 모델을 이쪽으로 업로드 하도록 별도 생성. input-type이 json인데, 특이한 형태로 받고 있어서 `gcloud ai-platform predict --help` 도움말로 확인 필요. new line으로 입력값을 구분한다. 바로 모델을 업로드 하니 잘 안되는데, `local predict` 기능이 있어서 여기서 먼저 잘 되는지 확인할 수 있다. 매우 편리. 실제로 로컬 모듈을 활용하기 때문에, tf도 설치되어 있어야 한다.

xgboost를 sckit-learn interface로 했다. 이 경우 framework이 XGBOOST인지 SCIKIT_LEARN인지도 애매하다. (xgboost가 직접 구동되어야 하니 당연히 XGBOOST이다.) xgboost DMatrix로 모두 변환. 기존에 XGBClassifier()가 자동으로 wrapping해서 처리해주는 것들이 많았는데, 전부 직접 해야 해서 매우 번거롭다. 그러나, 덕분에 early stopping 처리도 하고, LabelEncoder()도 직접 구현할 수 있었다. 이제 ai-platform에서 잘 동작한다. 다만, python/java만 제공해서 웹 프론트를 python으로 변경해야 한다. 다시 해보니 기존 scikit-learn wrapper도 잘 동작한다. 결과를 보면 `multi:softprob`로 되어 있다. 그래서 n개 중에 확률 높은 하나를 직접 처리해주는 작업이 필요함. 기존에 wrapper가 이 작업을 해줬다.

```
{"instances":[[46042,276,32.5,28.5,22.5,22.5,10,1,1,1]]}
```
