---
layout: wiki 
title: JetBrains
tags: ["Productivity"]
last_modified_at: 2025/06/08 19:25:15
---

<!-- TOC -->

- [장점](#장점)
- [단점](#단점)
- [설정](#설정)
- [JetBrains Gateway](#jetbrains-gateway)
- [PyCharm Scientific mode](#pycharm-scientific-mode)

<!-- /TOC -->

<img width="40%" src="https://user-images.githubusercontent.com/1250095/91637250-366bd880-ea42-11ea-9b34-9ef860abc3be.png" style="float: left; margin-right: 5px"> <img width="40%" src="https://user-images.githubusercontent.com/1250095/135706868-cc2505df-b395-4258-980f-7b1669893151.png">[^fn-date]

[^fn-date]: Oct 2021

JetBrains의 IDE를 사용해오면서 어떤 장단점이 있는지 정리해본다. 2014년 [IntelliJ IDEA 정품 구매](https://likejazz.tumblr.com/post/103532169020/intellij-idea-%EC%A0%95%ED%92%88-%EA%B5%AC%EB%A7%A4), 2015년 [CLions 1.0 릴리즈](https://likejazz.tumblr.com/post/118649049333/clion-1-0) 이후 정품 구매로 시작해 지금까지 계속 매년 정기 구독 중이다. [정기 구독 모델은 2015년 11월에 시행](https://likejazz.tumblr.com/post/133725850005/jetbrains-all-products-pack)됐다.

# 장점
1. 강력한 자동 완성  
메소드가 뭐가 있는지 헷갈릴때 점(.)을 찍고 기다린다. AI Assistant에서는 코드 완성 지원.
1. 실시간 에러 체크
1. 강력한 모듈 브라우징  
라이브러리의 in/out이 기억나지 않을때 모듈 코드를 따라가서(`⌘Click`) 어떻게 처리되는지 확인할 수 있다. ctags가 필요 없다.
1. 즉시 실행  
C++을 비롯해 바로 실행이 쉽지 않은 언어에서도 즉시 실행 가능한 화살표를 지원한다. 실행 할 경우 Run에 저장되며 그 다음 부터는 `⌃R`로 바로 가능하다. Docker도 지원한다.
1. 다양한 편집 기능  
    - `⌘⇧↓`: 줄단위 이동
    - `⌃⌥↓`: 특정 변수 또는 함수의 사용 현황을 쉽게 브라우징 가능 (Next Occurrence)
    - `⌥↑`: 블럭 지정 또한 강력함
    - `⌘F12`: File Structure, 전체를 조망하는 기능
    - `⌘⇧-`: 코드 전체를 Collapse
    - `⌘+`: 코드 일부를 Expand
    - `Column Selection Mode` 선택 후 마우스로 Shift + Click 하면 거기까지 멀티 커서로 된다.
    - 멀티라인 코드 완성 `⌥⇧\`
    - 코드 생성 `⌘\`
1. Toolbox  
언어에 따른 도구 선택. 프로젝트 단위로 선택 또한 가능. 자동 업데이트.
1. 빠른 대응  
새로운 기술이 등장하면 항상 발 빠르게 대응해 플러그인을 제공한다. Rust, Deno등이 대표적.
1. 디테일  
코드 편집 기능에서 recursive call까지 표현해주는 디테일이 돋보인다.

# 단점
1. 유료  
기업용은 연간 60만원. 스타트업은 5년 간 10 카피 50% 할인
1. 무거움
1. 환경 설정  
이전에 비해 많이 편해지긴 했으나 여전히 약간의 설정은 필요함. 설정이 제대로 되어 있지 않을 경우 온통 빨간색 오류가 발생한다.
    - ~~같은 이유로 IDE 이기 때문에 프로젝트 단위 설정만 가능하고, 파일 단위로 편집은 어렵다.~~ LightEdit mode가 생겼다.

# 설정
- IntelliJ의 테마는 Darcula를 사용한다. `Editor > General > Appearance`에서 **Show whitespaces**와 **Show method separators**는 활성화 한다.
- IntelliJ의 `Editor > Color Scheme > General`에서 **Identifier under caret** 기능은 가장 자주 확인하는 기능이라 기본 Darcula에서 이 색상만 눈에 띄게 지정하고 사용한다.
    - Background를 `FFFF00`로 지정
    - `Identifier under caret (write)`는 Background를 `FFFFE0`로 지정

# JetBrains Gateway
리눅스에 IDE 리눅스 버전을 구동하고 껍데기만 로컬에 씌워서 보여주는 형태
- `t4g.micro`에서는 ssh 연결 이후 진행이 되지 않음. x86 버전 설치를 시도하기 때문으로 추정.
- `t3.micro`는 너무 오래 걸리고 설치 실패. 사양 부족.
- `t3.medium` 이 사양으로 구동했고 성공

# PyCharm Scientific mode
DataSpell이 Jupyter Notebook을 지원하지만 여전히 많은 부분이 불편하다. PyCharm의 Scientific mode는,
1. 입력과 출력이 구분되어 편리하다.
2. Inspector가 있어 세션에서 관리 중인 모든 Variables를 확인할 수 있다.
3. 간단한 테스트 시 Jupyter는 직접 창을 만들어 실행해야 하나 Scientific mode는 출력창에서 바로 확인 가능하다.
4. Matplotlib나 Pandas를 별도로 도식화 해 보여준다. Matplotlib는 Jupyter의 경우 바로 밑에 보여주니 오히려 더 편할 수 있겠다.
