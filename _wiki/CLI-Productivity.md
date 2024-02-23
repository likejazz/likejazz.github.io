---
layout: wiki 
title: CLI Productivity
tags: ["Productivity"]
last_modified_at: 2024/02/23 14:09:41
---

<!-- TOC -->

- [모듈](#모듈)
- [diff와 patch](#diff와-patch)
- [Typora parser](#typora-parser)
- [bundle exec jekyll serve](#bundle-exec-jekyll-serve)
- [tar with excludes](#tar-with-excludes)
- [setup KST timezone](#setup-kst-timezone)
- [asciinema](#asciinema)
- [Mount disk](#mount-disk)
- [htop](#htop)

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

# Typora parser
<https://github.com/PegasisForever/typora-parser>

footnotes가 포함된 행에 superscript가 함께 있을 경우 제대로 처리되지 않는 문제가 있다. `src/inlines/inlineNode.ts`에서 아래 부분 패치:

```typescript
const nodePrecedenceGroups = [
  [RawHTMLNode, AutolinkNode, CodeSpanNode, EmojiNode],
  [HighlightNode, SubScriptNode, SuperScriptNode, FootnoteNode],
  [LinkNode.LinkNode],
  [EmphNode.EmphNode],
]
```

footnotes 순서를 superscript에 포함하여 패치. 빌드 과정:
```bash
$ npm i && \
chmod +x ./build/bin/typoraExport.js && \
./build/bin/typoraExport.js -g ./tags.txt -o ./ch5.html "5. test.md" && \
open ./ch5.html
```

# bundle exec jekyll serve
macOS를 업데이트 할 때 마다 실행에 문제가 있다.

macOS 기본 ruby에 system libraries를 설치할 수 없기 때문인데 Ventura에서 다음과 같이 해결했다.

```console
# rvm 설치
$ curl -sSL https://raw.githubusercontent.com/rvm/rvm/master/binscripts/rvm-installer | bash -s stable
$ rvm install "ruby-3.1.2"
$ brew install openssl@1.1

# https://stackoverflow.com/a/31516586
$ bundle config build.eventmachine --with-cppflags=-I$(brew --prefix openssl@1.1)/include

# https://stackoverflow.com/a/70916831
$ bundle add webrick

# runme.sh
$ bundle exec jekyll serve
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

# Mount disk
```
$ lsblk
# 파일시스템 포맷이므로 주의
$ sudo mkfs.xfs /dev/nvmexxx
$ sudo mkdir /models
$ sudo chown hyperai /models
$ sudo mount /dev/nvmexxx /models
```

# htop
top을 대체하는 최고의 프로젝트
- `Shift+H` Turn off userland threads
- `F2` Setup에서 설정 변경. Available meters에서 추가 및 이동 가능. `F10`으로 `~/.config/htop/htoprc`에 저장한다.