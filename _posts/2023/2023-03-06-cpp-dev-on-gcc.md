---
date: 2023-03-06 09:00:00 +0900
title: C++ 개발을 굳이 터미널로 해보자
excerpt: 고생하다보면 IDE가 얼마나 대단한지 알 것이다
categories: dev
---

## 사서 고생하게된 계기

2022년 1학기에 운영체제 수업을 들었다. 내가 선택한 교수님은 학생들 죽어나가는
과제를 내시기로 유명한 분으로, 수강신청을 한 학생들 모두 '이번학기는
죽었구나'라는 결연한 마음가짐으로 압도적인 분량과 난이도의 과제를 하게 된다.
코드 한 줄 조차 써내기 어려운 수준이라 매 학기 1/3 정도의 수강생들이 F를
받게 된다.

그 수업의 과제는 특이하게도 리눅스 터미널에서 해야 한다. 학생들이 IDE만 써
왔다는 사실을 아시는만큼 [vim](https://www.vim.org), [cscope](https://cscope.sourceforge.net),
[gcc](https://gcc.gnu.org), [gdb](https://www.sourceware.org/gdb/), [qemu](https://www.qemu.org)
등 개발에 필요한 모든 툴 사용법에 관한 강의 자료와 영상들을 별도로 올려주셨다.
'모른다고? 지금부터 알게 될거야'와 같은 공부와 과제의 무한굴레 속에서 학생들의
피지컬은 나날이 발전하게 된다.

그런 고난을 지나고 나니 터미널에서 다시 IDE로 돌아가기가 아까워졌다. 학기 초
맥북을 구매했을 때도 [Xcode](https://developer.apple.com/xcode/) 없이 [iterm2](https://iterm2.com)로만
개발을 해왔다. (다만 [Swift](https://iterm2.com) 개발을 위해 나중에 설치했다)
[Scala](https://www.scala-sbt.org)나 [Java](https://github.com/eclipse/eclipse.jdt.ls)도
무난히 개발할 수 있었고, 마침 C++을 쓸 일이 생겨서 이번 기회에 개발환경 설정
방법을 정리해보고자 한다.

## Prerequisites

우선 다음과 같은 프로그램이 필요하다. 설치 방법은 알아서 참고.

1. [gcc](https://gcc.gnu.org)
1. [cmake](https://gcc.gnu.org)
1. [vim](https://www.vim.org/download.php) 또는[neo-vim](https://neovim.io)

## 디렉터리 구성

C++ 프로젝트를 만든다고 할 때 정해지진 않았지만 일반적으로 다음과 같은 구조를
가진다.

```
├─ app/
│  ├─ CMakeLists.txt
│  ├─ include/
│  │  ├─ foo.h
│  ├─ src/
│  │  ├─ foo.cc
├─ test/
│  ├─ CMakeLists.txt
│  ├─ foo_test.cc
├─ CMakeLists.txt
├─ main.cc
├─ README.md
```

root 디렉터리에는 어플리케이션 관련 코드가 담길 `/app/`, 테스트 코드가 담길
`/test/` 등이 있다. `/app/`에는 헤더 파일이 담길 `/app/include/`와 cpp 파일이
담길 `/app/src/` 등이 있다.

본격적으로 개발을 시작하면 `foo.h`, `foo.cc`, `foo_test.cc`가 위치한 부분만
접근하게 된다. 나머지는 컴파일 및 실행 단계에 필요한 것 들이다. 우선 저
디렉터리 구조에서 3번이나 등장하는 `CMakeLists.txt`파일 부터 만들어보자.

## CMakeLists.txt

CMakeLists.txt는 `/` (root), `/app/`, `/test/` 총 세 곳에 있다. root
디렉터리의 파일이 전반적인 프로젝트의 설정을 관리하면서, 어플리케이션과
테스트 파일들을 참조하게 된다.

### /CMakeLists.txt

root 디렉터리에 있는 CMakeLists.txt는 다음과 같다.

```cmake
cmake_minimum_required(VERSION 3.21)

project(my_cpp_app VERSION 1.0.0)

include(CTest)

# C++ settings
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Options for libraries
option(USE_APP "Use the App library" ON)
option(USE_GOOGLE_TEST "Use GoogleTest for testing" ON)

# Project Library
add_subdirectory(app)
list(APPEND EXTRA_LIBS app)

# GoogleTest
add_subdirectory(test)

add_executable(${CMAKE_PROJECT_NAME} main.cc)

target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC ${EXTRA_LIBS})

target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC
  "${PROJECT_BINARY_DIR}"
)
```

빌드와 실행파일 생성 등 전반적인 프로젝트의 설정을 담당하고 있다.
`add_subdirectory(...)`를 통해 `/app/`과 `/test/`의 디렉터리에 접근하게 된다.

### /app/CMakeLists.txt

`/app/`에 있는 파일은 다음과 같다.

```cmake
set(APP_HEADER_DIR include)
set(APP_HEADERS
  # Add header files
  ${APP_HEADER_DIR}/foo.h
)

set(APP_SOURCE_DIR src)
set(APP_SOURCES
  # Add source files
  ${APP_SOURCE_DIR}/foo.c
)

add_library(app STATIC ${APP_HEADERS} ${APP_SOURCES})

target_include_directories(app
  PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/${APP_HEADER_DIR}"
)
```

`APP_HEADERS`와 `APP_SOURCES` 변수에 프로그램이 사용할 헤더파일과 소스파일
경로를 일일이 명시해 주어야한다.

### /test/CMakeLists.txt

`/test/`에 있는 파일은 다음과 같다.

```cmake
# GoogleTest
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        release-1.12.1
)
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

set(APP_TESTS
  # Add test files
  foo_test.cc
)

add_executable(app_test ${APP_TESTS})

target_link_libraries(
  app_test
  app
  gtest_main
)

include(GoogleTest)
gtest_discover_tests(app_test)
```

GoogleTest를 활용하느라 설정이 길어졌다. GoogleTest를 다운로드 받은 뒤 테스트
파일을 등록하고, 테스트 케이스를 탐색하게 된다.

## GoogleTest 사용법

[공식 문서](https://google.github.io/googletest/)에 자세한 설명이 나와있지만,
단순하게 쓰려면 다음의 템플릿으로도 충분하다.

```cpp
#include "gtest/gtest.h"
#include "stdio.h"

#include "foo.h"

class MainTestSuite : public ::testing::Test {
  void SetUp() override {
    printf("SetUp\n");
  }

  void TearDown() override {
    printf("TearDown\n");
  }
};

TEST_F(MainTestSuite, foo){
  EXPECT_EQ(addIntegers(1, 1), 1+1);
};
```

## 빌드

우선 root에 `/build/` 디렉터리를 생성해야 한다. 이후 root 디렉터리에서 다음과
같은 명령어를 실행하면 된다.

```sh
# 디렉터리 생성
$ mkdir build

# 빌드파일 생성
$ cmake -S . -B build

# 빌드
$ cmake --build build
```

## 실행

이후 `/build/bin/`에 어플리케이션 및 유닛테스트 파일이 생성된다.
테스트하고자 하는 파일을 다음과 같이 실행하면 된다.

```sh
# "/test/foo_test.cc"를 실행하고 싶다면
$ ./build/bin/foo_test
# 각 테스트 파일 별로 실행파일이 생성된다.

# "/main.cc"를 실행하고 싶다면
$ ./build/bin/my_cpp_app
# project 이름으로 생성된다.
```

## 결론: 대단한 IDE 

지금까지 CMake를 활용해 터미널로 C++ 개발 환경설정을 했다. 사실 개발자
입장에서는 `foo.h`, `foo.cc` 그리고 `foo_test.cc`에만 관심이 있었다. CMake라서
이 정도로 끝났지, [Makefile](https://www.gnu.org/software/make/)로만 했다면
어플리케이션 개발보다 컴파일 하는 데 온 신경이 곤두섰을 것이다. 이러한 일련의
과정들을 자동화 시키고자 IDE가 탄생했다. 그래도 무릇 전공자라면 노가다 좀
해봐야 IDE에 가려진 빌드 과정을 이해할 수 있다고 생각한다.
