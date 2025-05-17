---
layout: wiki 
title: Cloud Load Balancing
tags: ["Cloud"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [설정](#설정)
    - [Backend](#backend)
    - [Frontend](#frontend)

<!-- /TOC -->

App Engine이나 Cloud Run 모두 Allow internal traffic and traffic from Cloud Load Balancing이 가능하다. 즉, LB를 통하지 않고는 외부에서 직접 접근을 차단할 수 있다.

App Engine은 INGRESS 설정으로 가능하다.
```
$ gcloud app services update --ingress=internal-and-cloud-load-balancing
```

# 설정
서비스 용도로 L7 HTTP(S) LB를 이용한다. 

## Backend
Serverless는 외부망에서 접속할때만 백엔드로 지정 가능하다. Serverless NEG로 App Engine 또는 Cloud Run 설정이 가능하다. 이외에 CDN, Cloud Armor 설정이 가능하다.

## Frontend
HTTP 또는 HTTPS를 지정할 수 있으며, 외부망에서 사용할때 HTTP는 의미가 없다. Cloud Run을 백엔드로 할때는 HTTP를 사용할 수 있어 속도 증가의 효과는 있다. 그러나 외부에 그대로 노출되는 것이므로 보안 문제가 있다. 설정 이후 적용시 까지 시간이 조금 소요된다. HTTPS인 경우 인증서를 직접 발급받아 업로드가 필요하다.
