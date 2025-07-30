---
layout: wiki 
title: App Engine
tags: ["Cloud"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [App Engine vs Cloud Run](#app-engine-vs-cloud-run)
    - [코드](#코드)
    - [실행 및 속도](#실행-및-속도)
    - [기타](#기타)

<!-- /TOC -->

2008년에 등장하여 대표적인 PaaS로 유구한 역사를 자랑한다. 벤더 락인이 우려되어 그동안 계속 사용해보지 않았으나 2020년 GCP 서울 리전 오픈과 함께 실험해보는 중.

# App Engine vs Cloud Run
GAE Flexible은 Cloud Run과 매우 유사하다. 게다가 Flexible은 VM 기반에 Docker로 운영되므로 Cloud Run이 Dockerfile로 실행되는 점을 감안하면 거의 차이가 없다.

```
$ gcloud app create
$ gcloud app deploy
```
배포 또한 매우 단순하다. 다만, Flexible은 VM을 생성하므로 구성하는데 상당히 오랜 시간이 걸린다. 또한 flexible은 standard와 달리 VPC 설정만으로 해당 대역을 사용하기 때문에 serverless connector가 필요 없다. 설정 없이 Dockerfile만 만들어서 배포하는 Cloud Run과 달리 App Engine은 app.yaml이 필요하다.

```yaml
runtime: python
env: flex
threadsafe: true
entrypoint: gunicorn -b :$PORT main:app
service: my-second-app

runtime_config:
  python_version: 3

# This settings are to reduce costs during testing the App Engine flexible environment.
manual_scaling:
  instances: 1
resources:
  cpu: 1
  memory_gb: 0.5
  disk_size_gb: 10

network:
  name: projects/[PROJECT_ID]/global/networks/[VPC_NAME]
  subnetwork_name: [SUBNET_NAME]
```

## 코드

이외에 GAE에 맞춰 코드를 수정해줘야 한다. 다만 flask의 경우는 수정 사항이 없었고, entrypoint를 단순히 gunicorn으로 지정하기만 하면 되므로 쉽다. 하지만 Cloud Run은 bash로 쉘을 만들고 apt로 특정 유틸리티를 설치하여 python에서 실행하기도 했는데, App Engine은 이런 작업은 어렵고 제약이 있다.

## 실행 및 속도

https://xxx.du.r.appspot.com/  
Cloud Run의 *.run.app과 유사하게 *.r.appspot.com 도메인을 지원한다. https만 지원하는 Cloud Run과 달리 App Engine은 http도 지원한다. https 간에는 둘 사이의 속도차이가 거의 없지만 GAE는 http를 지원하므로 GCE에서 10배 이상 더 빠르게 실행된다. 대신 Cloud Run은 gRPC를 지원한다.

```console
gcp-user@on-prem-checker:~$ ./runme-cloud-run.sh
         time_total:  0.067664
         time_total:  0.067795
         time_total:  0.064981
gcp-user@on-prem-checker:~$ ./runme-app-engine.sh
         time_total:  0.061835
         time_total:  0.060745
         time_total:  0.060560
gcp-user@on-prem-checker:~$ ./runme-app-engine-http.sh
         time_total:  0.005134
         time_total:  0.005090
         time_total:  0.004924
gcp-user@on-prem-checker:~$ ./runme-app-engine-custom-domain.sh
         time_total:  0.332833
         time_total:  0.478309
         time_total:  0.475403
gcp-user@on-prem-checker:~$ ./runme-app-engine-standard.sh
         time_total:  0.032790
         time_total:  0.008085
         time_total:  0.009904
```

속도 측정에 사용한 `curltime`은 다음과 같다.
```bash
#!/bin/bash

curl -w @- -o /dev/null -s "$@" <<'EOF'
    time_namelookup:  %{time_namelookup}\n
       time_connect:  %{time_connect}\n
    time_appconnect:  %{time_appconnect}\n
   time_pretransfer:  %{time_pretransfer}\n
      time_redirect:  %{time_redirect}\n
 time_starttransfer:  %{time_starttransfer}\n
                    ----------\n
         time_total:  %{time_total}\n
EOF
```

flexible에 비해 standard는 cold start가 보인다. 전체적인 속도도 늦고 특히 중간중간 튀는 애들이 많아서 훨씬 더 불안정한 모습을 보인다. 아무래도 standard는 독립 인스턴스가 아니라 같이 쓰는 환경이다 보니 그런 것 같다.

## 기타

아직 asia-northeast3에서 custom domain을 지원하지 않는 Cloud Run과 달리 App Engine의 Settings에서 Custom domains를 잘 지원한다.

그러나 custom domains는 GCE에서 호출시 매우 늦다. 이 문제는 Cloud Run도 동일하게 발생하는듯 하다.[^fn-high-latency] internal microservices 용도라면 *.r.appspot.com 도메인을 이용해 직접 처리되도록 한다.

[^fn-high-latency]: <https://cloud.google.com/run/docs/issues#latency-domains>

프로젝트에서 App Engine은 하나만 구성 가능하다. 여러개는 service로 구분한다. firewall 정책이 같이 적용받기 때문에 ingress 설정으로 internal traffic and traffic from CLB로 하고, 외부 접근이 필요한 경우에만 CLB를 활용하는 방안이 좋을 것 같다. 반면 Cloud Run은 개별 설정이 가능하다.
