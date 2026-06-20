---
layout: wiki 
title: Zsh
tags: ["Productivity"]
last_modified_at: 2026/06/20 10:53:02
last_modified_history:
  - 2026/06/12 oh-my-zsh uninstall
  - 2021/10/02 초안 작성
---

<!-- TOC -->

- [Zsh](#zsh)
  - [zshrc](#zshrc)

<!-- /TOC -->

# Zsh
fish, oh my zsh 모두 쓰지 않는다. zoxide, ag, fzf로 끝. 프롬프트는 다음과 같이 날짜만 출력한다.

```bash
# Prepend the current time to the shell prompt in HH:MM:SS format.
PS1='🪟 \t \u@\h:\w\$ '
```

## zshrc
원래 macOS 설치 프로그램 정리 문서에 기입했던 내용을 여기에 정리한다.
- [zoxide](https://github.com/ajeetdsouza/zoxide)설치. .zshrc에서 `eval "$(zoxide init zsh)"`로 설정. `z`와 `zi`를 즐겨 쓴다.
- ag와 함께 준건님이 만든 fzf도 꼭 필요한 텍스트 검색 도구다.  
```
$ brew install the_silver_searcher
$ brew install fzf
$ brew install helix
```  
fzf는 `⌃R`(히스토리 검색)을 포함한 키 바인딩과 추가 맵핑을 .zshrc에서 `source <(fzf --zsh)`로 설치할 수 있다. 파일 검색은 `^T`, 모든 검색 인터페이스는 fzf로 통일.