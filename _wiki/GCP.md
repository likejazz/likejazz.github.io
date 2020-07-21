---
layout: wiki 
title: GCP
last-modified: 2020/07/21 20:41:20
---

<!-- TOC -->

- [SSH 접속](#ssh-접속)

<!-- /TOC -->

# SSH 접속
gcloud를 이용하는 방법은 너무 불편하여 ssh로 바로 접속이 가능하도록 셋팅하다.

compute engine에서 ssh keys에 gcp-user,
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSwZ37DGZfsOmT1zj40zT9uPnbNi5prL8fVEebvwWW9hYsbZ7ctcS9MXuUW3rUoDxlCyxIYBwqDqqbKTiUua94B1OpSzvsWwSOQ/... gcp-user
```
`cat ~/.ssh/id_rsa_gcp.pub`를 등록했다. 그러나 다른 서버에 등록한 설정도 동일 프로젝트에서 ssh 접속이 되는 현상이 있음. 동일한 centos에서 발생하는 것으로 보아 버그 같다. network interface details에서 내 IP에 대해 all allow 처리로 편리하게 사용한다.