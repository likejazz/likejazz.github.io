---
layout: wiki 
title: Elasticsearch 설치
tags: ["Information Retrieval"]
last_modified_at: 2022/05/12 02:37:38
---

<!-- TOC -->

- [개요](#개요)
- [설치](#설치)
  - [Enterprise Search 별도 정리](#enterprise-search-별도-정리)
  - [시스템 및 비용](#시스템-및-비용)
    - [OpenSearch](#opensearch)
    - [Elastic Cloud](#elastic-cloud)
  - [Enterprise Search](#enterprise-search)
  - [분산환경](#분산환경)
- [네트워크](#네트워크)
- [인덱스 및 로그](#인덱스-및-로그)
- [Elasticsearch 플러그인과 JDK 버전](#elasticsearch-플러그인과-jdk-버전)
- [Elastic Cloud에 Extensions 설치](#elastic-cloud에-extensions-설치)

<!-- /TOC -->

# 개요
Lucene은 인덱스 사이즈가 크고 압축 효율이 낮으며 시스템이 무겁고 느리다는 단점이 있지만 그럼에도 불구하고 es, solr등 좋은 엔진을 기반으로 성공적으로 안착.

1. es의 분산 시스템 scale-out 확장 구조
2. 다양한 생태계(구글/네이버/카카오의 in-house 엔진이 갖지 못한 장점)
3. REST를 넘어(solr도 REST는 지원) Query DSL 지원, 5.0부터 Aggregation API 대폭 강화.
4. Kibana 엔진 운영 및 시각화(solr 운영툴 써보면 ...) Logstash 데이터 파이프라인 표준화, 전체 플랫폼 아키텍처 설계에 큰 기여를 한다.
  - logstash는 배치 처리와 병렬 처리, 큐를 이용한 이벤트 1회 전송 보장, 데이터 부하에도 안정성 보장.

hyper-scale이 아닌 이상 es가 최선의 선택. 이외 C++로 구현하고 속도가 빠른 Typesense, SaaS만 제공되는 Angolia, Rust로 구현했지만 멀티 노드가 아직 지원되지 않는 MeiliSearch가 있다. 

# 설치
설치 표준화를 위해 docker를 기준으로 설치한다. 

- elasticsearch의 [Dockerfile](https://github.com/elastic/dockerfiles/blob/7.16/elasticsearch/Dockerfile)  
ubuntu 20.04 기반
- kibana의 [Dockerfile](https://github.com/elastic/dockerfiles/blob/7.16/kibana/Dockerfile)  
centos 8 기반. es는 apt, kibana는 yum 이므로 유의

7.x에서는 docker-compose로 여러 설정을 해서 구동했는데, 8.x에서 terminal attached 확인이 되어야 보안 설정이 출력된다. (엄청 불편) `data`에 아무것도 없어야 초기화가 진행된다. 데이터에 내용이 있으면 production mode로 동작하니 주의. 이 경우 bootstrap check를 진행한다.

```bash
$ docker run -p 9200:9200 \
-e ES_JAVA_OPTS="-Xms1200m -Xmx1200m" \
--net xxx_elastic \
-it elasticsearch
```

7.x과 달리 8.x에서는 모든 옵션을 제외하고 default로 구동. https 필수(로컬에서는 인증서 받아서 호출), elastic 계정 패스워드 자동생성, kibana enrollment token 제공 등이 모두 자동으로 진행된다. 이 부분은 편하다. 하지만 초기화에 버그[^fn-bug]가 있으니 주의(terminal attached가 확인되어야 하는데 매우 불편, 그래서 `-it`가 매우 중요. docker-compose는 안됨):

[^fn-bug]: <https://github.com/elastic/elasticsearch/pull/83566>

> If you installed Elasticsearch from an archive on an aarch64 platform like Linux ARM or macOS M1, the elastic user password and Kibana enrollment token are not generated automatically when starting your node for the first time.

https 인증서 verify 없이 요청:
```
$ curl -k -u elastic:xxxx https://localhost:9200
또는
$ curl --insecure \
-u elastic:XXX \
https://ec2-XXX.ap-northeast-2.compute.amazonaws.com:9200
```
`-k`와 `--insecure`는 같은 옵션이다.

백그라운드 구동과 로그 조회는 다음과 같이 했다. (8.x에서 더 이상 사용할 수 없음)
```bash
docker-compose up -d

CID=`docker ps | grep elasticsearch | awk '{print $1}'`
docker logs -f $CID | gnomon
```

## Enterprise Search 별도 정리
- enterprise search는 오픈소스가 아니다. dockerfile도 공개되어 있지 않음. 컨테이너 내부에는 jruby와 rails로 구성한 흔적이 있다. (한글 지원이 되지 않고 ranking customization이 오히려 더 힘들어 결국 사용하지 않음)
- heap 사이즈를 늘려주지 않으면 macOS에서 ent search가 동작하지 않아서 한동안 고생했다. 마찬가지로 es도 heap 사이즈를 조절하지 않으면 동작이 느려져서 동일하게 ent search가 접속을 못했다. 비슷한 질문이 딱 하나만 있고 다른 사례가 보이지 않는걸로 봐서 아마 macOS에서 리소스 제한(내 경우 1 CPU, 2G 메모리)으로 사용할 경우 발생하는 문제로 보인다. 
  - seunjeon을 설치할 경우 512m에서 out of memory 발생. 최종적으로 1200m으로 설정.
- ent search가 localhost로 redirect 되는 문제가 있어 다음과 같이 설정을 추가했으나 반영되지 않았다.
```
ent_search.external_url: http://XX.compute.amazonaws.com:3002
```
다른 설정과 동일하게 `:`를 `=`로 변경 후 반영됐다. 원래 반영이 되어야 하는데, 다른 설정과 달라서 인지 띄어쓰기 때문인지 원인 확인 필요.

## 시스템 및 비용
ent search가 es에 붙는데, 맥북에서 80초가 걸렸고, ec2 c6i.large에서 33초가 걸렸다. 약 3배 더 빠르다. 서울 리전에서 가격은 다음과 같다.

| Type | Instance | Pricing |
| ---- | -------- | ------- |
| EC2 | c6i.large(compute optimized) | $0.096 |
| EC2 | m6g.large | $0.094 |
| OpenSearch | m6g.large | $0.156(사실상 OpenSearch 중에는 가장 저렴) |
| OpenSearch | r6g.large(memory optimized) | $0.203 |
| Elastic Cloud | 120 GB storage / 4 GB RAM / Up to 2.2 vCPU (2 zones, Storage optimized, AWS Seoul, Gold) | $0.303 |
| Elastic Cloud | 10 GB storage / 1 GB RAM (Autoscale, Platinum) | $0.057 / $0.191 (2 zones) |

### OpenSearch
OpenSearch는 S3 통해 사전 추가는 가능하나(패키지라는 이름으로) 플러그인은 설치 불가. AWS 대시보드에서 인덱스 조회가 바로 가능하고 무엇보다 VPC 안으로 넣을 수 있어 보안에 유리하다. 이 경우 VPC내 서버만 접근 가능.

### Elastic Cloud
신용카드 없이도 가입이 가능하며 14일간 묻지도 않고 기본 무료 제공. AWS Marketplace에서 subscribe하면 비용이 AWS 계정으로 통합 청구도 가능하다. GCP에서는 리셀러 통한 구매는 되지 않는다며 subscribe 되지 않았음.

Elastic Cloud는 클라우드지만 Server 구성을 구체적으로 보여주고, configuration 변경시 과정을 대시보드에 실시간으로 보여주기 때문에 Serverless 보다는 운영 자동화에 더 가깝다.

주의: Elastic Cloud 플러그인은 Gold 이상에서만 가능하고 이 마저도 20MB 제한이 있다. Platinum을 해야 3G로 늘려줌. 사실상 플러그인을 쓰려면 Platinum을 할 수 밖에 없다.

## Enterprise Search
Elastic이 Swiftype을 2017년에 인수하고 2018년 5월, 이를 기반으로 Elastic App Search을 출시했다. 그리고 Elastic Enterprise Search 제품군으로 통합됐다. 운영툴이 거의 동일하다. optimized relevance for search use cases, typo-tolerance, relevance tuning등을 제공한다. relevance score가 기존 es와 다르고, 파라미터를 보여주진 않는다. 그러나 결정적으로 한글 검색을 지원하지 않기 때문에 쓸 수가 없다. Site Search는 crawler가 포함되어 있고, Workspace Search는 slack, teams 같은 내부 자산을 검색한다.

## 분산환경
분산환경에서는 이론상 데이터의 크기가 무한대로 늘어날 수 있기 때문에 많은 고민이 필요하다. 분산환경, 정확도, 실시간성 세 가지 요소가 CAP theorem처럼 동작한다.

# 네트워크 
서로 통신하기 위해 사용자 정의 bridge 네트워크 생성. 컨테이너의 name server는 컨테이너 내에서 다음과 같이 조회할 수 있다.

```console
$ cat /etc/resolv.conf
nameserver 127.0.0.11
options ndots:0
```

컨테이너에 `docker inspect 5a21533841b9`를 해보면
networks 정보 중에 host aliases가 보인다.

```json
"Networks": {
    "dir-name_elastic": {
        "Aliases": [
            "dir-name_elasticsearch_1",
            "elasticsearch",
            "5a21533841b9"
        ],
```

# 인덱스 및 로그
Elastic Cloud에서는 Kibana Monitoring Stack에서 아무것도 보이지 않고 Logs and metrics에서 enable해야 한다. cloud에서는 별도 모니터링 전용 클러스터로 shipping 권장.
- Logs: `elastic-cloud-logs`는 cloud에서는 Logs로, 여기서는 주로 gc log다. 기본 설정은 30일 50G rollover로 되어 있다. rollover 이후에도 삭제는 하지 않는 것으로 보인다.
- Metrics: `.monitoring-es`는 시스템 모니터링 로그 인덱스로 7일간 유지되며 이후 삭제된다. on-prem은 기본으로 이것만 enabled. xpack 설정으로 enable/disable 가능. cloud에서 이것만 enable했더니 보이지 않는데 버그로 보인다. Logs와 함께 enable해야 보인다.

모니터링 클러스터에는 es와 kibana만 남기고 apm이나 ent search는 모두 삭제했다.

# Elasticsearch 플러그인과 JDK 버전
es 8 이상에서는 JDK 17가 default다. 플러그인을 빌드 하려면 JDK 17이 필요하다. gradlew가 예전 버전으로 생성되어 있다면 `$ gradle wrapper`로 재생성이 필요하다. 당연히 Project SDK도 17로 올려야 하고, IntelliJ에서 Gradle JVM 설정도 Project SDK로 맞춰야 실행 문제가 없다. default는 JAVA_HOME으로 되어 있으므로 주의.

# Elastic Cloud에 Extensions 설치
Extension 등록할 때 맥에서 압축한 zip 올리면 인식을 못한다. CLI에서 zip으로 압축하고서야 정상적으로 인식. 아마 서버측에서 Extension Validation 체크를 Linux에서 하는 것으로 보인다. 하필 마지막 배포버전이 맥에서 압축한 상태로 올린거라 이게 Elasticsearch에서는 정상적으로 설치되는데, Cloud에서만 오류로 나오는 바람에 한참 시간을 허비했다.