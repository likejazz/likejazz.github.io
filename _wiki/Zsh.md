---
layout: wiki 
title: Zsh
tags: ["Productivity"]
last_modified_at: 2021/10/02 15:59:01
---

<!-- TOC -->

- [Zsh](#zsh)
    - [zshrc](#zshrc)

<!-- /TOC -->

# Zsh
fish를 쓰다 oh-my-zsh에 정착. 여러가지 themes도 고민한 결과 [spaceship-prompt](https://github.com/denysdovhan/spaceship-prompt)를 선택. 디렉토리 표시 등 shell 로서의 표현 보다 gcp 설정, conda 환경 표시 등 환경 설정을 표현하는데 더 강점이 있는 theme이다. 따라서 기존 conda에서 환경 설정을 표현하는 부분은 disabled 처리.

```console
$ conda config --set changeps1 False
```

<img width="70%" src="https://user-images.githubusercontent.com/1250095/93846989-a0456e00-fce0-11ea-952c-783b7f3044ca.png">

## zshrc
원래 macOS 설치 프로그램 정리 문서에 기입했던 내용을 여기에 정리한다.
- fasd 는 `brew install fasd`로 별도 설치. 아래 .zshrc의 plugins 설정에 fasd를 추가하면 편리한 aliases를 사용할 수 있다.
    - 사실상 `z`만 쓴다. interactive는 `zz`.
- ag와 함께 준건님이 만든 fzf도 꼭 필요한 텍스트 검색 도구다.  
```
$ brew install the_silver_searcher
$ brew install fzf
```  
fzf는 `⌃R`(히스토리 검색)을 포함한 키 바인딩과 추가 맵핑을 함께 설치했다. 디렉토리에서 `fzf`를 실행하면 하위 파일명에 대한 검색이 된다. 사실상 모든 검색 인터페이스는 fzf로 통일.
- 플러그인은 `plugins=(git history python fasd history-substring-search docker)`를 사용한다.
    - 플러그인의 역할은 `alias`를 비롯한 여러가지 함수/기능 등록
