---
layout: wiki 
title: AWS
last-modified: 2020/12/01 08:51:20
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
- [Books](#books)
    - [그림으로 배우는 클라우드 인프라와 API의 구조 <sub>2016, 2017</sub>](#그림으로-배우는-클라우드-인프라와-api의-구조-2016-2017)

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

# Books
## 그림으로 배우는 클라우드 인프라와 API의 구조 <sub>2016, 2017</sub>
- 클라우드를 제어하는 API의 동작 방식  
FQDN Fully Qualified Domain Name: 도메인과 호스트명이 하나로 연결된 전체 이름.  
ROA Resource Oriented Architecture, 리소스 지향 아키텍처란 REST API의 사상을 기반으로 리소스 중심적인 API를 사용하는 아키텍처를 말한다. REST는 프로토콜이 아니라 일종의 사상이나 접근 방식에 가깝다. DeveloperWorks에 올라온 RESTful Web services에서는 아래 4가지 설계 지침을 언급한다.
1. 상태를 갖지 않도록 만든다.
1. URI는 디렉터리 구조처럼 계층적으로 만든다.
1. HTTP 메모들을 명시적으로 사용한다.
1. 응답할 때는 XML이나 JSON을 사용한다(둘 다 사용해도 무방)

멱등성 <sup>idempotent</sup>: 입력값이 동일하면 항상 같은 결과를 보장한다. functional programming에서 pure function과 유사
- 오케스트레이션  
대표적인 도구로 puppet, chef, **ansible**. 소프트웨어 개발의 자동화를 위해 테스트를 만드는 작업에 해당. 
- 멀티 클라우드  
특이하게 멀티 클라우드를 적용하는 방법에 대해 소개. 컨테이너가 주목받는 이유는 OS와의 의존성을 끊어줌으로써 클라우드 간의 이행을 더 쉽게 만들어주기 때문 p.414
- 이뮤터블 인프라스트럭처  
기존 시스템은 하드웨어, 소프트웨어의 유지보수와 업그레이드, 관리와 운영이 지속적으로 필요하다.  
<img src="https://user-images.githubusercontent.com/1250095/62117431-e24e6c00-b2f6-11e9-907c-808809ff4d5f.jpg" width="45%" style="float: left; margin-right: 5px"><img src="https://user-images.githubusercontent.com/1250095/62117432-e24e6c00-b2f6-11e9-996f-60f99ee0847a.jpg" width="45%">
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==" style="clear: both">
그러나, 이뮤터블 인프라스트럭처는 인프라 환경을 자동으로 구축하고, 시스템을 변경해야 할 때는 이미 구축된 환경을 수정하는 대신, 구축된 환경을 파괴하고 수정된 환경으로 다시 구축한다.  

AWS의 CloudFormation 확인 필요