---
layout: wiki 
title: Terraform
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [인증](#인증)
- [import](#import)

<!-- /TOC -->

# 인증
서비스 계정을 만들고 JSON 인증으로 공통으로 사용할 수 있지만 혼자서 쓸때는 `credentials`를 설정하지 않으면 `gcloud` 개인 인증으로 사용한다.

# import
실제 구성을 가져와서 tf.state에 저장한다. tf 파일은 `terraform show`등을 활용해 직접 구성해야 한다.