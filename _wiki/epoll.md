---
layout: wiki 
title: epoll
tags:  ["Network Programming"]
last_modified_at: 2025/06/08 19:29:30
---

<!-- TOC -->

- [개요](#개요)
- [epoll](#epoll)
  - [빌드 방법](#빌드-방법)
    - [빌드 환경](#빌드-환경)
    - [submodule 추가 방법](#submodule-추가-방법)

<!-- /TOC -->

# 개요
Node.js는 원래 libev를 도입했으나 Windows를 지원하지 않아 IOCP도 함께 지원하는 libuv를 개발하고 전환했다.

# epoll
<img src="https://suchprogramming.com/wp-content/uploads/2018/01/poll-times.png" width="70%">[^fn-speed]

Linux는 모든게 file descriptor로 구성되는데, 갯수에 상관없이 일정하게 빠른 속도를 보인다는 큰 장점이 있다.

[^fn-speed]: <https://suchprogramming.com/epoll-in-3-easy-steps/>

epoll을 이용한 simple echo 서버[^fn-tutorial], submodule 구조에 cmake가 적용되어 있다.

[^fn-tutorial]: <https://github.com/isaacmorneau/simple-epoll>

## 빌드 방법
당연히 Linux에서만 빌드 가능
```console
$ git clone https://github.com/isaacmorneau/simple-epoll
$ cd simple-epoll
$ git submodule update --init --recursive

# 빌드 도구 설치
$ sudo apt update # Ubuntu 20.04에서는 생략 가능
$ sudo apt install -y build-essential cmake

$ mkdir bld && cd bld && cmake ..
$ make
```

### 빌드 환경
```console
$ hostnamectl
...
Operating System: Ubuntu 20.04.2 LTS
          Kernel: Linux 5.4.0-1043-gcp
```

Kernel 5.x에서 오류 발생. wrapper.c 수정이 필요하다. Kernel 4.15인 Unbuntu 16.04에서 빌드 시도. 빌드 오류 발생 `EPOLLEXCLUSIVE`가 Undeclared라 삭제하고 다시 빌드 성공.

```console
$ hostnamectl
...
Operating System: Ubuntu 16.04.7 LTS
          Kernel: Linux 4.15.0-1098-gcp
```

그러나 마찬가지 오류 발생
```
/home/gcp-user/simple-epoll/src/wrappers/wrapper.c::add_epoll_fd::348
	(ret = epoll_ctl(efd, EPOLL_CTL_ADD, ifd, &event)) != -1: Bad file descriptor
```

**현재 해결 못함**

### submodule 추가 방법
```console
$ git submodule add <remote_url> <destination_folder>
```
