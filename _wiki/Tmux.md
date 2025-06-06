---
layout: wiki 
title: Tmux
tags: ["Productivity"]
last_modified_at: 2025/05/20 12:14:53
---

- [세션](#세션)
- [화면](#화면)
- [마우스 및 복사](#마우스-및-복사)

# 세션

`CTRL + B`를 누르면 기본 제어모드로 진입할 수 있다. esc 또는 설정되어 있지 않은 다른 키를 누르면 제어모드에서 바로 해제된다.

- `CTRL + B`, `C`: 새 윈도우 실행
- `CTRL + B`, `1`: 해당 세션 번호 선택으로 세션 이동 
- `CTRL + D`: 해당 세션 종료. screen과 동일하다.
- `CTRL + B`, `D`: 세션을 열어둔 채 tmux에서 빠져나옴. screen에서 `CTRL + A`, `D`와 동일하다.
- `CTRL + B + "`: Split current pane horizontally.

# 화면

- `CTRL + B`, `[`: 스크롤 모드 진입
- `CTRL + C`: 스크롤 모드 해제

스크롤 모드에서는 방향키 및 Page Up/Down으로 자유롭게 이전 화면 조회 가능. 출력 시간도 보이므로 편리하다.

# 마우스 및 복사

tmux가 마우스를 제어하기 때문에 화면 선택이 잘 안된다. 이 경우 다음과 같이 마우스 지원을 끈다.

- `CTRL + B`: 제어 모드 진입
- `:set -g mouse off`: 마우스 지원 해제

이렇게 하면 마우스 휠로 스크롤 등이 안되지만 대신 화면 선택이 잘 되므로 복사가 가능하다.