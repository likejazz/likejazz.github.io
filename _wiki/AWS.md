---
layout: wiki 
title: AWS
last-modified: 2020/12/04 20:54:50
---

<!-- TOC -->

- [계정 생성](#계정-생성)
- [개요](#개요)
- [Azure V100](#azure-v100)
- [서비스](#서비스)
    - [RedShift](#redshift)
    - [RDS](#rds)
    - [S3](#s3)
    - [Athena](#athena)
- [스크립트](#스크립트)
- [기타](#기타)

<!-- /TOC -->

# 계정 생성
[IAM에서 생성](https://console.aws.amazon.com/iam/home?region=ap-northeast-2#/users)한다.  

# 개요
IAM, S3, VPC - ELB, EC2, RDS, Bridge, Auto Scaling, CloudFront, Route 53, CloudWatch  
(예제를 통해 쉽게 따라하는 아마존 웹 서비스, 2017)

- IAM에서 User는 Group으로 관리, Role은 OTP기반으로 동작하는 권한이다. 이미 설정된 Managed Policy를 권장하며 AWS에서 미리 정의했다.
- Subnet은 외부 접근이 허용된 Public과 차단된 Private이 있다.
- Elastic IP로 고정 IP를 받을 수 있다. Bastion Host는 gateway의 역할을 한다. 
- MySQL compatible한 Amazon Aurora DB를 권장한다. 
- ELB는 DNS만 제공. IP가 자주 변경된다. https도 지원한다.

(서비스 운영이 쉬워지는 AWS 인프라 구축 가이드, 2019)와 비슷한 내용

# Azure V100
V100 4ea x 3yrs = 171,651,472 KRW  
1 hour = 1,632 KRW

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

# 스크립트
AWS에서 Amazon Linux 2를 발급 받으면 최소 설치해야 하는 목록
- [aws-essential-install.sh](https://gist.github.com/likejazz/2254db815675bdfd636e55460cc5b270)

# 기타
AWS의 CloudFormation 확인 필요