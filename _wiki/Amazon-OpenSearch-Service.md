---
layout: wiki 
title: Amazon OpenSearch Service
tags: ["Information Retrieval"]
last_modified_at: 2022/01/12 11:57:36
---

<!-- TOC -->

- [설치](#설치)
- [설정](#설정)
- [특징](#특징)

<!-- /TOC -->

# 설치
몇 가지 설정만으로 어렵지 않게 구동할 수 있다. 노드 수와 인스턴스를 직접 지정할 수 있는데, m6g.large AWS Graviton2를 처음으로 선택했다. 다른 차세대 CPU에 비해 저렴하고 default도 메모리 최적화인 r6g.large로 되어 있어 이쪽으로 유도하는듯 하다. Fine-grained access control은 원하는 IAM ARN 지정 가능한데 다른 추가 설정없이 그냥 IAM으로는 dashboards에 로그인 할 수 없었다. node가 1대인데도 생성에 거의 20여분은 걸린거 같다.

# 설정
보안 때문인듯 초반에 많은 것들이 차단된 상태로 구동되어 처음 접속하기 위해서는 할 일이 많다. 자칫 잘못하면 여기서 많은 시간을 허비한다.
- VPC(recommended)를 권장하고 이 경우 private IP가 부여되어 별도 ec2 instance로 [proxy 설정해야](https://docs.aws.amazon.com/ko_kr/opensearch-service/latest/developerguide/vpc.html) 접속이 가능하다.
- 기본 access policy가 deny로 되어 있어 `User: anonymous is not authorized to perform: es:ESHttpGet` 에러가 발생한다. 직접 json에서 allow로 편집했다. (이건 애초에 domain access policy를 
Do not set domain level access policy로 설정해서 그런듯 하다)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "*"
      },
      "Action": "es:*",
      "Resource": "arn:aws:es:ap-northeast-2:XXXXXXXX:domain/airs-search/*"
    }
  ]
}
```
- ARN 지정만으로는 `/_dashboard` URI로 접근가능한 Kibana에 접속되지 않았는데 SAML이나 Amazon Cognito를 사용해야 한다. master 계정을 직접 생성해 별도로 로그인했다.
- 처음부터 ARN 지정없이 master 계정만 생성할 경우 콘솔에서 cluster health나 indices가 권한 없다며 조회되지 않는데, ARN을 지정하고(이대로 두면 키바나 로그인이 안됨) 다시 master 계정 생성으로 돌아오는 편법을 사용하면 콘솔 조회도 가능하다.

# 특징
- Kibana에 비해 dashboard가 꽤 단촐하다. 어차피 Kibana에서도 사용하는 메뉴가 제한적이긴 했으나 여기선 Dev Tools 조차 매우 단촐하다. profiler, painless등이 모두 빠져 있다.
- cluster health는 aws console로 빠져 있고 여기서 시간 단위로 추이를 모니터링 할 수 있다. 색인 문서 수나 노드의 메모리도 모두 표시된다. 노드 추가도 cluster configuration을 바꾸는 것으로 간단히 가능하다. 특별히 시스템 관리가 필요 없다. 그러나 추가 작업은 생성과 마찬가지로 시간이 오래 걸린다.
- 플러그인은 packages라는 이름으로 마찬가지로 aws console에서 s3를 통해 업로드하도록 되어 있다.
- fully managed라서 설정할게 거의 없고 이마저도 auto-tune(아마도 jvm tuning을 포함해) 기능을 지원한다. red/yellow/green status와 그래프만 보면서 정상 동작 여부만 확인하면 된다.