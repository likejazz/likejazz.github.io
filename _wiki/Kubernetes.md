---
layout: wiki 
title: Kubernetes
last-modified: 2020/09/20 03:55:41
---

<!-- TOC -->

- [Kubernetes](#kubernetes)
    - [Cluster 생성/삭제](#cluster-생성삭제)
    - [기타 명령](#기타-명령)
        - [상태 변경 조회](#상태-변경-조회)
    - [Tutorial](#tutorial)

<!-- /TOC -->

# Kubernetes
GKE 과정 정리

먼저, kubectl을 사용하기 위한 인증
```bash
$ gcloud container clusters get-credentials hello-cluster --zone asia-northeast3-b --project edith-xxx
Fetching cluster endpoint and auth data.
kubeconfig entry generated for hello-cluster.
$ kubectl get service
```

ClusterIP (default)로 expose하면 당연히 외부에서는 접근할 수 없다. 같은 네트워크 대역에서도 접속이 안된다. cluster 내에서만 접속 가능. 외부는 LoadBalancer로 설정. 이 경우 인증 없이 외부에 오픈되므로 주의.
```bash
$ kubectl get pods
$ kubectl exec -it [POD-NAME] -- sh
$ apk add --no-cache curl
$ curl [CLUSTER-IP]
```
LoadBalancer는 GCP의 기능을 이용하는데, GKE 지원 기능으로 보인다.
```bash
$ gcloud compute forwarding-rules list
```
위 명령으로 Load Balancing 조회 가능. k8s cluster를 삭제하니 함께 삭제된다.

cluster에 적용은,
```bash
$ kubectl apply -f redis-leader-deployment.yaml
$ kubectl apply -f redis-leader-service.yaml
```
이렇게 kubectl로 Console 사용하지 않고(터미널이 아닌 GKE 콘솔 의미) yaml 적용으로 바로 pod/service 적용 가능하다. 예전 DKOS v3도 이런식으로 yaml 적용으로 했던 기억. 물론 yaml은 하나의 파일로 합칠 수도 있다.

기본 VM 3대에 pod가 골고루 배치. 이 부분도 DKOS 동일. `$ kubectl scale deployment frontend --replicas=5` 하면 pod가 늘어난다. 물론 VM을 함께 늘려줘야 의미가 있다. VM 증설은 Workloads에서 ADD NODE POOL로 가능. 해보니까 원래 디폴트는 n1-standard-1 새롭게 추가한건 e2-medium이다. 기존 노드를 EDIT 하면 동일 구성으로 늘릴 수 있다. VM 하나를 다 차지하면서도 CPU requested 661 mCPU, Memory 346.03 MB 라고만 표시된다. 나머지 리소스는 언제 쓴다는 것인지 궁금.

기본 quotas가 8로 잡혀 있다. GPU로 마찬가지로 요청해서 늘리는 구조인듯.

```bash
$ kubectl get no
```

노드 삭제는 kubectl에서 하기 보다 그냥 콘솔에서 terminate 하니까 가장 깔끔하게 정리됐다. pod 전체를 삭제하는건 `$ kubectl delete -f polls.yaml` 이렇게 yaml을 지정해서 일괄 삭제했다. 또는 배포 삭제는,

```bash
$ kubectl delete deployment polls
```

## Cluster 생성/삭제
인증(생성도 gcloud로 할 경우 자동으로 인증된다)
```bash
$ gcloud container clusters get-credentials polls-cluster \
  --zone "asia-northeast3-b"
```

생성
```bash
$ gcloud container clusters create polls-cluster \
  --num-nodes 3 --zone "asia-northeast3-b"
```

삭제
```bash
$ gcloud container clusters delete polls-cluster \
  --zone "asia-northeast3-b"
```

## 기타 명령
```
$ kubectl api-resources
```
조회 가능한 resources 목록. shortnames도 확인할 수 있다. 다음은 nodes 조회
```
$ kubectl get no
```

### 상태 변경 조회
```bash
$ kubectl get pod --watch
# kubectl get pod -l app=wordpress --watch
```
`--watch` 옵션을 부여하면 상태가 변경될때 다시 출력된다. 특정 앱만 지정하려면 `-l` 옵션으로 부여. `$ watch -n 1 [CMD]`보다 매 번 껌뻑임 없이 변경될때만 출력해주어 편리하다.

## Tutorial

[GKE Tutorial 과정](https://cloud.google.com/kubernetes-engine/docs/tutorials) 유익하다.
- Deploying a containerized web application  
Go로 multi-stage builds 결과물을 kubectl, console 모든 방식으로 보여준다. console 만으로도 충분히 배포 가능. 새 버전 배포는 rolling update로 하는 방식을 보여준다.
- Create a Guestbook with Redis and PHP  
redis-leader / redis-follower 구조로 read가 많을때 효율적. PHP에서 get은 redis-follower에 요청을, set은 redis-leader로 요청을 하는 댓글창을 만든다. 내부 시스템은 ClusterIP 설정 만으로. 내부적으로 kube-dns가 동작한다.
- Deploying WordPress on GKE with Persistent Disks and Cloud SQL  
코드 전혀 없이 공식 이미지로 설치 배포 과정을 보여줌. Cloud SQL MySQL 생성. PV 사용을 보여준다. 그런데 사이트 접속이 상당히 늦다. static file 조차도 느린걸 보면 네트워크 문제로 보이는데, 원인 파악은 하지 못함. 위에 redis/php 예제는 엄청 빠른데 왜 느린지 의문.
- Deploying Memcached on Google Kubernetes Engine  
이건 helm 사용하는 부분에서 명령이 동작하지 않아서 시도하다가 중간에 그만둠. helm을 사용하면 복잡한 yaml에서 해방될 수 있다고 하나 익숙하지 않음.
- Running Django on Google Kubernetes Engine  
Cloud SQL에서 PostgreSQL를 사용하고, Cloud SQL Proxy를 이용해 로컬에서도 테스트가 가능하도록 한다. 최종 서비스는 Django로 구성해 gunicorn으로 서비스 하고 static 파일은 별도로 GCS에 public으로 서비스하도록 구성한다. ADD NODE POOL도 함께 실험해봤는데, 문제 없이 잘 된다. 그런데 `kubectl apply -f polls.yaml`은 `Does not have minimum availability` 오류 발생. CloudSQL 인증을 해야 실행된다. 오류가 마치 리소스 부족인것 처럼 명확하지 않아 원인을 찾는데 시간을 많이 소모함.
```
$ kubectl create secret generic cloudsql \
--from-literal=username=poll-user \
--from-literal=password=poll-user
```
삭제할때는,
```
$ kubectl delete secret cloudsql
```
