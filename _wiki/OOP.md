---
layout: wiki 
title: OOP
tags:  ["Languages & Framework"]
last_modified_at: 2023/04/12 15:42:39
---

<!-- TOC -->

- [특징](#특징)
  - [다형성Polymorphism](#다형성polymorphism)
- [디자인 패턴](#디자인-패턴)
  - [Factory Pattern](#factory-pattern)
  - [State Pattern](#state-pattern)

<!-- /TOC -->

# 특징
캡슐화<sup>Encapsulation</sup>, 추상화<sup>Abstraction</sup>, 상속<sup>Inheritance</sup> 및

## 다형성<sup>Polymorphism</sup>
다형성은 경우에 따라 객체가 다르게 동작하거나 또는 어떤 동작을 다른 방법으로 동작하게 하는 개념이다.

* **Template** 다루는 형이 다르지만 함수 내부의 처리 방법이 같다.
    * Java에는 Generic이 있고, C++과 달리 하나의 컴파일된 코드를 만든다. 하지만 컴파일 타임에 오류를 체크할 수 있는 용도는 동일하다.
* **Overriding** 이름과 파라미터가 동일한 함수를 상속 받아 재정의한다.

**Overloading** 이름은 같지만 파라미터가 다른 함수.
- C는 당연히 오버로딩이 없다.
- Go도 없다. 이름만으로 처리하기 때문이다. 하지만 편법으로 가능하다.

# 디자인 패턴
## Factory Pattern
가장 널리 사용하는 생성 패턴 중의 하나다. 팩토리 패턴의 유용성은 로직과 분리된 공통 인터페이스를 사용하는 방식에 있다. (안드로이드 디자인 패턴과 활용 사례, 2016) 실제로 if else가 복잡하게 중첩된 구문을 Factory Pattern으로 깔끔하게 정리할 수 있었다. 이외 Abstract Factory Pattern, Builder Pattern으로 더 깔끔한 형태로 구현 가능하나 [Factory Pattern의 기본적인 형태](https://github.com/likejazz/Android-Design-Patterns-and-Best-Practice/tree/master/Chapter01/FactoryPattern)로도 충분했다.

## State Pattern
State Pattern을 이용해 유한 상태 머신 <sup>finite state machine</sup>을 구현할 수 있다. coin-operated turnstile 개찰구를 예([위키피디어](https://en.wikipedia.org/wiki/Finite-state_machine))로 들 수 있는데, State Pattern으로 구현 가능하다. [안드로이드 구현](https://github.com/likejazz/Android-Design-Patterns-and-Best-Practice/tree/master/Chapter10/Turnstile) 수학적으로 모델링할 수 있는 모든 것을 모델링하는데 사용할 수 있다.

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/9e/Turnstile_state_machine_colored.svg/660px-Turnstile_state_machine_colored.svg.png" width="50%">

State in UML
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e8/State_Design_Pattern_UML_Class_Diagram.svg/800px-State_Design_Pattern_UML_Class_Diagram.svg.png" width="50%"> ([위키피디어](https://en.wikipedia.org/wiki/State_pattern))
