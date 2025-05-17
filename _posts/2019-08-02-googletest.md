---
layout: post
title: Google C++ Testing Framework
tags:  ["Software Engineering"]
last_modified_at: 2019/08/02 00:00:00
---

<div class="message">
C++ 신규 프로젝트를 진행하면서, Google Test를 빌드하고 설치한 과정을 정리한다.
</div>

<small>
*2019년 8월 2일 초안 작성*  
</small>

<!-- TOC -->

- [설치](#설치)
- [코드](#코드)
- [참고](#참고)

<!-- /TOC -->

## 설치
<https://github.com/google/googletest/releases>에서 최신 버전을 다운로드 하고 아래와 같이 압축을 풀고 빌드한다.

```bash
$ tar xzvf release-1.8.1.tar.gz
$ cd googletest-release-1.8.1/
$ mkdir bld
$ cd bld
$ cmake ../googletest/
$ make
```

이후 libgtest.a, libgtest_main.a 두 파일이 생성된다. Google Mock도 동일한 방식으로 빌드한다. 마찬가지로 libgmock.a, libgmock_main.a 두 파일이 생성된다. 

메인 프로젝트에 빌드한 Google Test 라이브러리와 `include`를 복사한다.

```
├── googletest
│   ├── include
│   │   ├── gmock
│   │   │   ├── gmock.h
│   │   │   └── ...
│   │   └── gtest
│   │       ├── gtest.h
│   │       └── ...
│   └── lib
│       ├── libgmock.a
│       ├── libgmock_main.a
│       ├── libgtest.a
│       └── libgtest_main.a
```

메인 프로젝트의 CMakeLists.txt에는 `include`와 `lib` 경로를 설정해준다. `link_directories`를 지정하면 `LD_LIBRARY_PATH`를 지정한 것과 유사한 역할을 한다.

```cmake
cmake_minimum_required(VERSION 2.8)
project(sentence_splitter_cpp)

set(CMAKE_CXX_STANDARD 11)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/googletest/include)
link_directories(${CMAKE_CURRENT_SOURCE_DIR}/googletest/lib)

set(SOURCE_FILES sentence_splitter.cpp)

add_executable(sentsplit main.cpp ${SOURCE_FILES})
add_executable(testcase test_sentence.cpp ${SOURCE_FILES})
target_link_libraries(testcase gtest gtest_main gmock)
```

메인 바이너리는 Google Test를 포함하지 않는 가벼운 바이너리를 생성하고, Google Test는 전용 바이너리를 만들어 라이브러리 링킹을 함께 한다. 이 중 `gtest_main`은 `main()` 함수 없이도 목업을 만들어주는 역할을 한다. 따라서 아래 코드의 주석 처럼 `main()` 함수 생략이 가능하다.

## 코드
테스트케이스를 포함한 test_sentence.cpp의 코드는 아래와 같다.
```c++
#include "split_sentence.h"
#include "gtest/gtest.h"
#include "gmock/gmock.h"

TEST(Sentence, Sentence_Input_Test) {
    ASSERT_THAT(std::vector<std::string>{"ABC"},
                testing::ElementsAre("ABC"));
}

// If you use `gtest_main`, you do not need the main() function below.
int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

`std::vector`에 대한 값은 위 코드 처럼 [Google Mock을 사용](https://stackoverflow.com/a/2797990)하면 비교할 수 있다. Mock 오브젝트 생성 외에도 이 처럼 다른 용도로 활용이 가능하다.

## 참고
`ASSERT`와 `EXPECT`는 실패 발생시 [계속 진행 여부에 차이](https://stackoverflow.com/a/2565309)가 있다.
> Usually `EXPECT_*` are preferred, as they allow more than one failures to be reported in a test. However, you should use `ASSERT_*` if it doesn't make sense to continue when the assertion in question fails.