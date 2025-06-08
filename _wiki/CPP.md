---
layout: wiki 
title: C++
tags:  ["Languages & Framework"]
last_modified_at: 2025/06/08 19:29:11
---

<!-- TOC -->

- [Modern C++](#modern-c)
- [매크로](#매크로)
  - [inline](#inline)
- [Generic Programming](#generic-programming)
  - [Templates](#templates)
    - [Template Metaprogramming](#template-metaprogramming)
    - [Template Specialization](#template-specialization)
    - [Functors](#functors)
  - [OOP](#oop)
    - [pure virtual function](#pure-virtual-function)
    - [객체의 클래스 알아내기](#객체의-클래스-알아내기)
- [스마트 포인터](#스마트-포인터)
  - [부실 레퍼런스stale reference](#부실-레퍼런스stale-reference)
  - [댕글링 포인터dangling pointer](#댕글링-포인터dangling-pointer)
  - [포인터 vs 참조](#포인터-vs-참조)
  - [우측값 참조rvalue reference](#우측값-참조rvalue-reference)
  - [Stack Memory, Heap Memory](#stack-memory-heap-memory)
- [Lambda](#lambda)
- [기타](#기타)
- [SFINAE](#sfinae)
- [include 기본 경로](#include-기본-경로)
- [Memory Allocation](#memory-allocation)
- [Books](#books)
  - [모던 C++로 배우는 함수형 프로그래밍 2017, 2018](#모던-c로-배우는-함수형-프로그래밍-2017-2018)

<!-- /TOC -->

# Modern C++
대부분의 내용은 『Effective Modern C++, 2014』 에서 가져왔다.

- `{}` curly brackets, braces로 초기화 하라
    - `()` Round brackets, parentheses의 경우 함수 선언과 혼동된다.
    - `std::vector<>` 생성시 괄호와 중괄호의 차이가 있다.
```
std::vector<int> v1(10, 20); // 값이 20인 요소 10개짜리 vector 생성
std::vector<int> v2{10, 20}; // 값이 각각 10, 20인 vector 생성
```
- 0과 NULL보다 nullptr를 선호하라
Using **nullptr** avoids overload resolution surprises<sup>잘못 해석된 오버로드</sup>
- `typedef` 대신에 `using`을 사용하라.
- 삭제된 함수<sup>deleted function</sup>를 선호하라.
```
public:
    basic_ios(const basic_ios&) = delete;
```
private 선언과 차이가 크다. 멤버 함수나 friend 함수에서 basic_ios 객체를 복사하려 하면 컴파일이 실패한다. 이는 링크 시점에 가서야 발견되는 C++98 방식에 비해 개선된 것이다.
- 재정의 함수들은 `override`를 선언하라. 
- 가능하면 항상 `constexpr`을 사용하라.  
변수를 constexpr로 선언하면 컴파일러는 컴파일 시점 상수임을 보장한다. 모든 constexpr 객체는 const 이지만 모든 const 객체가 constexpr인 것은 아니다.
- const 멤버 함수는 thread-safe하게 작성하라. (`std::lock_guard<std::mutex>` mutex lock을 사용하여)

# 매크로
매크로는 C에서는 매우 중요하지만 C++에서는 용도가 훨씬 적어졌다. 매크로에 대한 첫 번째 원칙은 어쩔 수 없는 경우가 아니라면 사용하지 말라는 것이다. 매크로는 컴파일러가 보기도 전에 프로그램 텍스트를 재배열하기 때문에 많은 프로그래밍 지원 도구 입장에서도 큰 골칫거리이기도 하다. 따라서, 매크로를 사용할때는 디버거, 상호 참조 도구, 프로파일러 같은 하위 수준의 서비스만 기대해야 한다. (TCPPL, 2013)
- 오버로딩 될 수 없다.
- 재귀적 호출을 처리할 수 없다.
- 자신의 개인적인 언어를 설계할 수 있지만, 다른 사람들은 이해하지 못할 것이다.
- 가능하면 `#pragma`는 피하는 편이 최선이다.

## inline
C는 매크로로 실행 최적화를 하는데 반해 C++은 `inline` 함수로 최적화 한다. 컴파일러는 런타임에 함수 정의를 참조하는 대신 컴파일 타임에 메서드 바디를 해당 위치에 삽입한다. 만약 크기가 너무 큰 메서드를 인라이닝 하면 해당 호출 위치마다 큰 바디의 중복 코드가 삽입되면서 실행 바이너리가 너무 커져 버린다.<sup>code bloat</sup> (전문가를 위한 C++, 2011)

# Generic Programming
제네릭 프로그래밍은 파라미터로 제공되는 특정 타입에 필요한 경우 나중에 구체화 될<sup>to-be-specified-later </sup> 타입을 지정하여 알고리즘을 작성하는 컴퓨터 프로그래밍 스타일로 재사용성을 높일 수 있다.

## Templates
```
template<typename T> T sum(T a, T b) { return a + b; } 
```

호환성을 위해 `class`도 허용하지만 `typename`을 사용한다.

Java에는 **Generics**가 있으며, C++과 달리 하나의 컴파일된 코드를 만든다. 

```
<T extends Something> T Sum(T a, T b) { return a.add(b); }
```

### Template Metaprogramming
C++ 컴파일러는 **컴파일 타임**에 클래스 템플릿을(클래스 메타 코드) 이용해 실제 클래스(클래스 정의 코드)를 생성한다.

<img src="https://www.geeksforgeeks.org/wp-content/uploads/gq/2014/06/templates-cpp.jpg" width="50%">

C++는 Generic Programming 테크닉을 구현하기 위해 템플릿을 사용하는데, 템플릿은 런타임이 아닌 컴파일 타임에 일부 코드를 사전 평가 하는 방법인 템플릿 메타프로그래밍<sup>template metaprogramming</sup>에도 사용된다.

Java의 Generics도 type error를 유발해 compilation-time error(run-time error가 아니라)를 내기 위한 동기로 1998년에 시작됐다.

### Template Specialization
[특정 데이터 타입에 대해 다른 알고리즘을 정의](https://www.geeksforgeeks.org/template-specialization-c/)한다.  
이 부분은 추가 학습 필요

### Functors
**Functors** are objects that can be treated as though they are a function or function pointer. 

## OOP
### pure virtual function
pure virtual fuction이 적용되어 있으면 상속만 가능하다. override를 미리 선언하는 효과가 있으며, Java는 명시적으로 interface와 class를 구분하는데 반해 C++은 pure virtual fuction으로 interface와 거의 유사한 클래스를 정의할 수 있다.
```
// pure virtual function
virtual 멤버 함수의 선언 = 0;
```

virtual 메서드만이 올바르게 오버라이딩 될 수 있다. virtual을 선언하지 않은 메소드의 경우 슈퍼클래스를 참조할 경우 서브클래스의 메소드 오버라이딩 여부를 알지 못하고, 따라서 슈퍼클래스의 메소드가 실행된다. 

Java는 모든 메서드가 virtual이며, C++도 모든 메서드를 virtual로 선언하는 것을 권고한다. 과거에는 vtable에서 오버라이딩 여부를 찾는 부가 작업을 최소화하고자 지정해야만 동작하는 결정을 내렸지만, 이제 그 정도의 오버헤드는 무시할 정도로 작다.

하지만 모든 메서드에 매 번 virtual을 선언하는 것은 현실적으로 어려운 일이다. (전문가를 위한 C++, 2011)

### 객체의 클래스 알아내기
```
#include <typeinfo>
...
if (typeid(*pVc[i]) == typeid(Car))
...
typeid(*pVc[i]).name() // 클래스 이름을 조사한다.
```

# 스마트 포인터
스마트 포인터는 자동 메모리 관리, 바운드 체킹등의 추가 기능을 제공하면서 포인터를 시뮬레이트 하는 추상 데이터 유형<sup>abstract data type</sup>이다.

메모리 해제등의 심각한 버그를 줄일 수 있는 장점이 크다. 디폴트 생성자로 초기화도 자동으로 할 수 있다. 예전에는 `std::auto_ptr`와 연산자 오버로딩을 통해 직접 구현했으나 모던 C++에서는 `std::unique_ptr`과 `std::shared_ptr`등을 제공한다. `std::weak_ptr`는 순환 참조를 방지할 수 있다.

스마트 포인터는 **RAII**의 variation이다.

- `new`를 직접 사용하는 것 보다 `std::make_unique`를 선호하라. make_shared와 달리 make_unique는 C++14에 포함되었다. 아래와 같은 형태로 직접 작성이 가능하다.
    - new를 사용하면 생성할 객체의 형식이 되풀이해서 나오지만, make 함수 버전은 그렇지 않다.
```
template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args)
{
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```

## 부실 레퍼런스<sup>stale reference</sup>
```
double &square_ref(double d) {
    double s = d * d;
    return s;
}
```
지역 변수는 함수내에서만 유용하다. square_ref 함수의 결과는 더 이상 존재하지 않는 지역 변수 s를 참조하게 되며 우연히 저장되어 있던 메모리가 남아 있을때만 동작한다. **로컬 변수가 리턴 됐다는 경고**가 발생한다.

## 댕글링 포인터<sup>dangling pointer</sup>
```
double *square_ptr(double d) {
    double s = d * d;
    return &s;
}
```
이 포인터는 스코프를 벗어난 지역 주소를 갖는다. **로컬 변수가 리턴 됐다는 경고**가 발생한다.

## 포인터 vs 참조
구글 가이드는 함수 인자로 포인터 또는 상수형 참조자만 쓴다.
```
void Foo(const string &in, string *out);
```
그러나 [비야네는 포인터를 쓰지 말고 가능하면 스마트 포인터와 참조자를 쓸 것을 권유](http://minjang.github.io/2016/03/21/talk-with-stroustrup/)한다.

포인터를 사용할 수 밖에 없는 유일한 상황은 가리키는 대상을 바꾸어야만 할 때 뿐이다. (전문가를 위한 C++, 2011)

## 우측값 참조<sup>rvalue reference</sup>
임시 객체일때 참조가 허용되지 않지만 rvalue reference는 deep copy 대신 포인터 주소만 shallow copy를 하여 오버헤드를 줄이고 상수등에도 이용할 수 있다. 하지만 함수 파라미터외에 일반 변수에 이용하는 경우는 흔치 않다.
```
int &&i = 4;
```

`std::move`는 rvalue로 무조건 캐스팅을 수행한다.`std::forward`는 주어진 인수가 rvalue에 묶인 경우에만 rvalue로 캐스팅한다. 둘 다, run-time에는 아무 일도 하지 않는다. 

- rvalue에는 `std::move`를, universal reference에는 `std::forward`를 사용하라.
- universal reference에는 overloading을 피하라.

## Stack Memory, Heap Memory
- Stack은 CPU 디자인의 결과로 LIFO를 따르며 할당/취소가 매우 빠르다. 로컬 변수가 위치한다. 또한 함수로 넘겨진 파라미터가 존재한다. 스택은 일반적으로 CPU 캐시에 있기 때문에 선호된다. 
- Heap은 Stack이 할당된 후 남겨진 메모리다. 일반적으로 new로 할당 된 C++ 객체나 malloc과 같은 방식으로 할당 되는 메모리 블록이 위치한다.

# Lambda
- default capture mode를 피하라.
- `std::forward`를 통해 전달할 `auto&&` 매개변수에는 `decltype`을 사용하라.
- `std::bind`를 사용하는 것보다 람다가 더 읽기 쉽고 표현력이 좋다. 그리고 더 효율적일 수 있다.

# 기타
- `std::string`은 almost container 이다.
- C++의 `struct`는 `class`와 똑같이 선언하고, 클래스에 있는 모든 기능을 사용할 수 있다. 차이점은 struct의 기본형은 public이라는 점이다.
- `static`은 유일한 하나의 변수만 존재하며, 다른 곳에서 수정할 경우 값이 함께 변한다.
- `extern`은 모든 소스 파일에서 전역적으로 접근할 수 있으나, 전역 변수는 혼란스러울 뿐만 아니라 버그의 온상이다. 따라서 전역 변수는 static 클래스 멤버로 대체하여 이용하는 것이 바람직하다.
- 잘못된 타입으로 인한 오류를 방지하기 위해 `auto`를 적극 활용하고, 어떤 타입인지는 `static_cast` 또는 부득이하게 런타임 일때는 `dynamic_cast`로 타입을 명시하는게 가장 가독성이 좋다. (Effective Modern C++, 2014)

# SFINAE
"Substitution Failure Is Not An Error"  
추가 정리 필요

# include 기본 경로
```
$ echo "" | g++ -xc - -v -E
...
#include <...> search starts here:
 /usr/local/include
 /Library/Developer/CommandLineTools/usr/bin/../lib/clang/8.1.0/include
 /Library/Developer/CommandLineTools/usr/include
 /usr/include
 /System/Library/Frameworks (framework directory)
 /Library/Frameworks (framework directory)
...
```

# Memory Allocation
- `memset` sets the bytes in a block of memory to a specific value.
- `malloc` allocates a block of memory.
  - `mmap`: In this respect an anonymous mapping is similar to malloc, and is used in some malloc implementations for certain allocations. 엔진에서는 이걸로 메모리를 할당한다.
- `calloc`, same as malloc. Only difference is that it initializes the bytes to zero.

# Books
## 모던 C++로 배우는 함수형 프로그래밍 <sub>2017, 2018</sub>
- 모던 C++ 소개: range-based for loop  
- 함수형 프로그래밍에서 함수 다루기  
first order function, pure function, currying등 함수형 프로그래밍의 핵심을 다룬다.
- 함수에 불변 객체 사용하기  
mutable을 immutable로 바꾸는 방법. 이를 위해 first order와 pure function 적용
- 재귀 함수 호출  
recursion은 iteration으로 구현할 수 있지만 recursion이 보다 함수형에 가깝다. 
- 지연 평가로 실행 늦추기  
지연 실행 및 캐시와 메모이제이션 소개
- 메타프로그래밍으로 코드 최적화
- 동시성을 이용한 병렬 실행  
데드락을 방지하기 위한 동기화 기법
- 함수형 방식으로 코드 작성하기
