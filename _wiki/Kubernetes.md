---
layout: wiki 
title: Kubernetes
tags:  ["Infrastructure"]
last_modified_at: 2024/12/12 13:43:19
---

- [kubectl](#kubectl)
  - [파일 복사](#파일-복사)
  - [use-context](#use-context)
  - [Assign Pods to Nodes using Node Affinity](#assign-pods-to-nodes-using-node-affinity)
- [GKE](#gke)
  - [인증](#인증)
  - [Deployments](#deployments)
  - [Services \& Ingress](#services--ingress)
- [기타 명령](#기타-명령)
  - [상태 변경 조회](#상태-변경-조회)
  - [Kubernetes Metrics Server](#kubernetes-metrics-server)
- [Tutorial](#tutorial)
- [Istio on GKE](#istio-on-gke)
- [Helm](#helm)
  - [nginx](#nginx)
  - [Elasticsearch](#elasticsearch)
- [NodePort를 이용한 포트 노출](#nodeport를-이용한-포트-노출)

# kubectl
kubectl 설치:
```
# Linux
$ snap install kubectl --classic
# Mac
$ brew install kubectl
```

(비공개) #1-10 `~/.kube/config` 설정  

## 파일 복사
```bash
$ kubectl cp xxx sangpark-ceph-0:/home/xx/data/
```

## use-context
현재 로컬에서 사용 중인 컨텍스트 정보 조회:
```
$ kubectl config use-context local
```
설정시 `~/.kube/config` 정보가 변경된다.

조회:
```
$ kubectl config get-contexts
```

## Assign Pods to Nodes using Node Affinity
label 설정으로 간단하게 제한할 수 있다.
```bash
$ kubectl label nodes dgx01 project=semi-parametric --overwrite
```

# GKE
## 인증
인증(생성을 gcloud CLI로 할 경우 자동 인증)
```bash
$ gcloud container clusters get-credentials CLUSTER-NAME
```

Private cluster는 kubectl로 바로 접속이 안되므로 bastion vm을 이용한 proxy 작업이 필요하다. 아무런 메시지가 출력되지 않으므로 유의.

## Deployments
- 패키지 개념으로 pods replica의 묶음
- kubectl외에도 Kuberbernetes Engine > Workloads에서 DEPLOY 제공.
- Deployment details에서 ACTIONS > Expose에서는 서비스 노출(외부 접속 허용은 Load Balancer로)
- GKE에서 사용하는 pod은 K8s가 기본으로 사용하는 pod과 함께 kube-system namespace에 배포된다. GKE에서는 fluentbit, gke-metrics-agent, stackdriver 등이 있다. Istio는 istio-system에 배포(DEPRECATED)된다. 
- 전체 namespace는 다음과 같이 조회한다.
```
$ kubectl get po -A
```
- 전체 항목에 대한 조회
```
$ kubectl get po,svc,deploy -o wide
```

Cloud Run에서 사용하던 이미지를 배포하면 PORT 환경 변수에서 오류가 발생한다. 다음과 같이 YAML 추가가 필요하다.

```
spec:
  containers:
  - env:
    - name: PORT
      value: "8080"
```

cluster에 적용은,

```bash
$ kubectl apply -f redis-leader-deployment.yaml
$ kubectl apply -f redis-leader-service.yaml
```

이렇게 kubectl에서 yaml 적용으로 가능하다. 예전 DKOS v3도 이런식으로 yaml 적용으로 했던 기억. 물론 yaml은 하나의 파일로 합칠 수도 있다.

기본 VM 3대에 pod가 골고루 배치. 이 부분도 DKOS 동일. 
```
$ kubectl scale deployment frontend --replicas=5
```
pod를 늘릴 수 있으며, 물론 node를 함께 늘려줘야 의미가 있다. GKE Autopilot에서는 node 관리를 알아서 한다. 최소 2개 유지. 배포할때 트래픽이 없음에도 5개까지 늘어남.

기본 quotas가 8로 잡혀 있다. GPU로 마찬가지로 요청해서 늘리는 구조인듯.

```bash
$ kubectl get no
```

노드 삭제는 kubectl에서 하거나 콘솔에서 terminate 하면 깔끔하게 정리된다. pod 전체를 삭제하는건 `$ kubectl delete -f polls.yaml` 이렇게 yaml로 하거나 `$ kubectl delete deployment polls` 이렇게 깔끔하게 deployment 단위로 정리 가능.

## Services & Ingress

Ingress 설정을 하려면 NodePort, 즉시 외부 오픈은 LoadBalancer로 설정한다.

ClusterIP는 pod 내에서 다음과 같이 조회를 시도할 수 있다.

```bash
$ kubectl get pods
$ kubectl exec -it [POD-NAME] -- sh
$ apk add --no-cache curl
$ curl "http://[CLUSTER-IP]:80"
```
LoadBalancer는 각 클라우드의 기능을 이용한다. 원래 K8s 자체는 `<pending>`으로 진행되지 않음.

LoadBalancer로 설정시 방화벽 0.0.0.0에 80포트 등을 오픈하는 정책이 자동으로 추가된다고 하는데(http-server 태그가 추가되는거 아닌지) 확인이 필요하다.

Ingress는 Services 상위 개념. 트래픽을 Ingress의 도메인이 받아서 TLS 지원 하고, path based routing으로 각 Service에 연결한다. Ingress는 헬스 체크도 진행하는데, 로그인 처리로 루트를 302로 처리했더니 헬스 체크에서 실패한다. 임의로 헬스 체크를 다른 경로로 지정 후에야 unhealthy 상태가 사라지고 정상적으로 연결 가능. Ingress만 문제가 생기는 것으로 봐서 LoadBalancer는 HTTP Status 헬스 체크는 하지 않는 것으로 보인다.

# 기타 명령
```bash
$ kubectl api-resources
```
조회 가능한 resources 목록. shortnames도 확인할 수 있다. 다음과 같이 조회를 shortnames로 가능.
```bash
$ kubectl get no
$ kubectl get svc
$ kubectl get po
$ kubectl get deploy
```

## 상태 변경 조회
```bash
$ kubectl get pod --watch
# kubectl get pod -l app=wordpress --watch
```
`--watch` 옵션을 부여하면 상태가 변경될때 다시 출력된다. 특정 앱만 지정하려면 `-l` 옵션으로 부여. `$ watch -n 1 [CMD]`보다 매 번 껌뻑임 없이 변경될때만 출력해주어 편리하다.

## Kubernetes Metrics Server
NOTICE: GKE에는 설치되어 있기 때문에 추가로 설치할 필요가 없다. (추가 설치시 Authorization 오류 발생했음)
시스템 사용량 조회
```console
# 설치
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Helm을 이용한 설치
$ helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
$ helm upgrade --install metrics-server metrics-server/metrics-server
```

삭제는 다음과 같이,
```console
$ kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

`$ kubectl top no`를 넘어선 시각화 모니터링 툴 k9s 설치:
```console
$ brew install k9s
```

# Tutorial

[GKE Tutorial 과정](https://cloud.google.com/kubernetes-engine/docs/tutorials) 유익하다.
- Deploying a containerized web application  
Go로 multi-stage builds 결과물을 kubectl, console 모든 방식으로 보여준다. console 만으로도 충분히 배포 가능. 새 버전 배포는 rolling update로 하는 방식을 보여준다.
- Create a Guestbook with Redis and PHP  
redis-leader / redis-follower 구조로 read가 많을때 효율적. PHP에서 get은 redis-follower에 요청을, set은 redis-leader로 요청을 하는 댓글창을 만든다. 내부 시스템은 ClusterIP 설정 만으로. 내부적으로 kube-dns가 동작한다.
- Deploying WordPress on GKE with Persistent Disks and Cloud SQL  
코드 전혀 없이 공식 이미지로 설치 배포 과정을 보여줌. Cloud SQL MySQL 생성. PV 사용을 보여준다. 그런데 사이트 접속이 상당히 늦다. static file 조차도 느린걸 보면 네트워크 문제로 보이는데, 원인 파악은 하지 못함. 위에 redis/php 예제는 엄청 빠른데 왜 느린지 의문.
- Deploying Memcached on Google Kubernetes Engine  
이건 helm 사용하는 부분에서 명령이 동작하지 않아서 시도하다가 중간에 그만둠. 가이드에서 helm 버전이 2.x으로 옛날 버전이다. helm을 사용하면 복잡한 yaml에서 해방될 수 있다고 하나 아직 익숙하지 않다.
- Running Django on Google Kubernetes Engine  
Cloud SQL에서 PostgreSQL를 사용하고, Cloud SQL Proxy를 이용해 로컬에서도 테스트가 가능하도록 한다. 최종 서비스는 Django로 구성해 gunicorn으로 서비스 하고 static 파일은 별도로 GCS에 public으로 서비스하도록 구성한다. ADD NODE POOL도 함께 실험해봤는데, 문제 없이 잘 된다. 그런데 `kubectl apply -f polls.yaml`은 `Does not have minimum availability` 오류 발생. CloudSQL 인증을 해야 실행된다. 오류가 마치 리소스 부족인것 처럼 명확하지 않아 원인을 찾는데 시간을 많이 소모함.
```bash
$ kubectl create secret generic cloudsql \
--from-literal=username=poll-user \
--from-literal=password=poll-user
```
삭제할때는,
```bash
$ kubectl delete secret cloudsql
```

# Istio on GKE
> A service mesh, like the open source project Istio, is a way to control how different parts of an application share data with one another.

Tutorial 참고[^fn-istio] 여타 설정 제외하고 가능한 default로 시도해봄.

[^fn-istio]: <https://cloud.google.com/istio/docs/istio-on-gke/installing#command-line>

```console
$ gcloud beta container clusters create demo-cluster \
    --addons=Istio --istio-config=auth=MTLS_PERMISSIVE \
    --num-nodes=4
```

설치 여부 조회
```console
$ kubectl get svc -n istio-system
$ kubectl get po -n istio-system
```

# Helm
```console
# 공식 레포
$ helm repo add stable https://charts.helm.sh/stable

# 검색
$ helm search repo elasticsearch
```
## nginx
```console
$ helm install bitnami/nginx --generate-name --create-namespace -n nginx-system
# <pending>에서 다소 시간이 1분 정도 있으면 svc에 external ip가 등록된다.
$ kubectl get svc -n nginx-system
```
삭제:
```console
$ helm uninstall nginx-1650011765 -n nginx-system
```
Scaling resources
```bash
$ kubectl scale --replicas=3 deployment.apps/nginx-1650011765
```

## Elasticsearch
Elasticsearch는 다음과 같이 설치할 수 있다.
```console
# Elastic repo 추가
$ helm repo add elastic https://helm.elastic.co
$ helm repo update

# namespace 생성
$ kubectl create ns elastic-system

# 설치
$ helm install elasticsearch elastic/elasticsearch -n elastic-system
# or
$ helm install elasticsearch elastic/elasticsearch -n elastic-system --create-namespace
# or
$ helm install elastic/elasticsearch --generate-name
```

GKE Autopilot에서는 `container configure-sysctl is privileged; not allowed in Autopilot` 오류가 발생하며 설치되지 않는다. Autopilot에서는 아직 설치 안되는 helm 차트가 많다. Standard로 다시 생성해 deployed가 되면 master 3대가 배포되면서 다음과 같이 모니터링 하라는 가이드가 제공된다.

```console
# 1. Watch all cluster members come up.
$ kubectl get pods --namespace=elastic-system -l app=elasticsearch-master -w

# 2. Test cluster health using Helm test.
$ helm --namespace=elastic-system test elasticsearch
```

Autoscaling 설정이 되어 있지 않고, Insufficient CPU로 Pending에서 진행되지 않는 오류가 발생한다. 
```
$ kubectl describe po elasticsearch-master-0 -n elastic-system
```
describe에서 오류 메시지를 확인할 수 있고, GKE 콘솔에서도 Unscheduled tasks라고 출력된다. Enable Autoscaling으로 설정을 변경하고, 수동으로 node pool을 추가하면서 호스트도 늘려보았으나 이로 인해 Repairing the cluster 상태에 들어가고, kubectl 접근조차 되지 않는다. 만약 실서비스라면 문제가 될것 같다.

기본 e2-medium은 시스템 사양이 낮은 문제여서 add node pool을 하여 머신 타입을 n2-standard-4로 올려서 3대 추가하니 이제 Pending에서 Running으로 동작한다. 그러나 이내 Repairing the cluster로 다시 들어간다. 이때는 kubectl 접속이 중단되므로 문제가 있다. 3분 정도 후 자동 복구됐다.

```
# Kibana 설치
$ helm install elastic/kibana -n elastic-system --generate-name

# Port-forwarding 설정, pods replica 묶음인 패키지에 대한 설정
$ kubectl port-forward deployment/kibana-kibana 5601 -n elastic-system
```

localhost:5601로 Kibana 접속이 가능하다. 차트 배포 자체는 로컬에서 한 줄로 가능하므로 매우 편리하다. 컨테이너에 직접 접속해 호출하는 형태로 디버깅도 가능하다.

```console
$ kubectl exec elasticsearch-master-0 -c elasticsearch -n elastic-system -- curl http://localhost:9200
```

# NodePort를 이용한 포트 노출
```
$ kubectl expose pod xxx --type=NodePort --port=18888 --target-port=80
$ kubectl describe svc xxx
NodePort:                 <unset>  30211/TCP
$ curl http://192.168.100.11:30211
```
컨테이너 내부 80을 18888로 맵핑. 이후 NodePort가 30211로 자동 설정되며 접속 가능하다.

보안 문제로 다음과 같이 해제 필요:
```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
 name: allow-all
 namespace: slm
spec:
 rules:
 - {}
```
`$ kubectl apply -f rule.yaml`로 권한 처리 필요. 1회만 하면 된다.
