---
layout: wiki 
title: Jetson
tags: ["MLOps & HPC"]
last_modified_at: 2024/03/26 16:33:52
---

<!-- TOC -->

- [설치](#설치)

<!-- /TOC -->

# 설치
[가이드](https://developer.nvidia.com/embedded/learn/get-started-jetson-agx-orin-devkit) 초기 설정에 10분 이상 소요. headless configuration으로 monitor, input device없이 진행이 가능하다. USB-C 케이블을 연결해 USB 모뎀으로 진행:
```shell
$ screen /dev/cu.usbmodem1420xxxx 115200
```
가이드에는 `sudo`로 안내하고 있으나 굳이 sudo를 할 필요는 없었다.

screen 조정을 위해 `.profile`에서 수정
```shell
# .profile
...
# Fix for screen session
export TERM=xterm-256color
resize
...
```

`sudo apt install nvidia-jetpack`로 Jetpack Components를 설치한다. 9.1G이며 1시간 이상 소요.