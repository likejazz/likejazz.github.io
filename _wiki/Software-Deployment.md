---
layout: wiki 
title: Software Deployment
tags:  ["Infrastructure"]
last_modified_at: 2022/01/02 23:42:34
---

<!-- TOC -->

- [Ansible](#ansible)
    - [접속 테스트](#접속-테스트)
    - [배포의 문제점](#배포의-문제점)
- [Jenkins](#jenkins)
    - [Daemonize 및 Jenkins의 ProcessTreeKiller](#daemonize-및-jenkins의-processtreekiller)

<!-- /TOC -->

# Ansible

docker, K8s를 사용하면서 이제 기존 배포 시스템은 살펴볼 일이 없다. 예전에도 ansible은 좋아하지 않았는데, 그 이유는 추상화 수준이 너무 높아 실제로 어떤 동작이 수행되는지 알기 힘들었기 때문. 특히 agentless한 구조는 달리 말하면 ssh 만으로 모든 프로세스를 처리한다는 말인데 그렇게 해도 아무 문제가 없는지, 그냥 ssh로 확인하는 것과 뭐가 다른지 매번 의구심이 들었다. 컴맨드도 직관적이지 않고 ssh로 어떤 컴맨드가 수행되는지도 불안했다. 반면 docker는 어떠한 경우에도 동일한 환경에 항상 실행 가능한 상태를 보장하기 때문에 신뢰할 수 있다. 게다가 처음 dockerfile을 봤을때도 그 직관적인 컴맨드에 감탄했던 기억이 난다.

'추상화'는 초보자나 전문가 모두 큰 차이없이 업무를 진행할 수 있는 장점이 있는 반면, 전문 지식을 갖춘 전문가도 본인의 전문성을 발휘하기 어렵고, 추상화를 위한 별도의 기술을 전문가 조차 학습해야 하는 러닝커브가 존재한다. 뿐만 아니라 추상화 기술이 인지도가 높지 않아 사장된다면 초보자나 전문가 모두 그간의 경험을 재활용하기 어렵다. 따라서 '추상화'를 택할때는 그 기술을 개발하는 개발자나 사용하는 개발자 모두가 신중할 필요가 있다.

## 접속 테스트
```
$ ansible fusion-web -i inventory/test/hosts_etc_tomcat_only -m command -a 'ls -al'
```

인벤토리 파일내에 fusion-web 그룹 서버군에 `ls -al` 컴맨드를 날린다. 이를 통해 접속 여부 테스트가 가능하다.

좀 더 간단한 방법으로 아래와 같이 ping 할 수 있다.

```
$ ansible fusion-web -i inventory/test/hosts_etc_tomcat_only -m ping
```

## 배포의 문제점
ansible은 전체 배포를 하는데, 배포 중간에 http status를 확인하는 스크립트가 포함되어 있었다. 그러나, 모든 서버가 장애 상태에 빠져 http 확인이 안되어 이 스크립트가 실패로 기록되어 배포가 중간에 멈추는 경우가 발생했다. 강제로 웹 서버를 foreground로 띄워두고 http status 체크가 가능하도록 한 다음에야 겨우 배포가 가능했다.

설치 스크립트까지 포함된 ansible은 항상 위험 요인을 안고 있고, 배포할때 마다 늘 불안하다. 때문에 docker로 이미지를 말아서 최종 테스트를 해보고 문제 없을때 이미지를 배포하는 docker의 방식이 가장 편리했고, 무엇보다 안전하다.

# Jenkins
## Daemonize 및 Jenkins의 ProcessTreeKiller
바이너리를 데몬으로 띄우기 위해 처음에는 daemonize를 사용했으나 Go 바이너리는 잘 되지 않았다.

결국 Go 용도로 만들어낸 라이브러리를 찾게 되었고 go-daemon을 테스트 해보니 `go run`으로는 문제가 없는데 빌드 후 바이너리로 실행시 pid가 남지 않는 등의 문제가 있었다. 그러나 이외에는 잘 동작하여 그대로 사용하기로 했다.

그런데 jenkins에서 실행할때 데몬이 구동되지 않았다. nohup으로 해봐도 마찬가지. 알고보니 ProcessTreeKiller라는게 있어 해당 빌드에서 실행된 바이너리는 완료 후 모두 종료하도록 되어 있었다. [`BUILD_ID`를 지정](https://stackoverflow.com/a/37161006/3513266)하면 건드리지 않는다고. 이걸 몰라 한참을 고생했다.
