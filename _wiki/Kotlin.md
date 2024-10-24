---
layout: wiki 
title: Kotlin
tags:  ["Languages & Framework"]
last_modified_at: 2023/10/04 01:01:27
---

<!-- TOC -->

- [실행](#실행)
- [정보](#정보)
- [정리 필요](#정리-필요)
- [Kotlin Native](#kotlin-native)
- [책](#책)
  - [코틀린을 다루는 기술 2019, 2020](#코틀린을-다루는-기술-2019-2020)
  - [빅 너드 랜치의 코틀린 프로그래밍 2018, 2019](#빅-너드-랜치의-코틀린-프로그래밍-2018-2019)
  - [Kotlin in Action 2016, 2017](#kotlin-in-action-2016-2017)

<!-- /TOC -->

# 실행
별도 정리 필요

```
기본 

$ kotlinc hello.kt -include-runtime -d hello.jar // 독립 배포 가능
$ java -jar hello.jar

또는 

$ kotlinc hello.kt // HelloKt.class 생성
$ java HelloKt

REPL

$ kotlinc-jvm or kotlin
```

# 정보
빌드
```
$ gradle jar
$ java -jar build/libs/example-0.1-SNAPSHOT.jar
```

디버깅

```
$ javap -c HelloKt
Compiled from "hello.kt"
public final class HelloKt {
  public static final void main();
    Code:
       0: ldc           #8                  // String Hello, World!
       2: astore_0
       3: iconst_0
       4: istore_1
       5: getstatic     #14                 // Field java/lang/System.out:Ljava/io/PrintStream;
       8: aload_0
       9: invokevirtual #20                 // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
      12: return

  public static void main(java.lang.String[]);
    Code:
       0: invokestatic  #23                 // Method main:()V
       3: return
}
```

Kotlin Worksheet는 Scratch와 달리 프로젝트에 종속적이다. Interactive Mode가 있어 초기 개발이 매우 편리하다. 사실상 REPL을 대체할 수 있다. 예전에 Swift가 XCode에서 이렇게 동작했다.

# 정리 필요
- abstract vs. interface
- mutableMap map.getOrPut() 자바에서는 이런걸 보지 못했는데 메소드명이 재밌다.
- DSL custom language 설계할때 필요

# Kotlin Native
LLVM backend로 지원한다. 이미 kotlin이 설치된 상황에서 설치되지 않으며, cask로 설치한다. `$ brew install --cask kotlin-native` 기존 kotlin은 uninstall 했는데, IntelliJ는 어플리케이션 dir내에 kotlin을 탑재하고 있기 때문에 문제가 없다.

```
$ kotlinc-native a.kt
```

맥에서는 권한 문제가 발생하므로 추가 처리가 필요하다.
```
$ xattr -rd com.apple.quarantine /usr/local/Caskroom/kotlin-native
```

이외에도 최초 실행시 필요한 모듈을 `~/.konan` 아래로 다운 받는다. konan은 kotlin native의 애칭이다. 컴파일은 당연히 `package`가 지정되어 있으면 안되고 기본 파일명은 `program.kexe`이다. 컴파일 시간이 비교적 오래 걸리는데 간단한 큐 구현 파일이 맥북에서 10초 걸렸다.

# 책
## 코틀린을 다루는 기술 <sup>2019, 2020</sup>
★★★☆☆  
팩트 책과는 달리 깊이 있는 내용이 많고 특히 함수형을 자세히 소개한다. joy of kotlin이라는 원제 답게 코틀린으로 다양한 재미거리를 많이 시도하며 연습 문제로 구성되어 있어 자율 학습하기에 좋다. 그러나 언어의 흥미있는 활용에 너무 집중해 실무에 도움이 될 만한 내용은 많지 않다.

## 빅 너드 랜치의 코틀린 프로그래밍 <sup>2018, 2019</sup>
★★★☆☆  

## Kotlin in Action <sup>2016, 2017</sup>
★★★★★  
저자인 Dmitry Jemerov는 코틀린 컴파일러 개발에 관여했으며 현재 제트브레인 CTO다.