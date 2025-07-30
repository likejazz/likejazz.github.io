---
layout: wiki 
title: AWS
tags: ["Cloud"]
last_modified_at: 2025/06/08 19:28:24
---

<!-- TOC -->

- [계정](#계정)
- [개요](#개요)
- [EC2](#ec2)
  - [GPU instances](#gpu-instances)
  - [Azure V100](#azure-v100)
- [서비스](#서비스)
  - [RedShift](#redshift)
  - [RDS](#rds)
  - [S3](#s3)
  - [Athena](#athena)
- [설치 및 링크](#설치-및-링크)
  - [도메인 설정 및 ELB 맵핑 과정](#도메인-설정-및-elb-맵핑-과정)

<!-- /TOC -->

# 계정
유닉스 root 같은 루트 계정이 있고 이 하위에 IAM 계정을 등록하여 사용한다. 유닉스와 마찬가지로 루트 직접 사용은 권장하지 않고 IAM을 등록하여 사용한다.

multiple AWS accounts는 다음과 같이 설정한다.
```
$ aws configure --profile=anotheruser
```

이후 `awless`를 이용해 조회할 수 있다.
```
$ AWS_PROFILE=anotheruser awless ls instances
```

설정은 `~/.aws`에 위치하며 단순 텍스트 파일이라 GCP와 달리 개별 조회가 가능하다.

# 개요
IAM, S3, VPC - ELB, EC2, RDS, Bridge, Auto Scaling, CloudFront, Route 53, CloudWatch  
(예제를 통해 쉽게 따라하는 아마존 웹 서비스, 2017)

- IAM에서 User는 Group으로 관리, Role은 OTP기반으로 동작하는 권한이다. 이미 설정된 Managed Policy를 권장하며 AWS에서 미리 정의했다.
- Subnet은 외부 접근이 허용된 Public과 차단된 Private이 있다.
- Elastic IP로 고정 IP를 받을 수 있다. Bastion Host는 gateway의 역할을 한다. 
- MySQL compatible한 Amazon Aurora DB를 권장한다. 
- ELB는 DNS만 제공. IP가 자주 변경된다. https도 지원한다.

(서비스 운영이 쉬워지는 AWS 인프라 구축 가이드, 2019)와 비슷한 내용

# EC2
코드명에 g가 붙는건 AWS Graviton2 Processor, a는 AMD, i는 인텔. General Purpose는 T와 M, 숫자는 세대. 2022년 5월 현재 가장 빠른 CPU는 M6x, C6x. T4g가 좀 더 사양이 낮고 저렴하며 Graviton만 지원. C6i는 컴퓨팅 최적화 인텔 제온으로 c6i.large는 2 CPU/4G 서울 리전에서 시간당 $0.096, 월 $70.

- c6i.large: Intel(R) Xeon(R) Platinum 8375C CPU @ 2.90GHz 2x

## GPU instances
GPU 가능한 인스턴스는 다음과 같다. ap-northeast-2a에서도 p3 일부와 g4dn 가능
<img src="https://user-images.githubusercontent.com/1250095/167750886-b239139a-2722-42a8-96ec-cab11f2e5733.png" width="80%">[^fn-aws-gpus]

[^fn-aws-gpus]: <https://towardsdatascience.com/choosing-the-right-gpu-for-deep-learning-on-aws-d69c157d8c86>

p3 시리즈에서 nvlink 연결 여부는 `$ nvidia-smi topo --matrix`로 확인 가능하다. 2022년 5월 현재 p4d instance는 전세계 어디에도 생성되지 않고 있다.

## Azure V100
V100 4ea x 3yrs = 171,651,472 KRW  
V100 1ea 1 hour = 1,632 KRW

# 서비스
## RedShift
Amazon Redshift is an Internet hosting service and data warehouse product.
serverless가 아니라서 instance를 구동해야 하는데, creating이 너무 오래 걸린다. BigQuery는 serverless.

## RDS
Amazon RDS, in its ability to handle analytic workloads on big data data sets stored by a column-oriented DBMS principle. column-oriented DBMS principle은 Apache Arrow에도 Columnar In-Memory 방식으로 적용되어 있다.

Apache Arrow vs. Parquet: 둘 다 동일한 Columnar Data를 저장하며, in-memory 방식과 on-disk 방식이라는 차이점이 있다. BigQuery도 Columnar 방식

## S3
데이터를 쉽게 보관하고 access 할 수 있으나 분석 기능은 제대로 활용이 어렵다. 한글 JSON은 parsing하지 못했다.

## Athena
S3에 올린 파일이 128MB 이내일 경우 직접 Select from을 할 수 있으나 그 이상은 Athena에서 처리한다. 그러나 한글 JSON을 제대로 parsing하지 못했다.

# 설치 및 링크
AWS에서 Amazon Linux 2 설치 정리:
- [aws-essential-install.sh](https://gist.github.com/likejazz/2254db815675bdfd636e55460cc5b270)

## 도메인 설정 및 ELB 맵핑 과정
- 도메인: Route 53에서 설정
- 인증서 발급: 처음에는 Pending status이며 Domain에 CNAME 등록으로 인증 필요.
- Load Balancer: Application level로(아마 L7) https 트래픽을 forward 하도록 설정
    - http 트래픽은 https로 redirect
- Target Group: 내부 서버(www)로 http 연동. 즉 LB가 https로 받으면 내부에서는 http로 흐른다. 보안을 위해 80외에 임의 포트 지정하고 그쪽으로 연동.
- Route 53에서 A 레코드 등록 Alias to Application and Classic Load Balancer에서 ELB 선택.
- www 서버: Elastic IP 설정(필요 없으나 개발 편의를 위해 구성) 서버 한 대 구성. docker상에 flask를 올리기만 함. IP 대신 첫 번째 리소스를 택할 수 있고, 서버를 간단히 클릭하는 것으로 타겟에 구성 가능.
    - 추후 EKS 기반으로 개선하고 helm chart로 관리 필요

CloudFront와 헷갈리지 말 것. CDN 없이도 ELB만으로 충분히 셋팅 가능하다.
