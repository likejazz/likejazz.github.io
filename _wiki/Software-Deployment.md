---
layout: wiki 
title: Software Deployment
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [Ansible](#ansible)
    - [접속 테스트](#접속-테스트)
    - [배포의 문제점](#배포의-문제점)
- [Jenkins](#jenkins)
    - [Daemonize 및 Jenkins의 ProcessTreeKiller](#daemonize-및-jenkins의-processtreekiller)
- [What's the version of my OS?](#whats-the-version-of-my-os)

<!-- /TOC -->

# Ansible
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

# What's the version of my OS?
**Linux**[^fn-linux]

[^fn-linux]: <https://whatsmyos.com>

Open a terminal and type `uname -a`. This will give you your kernel version, but might not mention the distribution your running. To find out what distribution of linux your running (Ex. Ubuntu) try `lsb_release -a` or `cat /etc/*release` or `cat /etc/issue*` or `cat /proc/version`.
