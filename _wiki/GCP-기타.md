---
layout: wiki 
title: GCP 기타
tags: ["Cloud"]
last_modified_at: 2022/02/10 03:55:23
---

<!-- TOC -->

- [Cloud Domains](#cloud-domains)
- [Memorystore](#memorystore)

<!-- /TOC -->

# Cloud Domains
Google Cloud Domains(Beta)는 프로젝트 기반이고 도메인을 다른 프로젝트로 이동 불가[^fn-gdomains] export 하게 되면 더 이상 관리 안되며 [Google Domains](https://domains.google.com/registrar)에서만 확인 가능. 여기는 계정 기반으로 프로젝트도 없는 별도 서비스다. 우리나라에서는 지원 안된다고 나오지만 My domains에서 확인 가능. 여기서 exported 도메인을 찾아 완전히 삭제할 수 있다.

[^fn-gdomains]: <https://stackoverflow.com/a/65962034/3513266>

# Memorystore
Redis를 Managed로 지원한다. AUTH 및 TLS도 지원한다. Private IP만 제공하지만 로컬에서 포트 포워딩[^fn-port]으로 접속 가능하다.

[^fn-port]: <https://cloud.google.com/memorystore/docs/redis/connecting-redis-instance#connecting_from_a_local_machine_with_port_forwarding>

```
$ gcloud compute ssh server-proxy \
	--project=XXX \
	--zone=asia-northeast3-a \
	-- -N -L 6379:XXX.XXX.XXX.XXX:6379
```
이렇게 하면 로컬에서 접속 가능하다. 당연히 기본적으로 외부망에 열어버리면 심각한 보안 문제가 있으니 이런 식으로 우회 접근 괜찮아보인다. 공식 가이드에서도 권장하는 방식.

<img src="https://user-images.githubusercontent.com/1250095/136141761-f62ddf3e-2420-47c7-b614-22b31b1335dc.png" width="70%">

MySQL(Cloud SQL)도 동일한 방식으로 가능하다.
