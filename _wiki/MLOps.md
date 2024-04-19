---
layout: wiki 
title: MLOps
tags: ["MLOps & HPC"]
last_modified_at: 2022/05/26 17:52:44
---

<!-- TOC -->

- [MLflow](#mlflow)
- [Seldon Core](#seldon-core)

<!-- /TOC -->

# MLflow
데이터브릭스에서 만든 ML lifecycle 플랫폼. 모델 실험 결과 및 호스팅, 직접 서빙도 가능하다.
```
$ mlflow models serve --no-conda -m runs:/b5d0ba60635f4e50a63041e128ce529f/model -p 400
```
기본으로 conda 새환경을 구성하는데 conda가 너무 느리기 때문에 `--no-conda`로 구동. azure와 sagemaker는 deploy도 가능하다. pycaret에서 지원.

# Seldon Core
sklearn serving을 찾다가 발견했는데 MLflow serving도 모두 지원하는 모습에 감동. 당연히 tf, xgb도 지원한다. 생소한 런던 기반 스타트업이고 레퍼런스가 많지 않을거 같았는데, 의외로 SKT 내부 Vision API에서 사용중인듯 하다. GKE에서 GPU 배포[^fn-seldon]도 지원한다. 가이드 확인 필요.

[^fn-seldon]: <https://docs.seldon.io/projects/seldon-core/en/latest/examples/gpu_tensorflow_deep_mnist.html>