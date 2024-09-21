---
layout: wiki 
title: Kubeflow
tags: ["MLOps & HPC"]
last_modified_at: 2024/09/21 13:45:27
---

<!-- TOC -->

- [설치](#설치)
- [GPU](#gpu)
- [조회](#조회)
- [특징](#특징)
- [예제](#예제)
  - [End-to-End kubeflow tutorial using a Pytorch model in Google Cloud](#end-to-end-kubeflow-tutorial-using-a-pytorch-model-in-google-cloud)
  - [Ames housing value prediction using XGBoost on Kubeflow](#ames-housing-value-prediction-using-xgboost-on-kubeflow)
  - [Simple GPU Pipeline](#simple-gpu-pipeline)
- [용도](#용도)

<!-- /TOC -->

# 설치
Standalone Deployment[^fn-stand] on GKE로 설치 진행:
```console
$ export PIPELINE_VERSION=1.8.1
$ kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/cluster-scoped-resources?ref=$PIPELINE_VERSION"
$ kubectl wait --for condition=established --timeout=60s crd/applications.app.k8s.io
$ kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/env/dev?ref=$PIPELINE_VERSION"

# Kubeflow Pipelines UI
$ kubectl describe configmap inverse-proxy-config -n kubeflow | grep googleusercontent.com
```

[^fn-stand]: <https://www.kubeflow.org/docs/components/pipelines/installation/standalone-deployment/>

# GPU
GKE에 install the Nvidia driver on any GPU-enabled cluster nodes:
```console
$ kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/container-engine-accelerators/master/nvidia-driver-installer/cos/daemonset-preloaded.yaml
```

GPU pod 로그 조회:
```console
$ kubectl logs -f nvidia-gpu-device-plugin-kfmq9 -n kube-system
```

MySQL 설치가 진행되지 않는 오류가 발생했다.
```
mysql-f7b9b7dd4-j2g5f                              0/1     Pending            0          69s
```
새 클러스터에서 진행하니 이번에는 정상적으로 Running 된다.

# 조회
```console
$ kubectl get svc,po,deploy -n kubeflow
```

Pipeline을 실행하면 K8s에 Pod이 생성되었다가 Completed 되는 모습을 확인할 수 있다. 

UI에서 Runs > Graph에 학습 도중 실시간으로 반영된다.

# 특징
모델 관리 보다는 패러럴리즘에 포커싱되어 있으며, 코드도 이에 맞춰 작성.  
아래는 Kubeflow Pipelines UI의 Pipelines 항목 예제:  
<img src="https://user-images.githubusercontent.com/1250095/164163790-f169870c-da93-40d1-80ac-abda164f2c95.png" width="80%">

tfx 예제의 경우 기본 GKE 클러스터는 리소스가 부족해 scale-up이 필요하다. Node auto-provisioning(사용자를 대신해 노드 풀 관리)을 Enabled로 설정하고 Maximum을 10으로 지정했다. 또한 Vertical Pod Autoscaling을 Enabled 했다.

기본 e2-medium은 스펙이 너무 낮아서(Elasticsearch가 설치 안되던 문제와 동일) n2-standard-4로 클러스터를 신규 생성했다. 4CPU/14GB로 구성되어 있다. tfx 예제는 1.5CPU/3.5GB로 요청이 들어간다.

GPU 노드(T4) n1-standard-4(4CPU/13GB)에서 scale up 없이 바로 실행된다. GPU 문제로 us-central1-a에서 생성. 그러나 gcs bucket 설정 문제로 파이썬 오류가 발생하여 실해오디지 않는다.

# 예제
## End-to-End kubeflow tutorial using a Pytorch model in Google Cloud

[^fn-mnist]

[^fn-mnist]: <https://github.com/kubeflow/examples/tree/master/pytorch_mnist>

There are two primary goals for this tutorial:
- Demonstrate an End-to-End kubeflow example
- Present an End-to-End Pytorch model

By the end of this tutorial, you should learn how to:
- Setup a Kubeflow cluster on a new Kubernetes deployment
- Spawn up a shared-persistent storage across the cluster to store models
- Train a distributed model using Pytorch and GPUs on the cluster
- Serve the model using Seldon Core
- Query the model from a simple front-end application

가이드[^fn-mnist]에 따라 진행하려 했으나 오래된 프로젝트라 진행이 어려움

- asia-northeast1-a(Tokyo) Tesla T4 노드 구성
  - 예제에서 ksonnet 필요. 설치시 Xcode 업데이트 필요.

## Ames housing value prediction using XGBoost on Kubeflow

[^fn-ames]

[^fn-ames]: <https://github.com/kubeflow/examples/tree/master/xgboost_ames_housing>

- `s2i` 설치 필요. `$ brew install source-to-image`
- docker build 오류 발생 및 s2i 빌드도 마찬가지 오류 발생:
```console
$ s2i build . seldonio/seldon-core-s2i-python2:0.4 gcr.io/${PROJECT_ID}/housingserve:latest --loglevel=3
```

```
E0421 14:29:19.610652   45759 errors.go:296] An error occurred: non-zero (13) exit code from seldonio/seldon-core-s2i-python2:0.4
```

## Simple GPU Pipeline
[^fn-gpu]

[^fn-gpu]: <https://github.com/kubeflow/examples/tree/master/demos/simple_pipeline>

간단한 pipeline 샘플. 가이드대로 `kfp`를 설치하고 소스를 실행하면 tar.gz가 나오고 이를 kubeflow pipeline으로 등록하면 된다. GPU 예제로, 노드에 GPU가 없다면 실행되지 않는다. 멀티 노드가 아니라 하나의 노드만 동작한다.

# 용도
notebooks를 통해서 pod을 생성하고 노드에 deploy하는 역할을 WebUI로 제공한다. 기존 `kubectl apply`의 대체제 역할을 한다.