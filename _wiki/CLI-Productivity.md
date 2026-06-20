---
layout: wiki 
title: CLI Productivity
tags: ["Productivity"]
last_modified_at: 2026/06/20 00:12:32
---

<!-- TOC -->

- [모듈](#모듈)
- [diff와 patch](#diff와-patch)
- [bundle exec jekyll serve](#bundle-exec-jekyll-serve)
- [tar with excludes](#tar-with-excludes)
- [setup KST timezone](#setup-kst-timezone)
- [asciinema](#asciinema)
- [htop](#htop)
- [`set -eux`](#set--eux)
- [맥의 한글 파일이 리눅스에서 escaped sequences로 보이는 경우](#맥의-한글-파일이-리눅스에서-escaped-sequences로-보이는-경우)
- [To determine your Linux distribution](#to-determine-your-linux-distribution)
- [iTerm2 관련 설정 백업](#iterm2-관련-설정-백업)
- [mac에서 유용한 CLI 도구](#mac에서-유용한-cli-도구)

<!-- /TOC -->

# 모듈
모듈은 요청 시 커널에 로드할 수 있는 프로그램으로 커널을 재컴파일하거나 재부팅할 필요가 없다(p52, 모던 리눅스 교과서, 2023).
```shell
$ find /lib/modules/$(uname -r) -type f -name '*.ko*'
```
커널이 실제로 로드한 모듈 `$ lsmod`  
모듈 종속성 목록 조회 `$ modprobe --show-depends async_memcpy`

# diff와 patch
다음과 같이 변경 사항을 생성한다.

```console
$ diff -u a-original/a.py a-patched/a.py
--- a-original/a.py	2022-05-11 10:07:21.000000000 +0900
+++ a-patched/a.py	2022-05-11 10:07:42.000000000 +0900
@@ -1,4 +1,4 @@
 aaa
 bbb
-ccc
+ddd
 ddd
```

`> a.patch`를 이용해 패치 파일로 저장한 다음, 다음과 같이 적용할 수 있다.

```console
$ patch -p0 < a.patch
```

`a-original/a.py`에 패치가 적용된다.

# bundle exec jekyll serve
macOS를 업데이트 할 때 마다 실행에 문제가 있다.

macOS 기본 ruby에 system libraries를 설치할 수 없기 때문인데 Ventura에서 다음과 같이 해결했다.

```console
# rvm 설치
$ curl -sSL https://get.rvm.io | bash -s stable
$ rvm install "ruby-3.1.2"
$ brew install openssl@1.1

# https://stackoverflow.com/a/31516586
$ bundle config build.eventmachine --with-cppflags=-I$(brew --prefix openssl@1.1)/include

# likejazz.github.io
# https://stackoverflow.com/a/70916831
$ bundle add webrick

# runme.sh
$ bundle exec jekyll serve --host 0.0.0.0
```

# tar with excludes
디렉토리 제외 설정은 tar 옵션 맨 앞에, `cvf`는 반드시 하이픈 부여 필요.
```
$ tar --exclude='*.json' --exclude='FastChat/wandb' --exclude='FastChat/output' -cvf FastChat.tar FastChat/
```

# setup KST timezone
```
$ sudo apt install tzdata
(6 - 69)
```
It will automatically set `/etc/localtime`.

# asciinema
터미널 캡처 gif를 가장 깔끔하게 만드는 방법
- `$ brew instal asciinema` 설치
  - 굳이 업로드 할 필요는 없다. `$ asciinema rec`로 record 후 로컬 저장
- [asciinema-edit](https://github.com/cirocosta/asciinema-edit)로 quantize(딜레이 조정), cut(잘라내기)
- [gifcast](https://dstein64.github.io/gifcast/)를 이용해 animated gif로 변환. agg와 달리 한글 문제도 없으며 가장 깔끔하게 변환된다.

또 다른 옵션으로 [termtosvg](https://github.com/nbedos/termtosvg)도 있다. 바로 svg로 저장되므로 편리하지만 한글 출력시 약간씩 좌우로 흔들리는 버그가 있다.

# htop
top을 대체하는 최고의 프로젝트
- `Shift+H` Turn off userland threads
- `F2` Setup에서 설정 변경. Available meters에서 추가 및 이동 가능. `F10`으로 `~/.config/htop/htoprc`에 저장한다.

# `set -eux`
- `set -e`: 에러 발생시 실행 종료
- `set -u`: 초기화되지 않은 변수를 참조하면 에러를 발생시키고 실행 중단 
- `set -x`: 모든 실행 명령 출력

# 맥의 한글 파일이 리눅스에서 escaped sequences로 보이는 경우
```shell
$ export LC_ALL=ko_KR.UTF-8
$ ls -al
```

# To determine your Linux distribution
1. `cat /etc/os-release`
2. `lsb_release -a`
3. `uname -r`
4. `hostnamectl`

# iTerm2 관련 설정 백업
Jun 2026 맥 기본 Terminal 실험 중이며 iTerm2 기존 설정은 여기에 백업한다.

  - Preferences > Profiles > Colors에서 Blue/Normal의 색상이 눈이 아프므로 조정한다.
  - Settings > Profiles > Keys > Configure Hotkey Window `⌥⌘T` 등록
    - Pin hotkey window 체크

# mac에서 유용한 CLI 도구
```
$ brew install huggingface-cli
```
- `kubernetes-cli`: kubectl
- `firebase-cli`: HTML 호스팅 배포
- `uv`: 파이썬 버전 관리
- `pipx`: 파이썬 실행 패키지 별도 관리
- time을 대체하는 **gnomon**:
```
$ npm install -g gnomon
```

- 실행 스크립트는 `~/bin`에 두고 활용한다. bin은 GitHub에 백업
- 개발 관련 프로젝트는 `~/workspace` 디렉토리를 만들어 저장한다. 디렉토리 구조는 깃헙 URL 기준으로 만들어 관리하고 있다. 실험은 `~/workspace/sandbox`에서 진행한다.
- 터미널에서 Touch ID를 사용하기 위해 다음 설정
```
$ sudo cp /etc/pam.d/sudo_local.template /etc/pam.d/sudo_local
```