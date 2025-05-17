---
layout: wiki 
title: Python
tags:  ["Languages & Framework"]
last_modified_at: 2024/09/01 12:59:11
---

<!-- TOC -->

- [Logging](#logging)
- [Python Docker Image Size](#python-docker-image-size)
- [Pythonic way](#pythonic-way)
  - [Array](#array)
- [setup.py install](#setuppy-install)
- [Books](#books)
  - [고성능 파이썬 2013, 2016](#고성능-파이썬-2013-2016)

<!-- /TOC -->

# Logging
콘솔과 파일에 동시에 로깅을 하려면 다음 옵션을 사용:
```python
logging.basicConfig(filename='preprocess.log', level=logging.INFO)
logging.getLogger().addHandler(logging.StreamHandler())
...
logging.info(f'START {i}')
```

# Python Docker Image Size
```
python              3.9-alpine          c89f09476494        4 days ago           44.7MB
python              3.9-slim            ce689abb4f0d        4 days ago           118MB
python              3.9                 254d4a8a8f31        4 days ago           885MB
```

alpine은 사이즈가 가장 작을뿐더러 slim과 달리 `apk`로 패키지를 설치할 수 있어서 매우 유용하다. 그러나 alpine은 `pip` 설치시 wheel 설치가 아니라 소스 tar.gz을 받아서 직접 컴파일을 하기 때문에 패키지 설치에 매우 오랜 시간이 소요된다.

# Pythonic way
아래 내용들은 『파이썬 코딩의 기술 <sub>2015, 2016</sub>』에 나오는 내용들이다.  
[Effective Python 내용 정리](https://github.com/shoark7/Effective-Python)

- map과 filter 대신 list comprehension을 사용하라
- range 보다는 enumerate를 사용하라
- def 에서 `*args`를 사용하면 함수에서 가변 개수 <sup>variable argument</sup>의 위치 인수를 받을 수 있다.
- class에 필요한 다른 생성자를 정의하려면 `@classmethod`를 이용하라
- `super`로 부모 함수를 초기화 하라
- 파이썬의 언어 후크<sup>language hook</sup>를 사용하면 시스템을 연계하는 범용 코드를 쉽게 만들 수 있다.
    - `__getattr__`, `__getattribute__`, `__setattr__`
- `__doc__`이라는 속성에 접근하면 파이썬 프로그램 자체에 포함된 docstring을 추출할 수 있다.
- 최적화 하기 전에 `Profile`을 이용해 프로파일 하라

(Effective Python, 2015)

추가:  

28: 커스텀 컨테이너 타입은 `collections.abc`의 클래스를 상속받게 만들자

42: `functools.wraps`로 함수 데코레이터를 정의하자

`itertools`는 이터레이터를 구성하거나 이터레이터와 상호 작용하는데 유용한 함수를 제공한다.  
`product`: 이터레이터에 있는 아이템들의 카테시안 곱을 반환한다.  
`permutations`, `combinations` 포함

## Array
```python
A = array.array('i', range(1, 10000))
L = list(range(1, 10000))
R = range(1, 10000)
N = np.array(range(1,10000))
```

배열을 설정하고 `sum()` 결과를 확인한다.

```
In [38]: %timeit sum(A)
    ...: %timeit sum(L)
    ...: %timeit sum(R)
    ...: %timeit sum(N)
147 µs ± 1.19 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)
37.9 µs ± 588 ns per loop (mean ± std. dev. of 7 runs, 10000 loops each)
159 µs ± 1.25 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)
748 µs ± 3.72 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```

`array`는 `range`와 아무 차이가 없다. 오히려 리스트가 $$\frac{1}{4}$$수준이다. 뿐만 아니라 `numpy`는 5배 더 느리다. 문제는 "unboxed" 이기 때문이다. [^fn-tim] array, numpy 모두 primitive type을 갖지만 `sum()`이 동작하기 위해서는 python object가 되어야 한다. 자바에도 동일한 boxing/unboxing이 있다. numpy도 그래서 object로 변환되면서 오히려 더 늦다. 따라서, 자료형이 아무리 primitive type이라도 여러 동작을 수행하기 위한 메소드가 primitive type을 처리하지 못한다면 아무 의미가 없다. 따라서 sum 조차 없는 `array`는 그다지 유용하지 않다. 반면 numpy는 풍부한 메소드를 제공하며, sum 또한 자체 numpy의 primitive type을 지원하는 `np.sum()`이 있기 때문에 이를 이용하면 range 대비 30배 더 빠르게 수행할 수 있다.

[^fn-tim]: <https://stackoverflow.com/a/36778655>

```
In [39]: %timeit np.sum(N)  # 또는 N.sum()
5.07 µs ± 36.8 ns per loop (mean ± std. dev. of 7 runs, 100000 loops each)
```

# setup.py install
`pyproject.toml`을 이용해 환경 구성을 하면(kss에 적용) 빌드 환경을 완전히 새로 구성하기 때문에 빌드 시간이 매우 오래 걸린다. incremental build도 제대로 동작하지 않고 처음부터 새로 빌드하는 시간이 소요된다.

# Books
## 고성능 파이썬 <sub>2013, 2016</sub>
컴퓨터 시스템의 기본:  
암달의 법칙 <sup>Amdahl's law</sup>: 멀티 코어에서 작동하도록 설계된 프로그램 일지라도 하나의 코어에서 실행되어야 하는 루틴이 존재하고, 더 많은 코어를 투입해도 병목으로 작용한다는 법칙.

예를 들어, 1분이 소요되는 설문 조사를 100명을 대상으로 했을때, 조사원이 2명이라면 50분에 끝낼 수 있겠지만 조사원이 200명이라도 1분 이내로 끝낼 수는 없다. 여기서 1분의 루틴이 존재한다. p.24

파이썬의 성능:  
1. 파이썬 객체가 메모리에 최적화된 상태로 저장되지 않는다. 메모리를 자동으로 할당/해제하는 GC를 사용하는데, 이는 CPU 캐시에 데이터를 전송하는데 영향을 미치는 메모리 단편화를 일으킨다.
1. 동적 타입을 사용하며, 컴파일 되지 않는다. 이 문제를 극복하는 방법 중에는 Cython이 대표적이다.
1. GIL: CPU의 병렬 연산이 불가능하다. 이 문제는 멀티쓰레드가 아닌 멀티프로세스(mulprocessing 모듈 사용)를 사용해서 회피할 수 있다.

p.33

파이썬을 쓰는 이유:  
표현력이 좋고 배우기 쉽다. 파이썬 라이브러리는 타 언어로 작성된 도구를 감싸서 다른 시스템도 쉽게 호출할 수 있도록 하고 있다. 예를 들어 scikit-learn은 C로 작성된 liblinear, libsvm 사용. NumPy는 BLAS와 또 다른 C, Fortran 라이브러리 포함.

프로파일링:  
데코레이터(자바의 어노테이션)를 활용한 시간 측정. line_profiler가 좋아 보이지만 최근 버전에서 잘 동작하지 않는듯 하다. (확인 필요)

더 효율적인 탐색:  
팀 정렬은 다양한 정렬 알고리즘을 활용하여 주어진 데이터에 어떤 알고리즘을 적용하는 것이 최선인지를 추측하는 휴리스틱을 사용한다(더 자세히 말하자면, 삽입 정렬과 병합 정렬 알고리즘을 조합해서 사용한다)  
p.90

<img src="https://user-images.githubusercontent.com/1250095/68547287-24716880-0423-11ea-9ade-0840a4a8a2d9.jpg" width="70%">

리스트의 경우 꽉 찼을때 리사이징을 한다고 언급한다.

해시 테이블에서 데이터가 얼마나 균등하게 분포되어 있는지를 로드 팩터라 하며, 해시 함수의 엔트로피와 관련 있다. p.106 최소 충돌일때 당연히 엔트로피는 최대가 된다.

Dict의 최소 크기는 8이다. 이 크기는 50,000까지는 4배씩 증가하고 그 뒤로는 2배씩 증가한다. (위 사진은 List이므로 혼동 x)

`8, 32, 128, 512, 2048, 8192, 32768, 131072, 262144, ...`

많은 항목이 삭제되면 크기가 줄어들 수도 있다.

JIT vs. AOT:  
미리 컴파일 하는 방식 <sup>ahead of time</sup>으로 Cython, 적절한 때에 컴파일 하는 방식 <sup>just in time</sup>으로 Numba, PyPy가 있따. GCC, Clang등은 당연히 AOT 방식.

이 책은 multiprocessing 모듈에 대해 많은 부분을 할애하고 있다.
