---
layout: wiki 
title: Java
tags:  ["Languages & Framework"]
last_modified_at: 2022/02/07 20:25:22
---

<!-- TOC -->

- [객체의 생성과 삭제](#객체의-생성과-삭제)
- [모든 Objects의 공통 메서드](#모든-objects의-공통-메서드)
- [제네릭](#제네릭)
- [enums와 annotations](#enums와-annotations)
- [methods](#methods)
- [일반적인 프로그래밍 원칙](#일반적인-프로그래밍-원칙)
- [동시성<sup>concurrency</sup>](#동시성supconcurrencysup)
- [클린 코드](#클린-코드)
- [Spring Initializr](#spring-initializr)
- [Books](#books)
    - [객체지향의 사실과 오해 <sup>2015</sup>](#객체지향의-사실과-오해-sup2015sup)
    - [Practical 모던 자바 <sup>2020</sup>](#practical-모던-자바-sup2020sup)

<!-- /TOC -->

# 객체의 생성과 삭제
- 생성자 인자가 많을때는 Builder 패턴을 고려한다. 
    - C++은 default parameters가 있기 때문에 굳이 필요하지 않으며, Lombok에 `@Builder`가 있다. gradle-lombok 플러그인은 lombok 최신 버전이 없어서 1.16.4 버전을 사용했다.
- JDK 1.5 부터 autoboxing이 생겼는데, primitive type을 boxed primitives와 섞어 사용할 수 있도록 해주며, 자동 변환된다. 의미<sup>semantic</sup> 차이는 미미하지만 성능 차이는 무시하기 어렵다. 따라서 `long` 대신에 `Long`을 사용하지 않도록 유의한다. 
```
Long sum = 0L;
long i = 100;
sum += i;
```
자바 제네릭 자료형 시스템의 한계 때문에 제네릭에는 boxed primitive type을 사용한다.
- 자바도 memory leak이 발생한다. 예를 들어 Stack 구현에서 `pop()` 이후에는 `elements[size] = null` 처리가 필요하다.

# 모든 Objects의 공통 메서드
- Object는 override 하도록 설계된 메서드 `equals`, `hashCode`, `toString`, `clone`, `finalize`가 있다. 
    - hash table 기반 컬렉션과 함께 사용하려면 `hashCode`를 재정의해야 한다. 만약 항상 같은 값으로 구현한다면 키 충돌이 일어나고 linked list를 사용하는 것과 다르지 않을 것이다. Lombok의 `@EqualsAndHashCode`.
    - `toString`이 유용하도록 재정의하라. Lombok의 `@ToString`.

# 제네릭
- 선언부에 type parameter가 포함된 클래스나 인터페이스는 제네릭<sup>generic</sup> 클래스나 인터페이스라고 부르며 JDK 1.5에 도입되었다.
- unchecked warning은 ClassCastException이 발생할 가능성을 나타낸다. `@SupressWarnings("unchecked")`는 확실할때만 사용하고, 발생하지 않도록 typesafe를 보장해야 한다.
- 배열 대신 리스트를 사용하고, 가능하면 제네릭 자료형/메서드로 만든다.
    - 어떤 와일드 카드를 쓸지는 PECS(Produce - Extends, Consumer - Super)를 기억한다. e.g. `<? extends T>`, `<? super T>`

# enums와 annotations
- int 상수 대신 `enum`을 사용하라.
- naming pattern 대신 annotation을 사용하라.
    - JUnit 은 test로 시작하는 메서드 이름을 써야 했지만 자바 1.5 부터 `@Test`를 사용하며, 파라미터를 넘길 수 있는등 장점이 많다.

# methods
- null 대신 빈 배열이나 컬렉션을 반환하라.

# 일반적인 프로그래밍 원칙
- 정확한 답이 필요하다면 float와 double은 피하라
```
>>> 0.3 * 3
0.8999999999999999
```
파이썬도 마찬가지, 부동 소수점 연산은 모든 언어가 겪는 공통적인 문제다.
- boxed primitives에 `==` 연산은 값 비교가 아닌 identity comparison이 수행되므로 주의해야 한다.
    - 제네릭과 같은 경우를 제외하고는 항상 기본 자료형<sup>primitive types</sup>을 사용하며, 불필요하게 사용할 경우 성능 저하 이슈가 있다.
- String의 `+=`는 [재할당과 복사가 일어난다.](https://stackoverflow.com/a/22439433/3513266) StringBuilder를 사용해야 한다.
- 객체를 참조할때는 인터페이스를 참조하는 방법이 유연하다.
```
List<Subscriber> subscribers = new Vector<Subscriber>();   // 바람직한 예제
Vector<Subscriber> subscribers = new Vector<Subscriber>(); // 나쁜 예제
```

# 동시성<sup>concurrency</sup>
- `synchronized`는 특정 메서드나 코드 블록을 한 번에 한 스레드만 사용하도록 보장하며, 다른 스레드가 변경 중인 객체의 상태를 관측할 수 없어야 한다는 관점으로 바라본다.
    - 자바 언어 명세에는 long이나 double이 아닌 모든 변수는 atomic하게 읽고 쓸 수 있다고 되어 있다.
    - 이외에는 읽기/쓰기 메서드 모두에 `synchronized` 지정이 필요하다. 또한 변수에 `volatile`로 지정하는 간단한 방법이 있으나 `++` 같은 연산자가 atomic 하지 않아 잘못된 결과를 반환할 수 있다.
    - 가장 좋은 방법은 변경 가능 데이터는 한 스레드만 이용하도록 하는 방법이며, 변경 가능 데이터를 공유할 때는 해당 데이터를 읽거나 쓰는 모든 스레드는 동기화를 수행해야 한다.
- 자바 플랫폼 초기에는 과도한 동기화가 적용되어 있었다. `StringBuffer`가 `StringBuilder`로 대체된 것은 이 때문이며, 이외에도 `HashTable`, `Vector`등이 모두 synchronized로 구현되어 있어 지금은 `HashMap`, `ArrayList`등을 사용한다. 동기화가 필요한 경우 구식이 된 `HashTable` 대신 성능이 훨씬 좋은 `ConcurrentHashMap`을 사용한다.
- 1.5부터 java.util.concurrent가 추가되어 스레드를 직접 쓰는 대신 executor와 task를 사용한다.
```
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.execute(runnable);
```
- `wait()`, `notify()` 대신 concurrent 유틸리티를 이용한다. 예를 들어 concurrent 컬렉션은 concurrency를 높이기 위해 synchronized를 내부적으로 처리한다. 외부에서 처리하는 것은 불가능하며, lock을 걸어봐야 효과가 없을 뿐 아니라 프로그램만 느려진다.

(Effective Java, 2008)

# 클린 코드
『클린 코드』를 [스터디 하면서 정리한 내용](https://github.com/Yooii-Studios/Clean-Code)이 잘 정리되어 있다.

# Spring Initializr
<img src="https://user-images.githubusercontent.com/1250095/152762349-3f19a31c-9a0f-44eb-9be8-f5c5d5d8a9b2.png" width="60%">

[Spring Initilizr](https://start.spring.io/)를 이용해 skeleton 생성. 기본적으로 Spring Web을 이용해 웹 서비스를 생성할 수 있다.

- build.gradle을 수정하면 Load Gradle Changes라는 아이콘이 조그맣게 뜨고 이를 이용해 rebuild 할 수 있다.
- `$ ./gradlew bootRun`으로 실행한다.
- Kotlin 클래스를 추가하면 자동으로 build.gradle에 추가할 것이냐는 툴팁이 뜨고 추가할 수 있다. (Scala는 지원안하면서 반칙) 컨트롤러를 코틀린으로 만드는 것도 가능하다.
- 하단 툴바에서 Endpoints는 RequestMappings를 모두 보여준다. Spring - MVC에서도 조회할 수 있다.

# Books
엄밀히 하면 자바 책은 아니지만 객체지향에 관한 내용을 여기에 함께 정리

## 객체지향의 사실과 오해 <sup>2015</sup>
- 협력하는 객체들의 공동체
    - 객체지향의 본질: 시스템을 상호작용하는 **자율적인 객체들의 공동체**로 바라보고 객체를 이용해 시스템을 분할하는 방법
    - 자율적인 객체란 **상태**와 **행위**를 함께 지니며 스스로 자기 자신을 책임지는 객체를 의미
    - 객체는 시스템의 행위를 구현하기 위해 다른 객체와 **협력**. 각 객체는 협력 내에서 정해진 **역할**을 수행하며 역할은 관련된 **책임**의 집합
    - 객체는 다른 객체와 협력하기 위해 메시지를 전송하고, **메시지**를 수신한 객체는 메시지를 처리하는 데 적합한 **메서드**를 자율적으로 선택 
- 이상한 나라의 객체
- 타입과 추상화
- 역할, 책임, 협력
- 책임과 메시지
- 객체 지도
- 함께 모으기

## Practical 모던 자바 <sup>2020</sup>
- 자바 발전 과정
- 인터페이스와 클래스
- 함수형 프로그래밍
- 람다
- 스트림
- 병렬
- 파일 IO
- 날짜와 시간
- 모듈
- JShell
- 새 기능
- 제네릭
