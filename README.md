# crtsys

C/C++ Runtime library for system file (Windows Kernel Driver)

커널 드라이버에서 C++ 및 STL 기능을 사용할 수 있도록 도와주는 CRT 라이브러리 입니다.

- [crtsys](#crtsys)
  - [Overview](#overview)
  - [Requirements](#requirements)
  - [Test Environments](#test-environments)
  - [Goal](#goal)
    - [C++ Standard](#c-standard)
      - [STL](#stl)
    - [C Standard](#c-standard-1)
    - [NTL (NT Template Library)](#ntl-nt-template-library)
  - [Build](#build)
  - [Test](#test)
  - [Usage](#usage)
    - [CMake](#cmake)
      - [CMakeLists.txt](#cmakeliststxt)
    - [main.cpp](#maincpp)
  - [TODO](#todo)

## Overview

커널 드라이버에서 C++ 및 STL을 사용할수있도록 도와주는 기능을 조사해보았는데, 그중 사용하기 가장 잘 구현되어있는 프로젝트는 아래와 같습니다.

(22-04-26 기준)

- [UCXXRT](https://github.com/MiroKaku/ucxxrt)
  - 장점
    - 타 프로젝트와는 다르게 [Microsoft STL](https://github.com/microsoft/STL)를 직접 사용한다는 개념 자체가 좋아보임.
  - 단점
    - 현재까지는 Win32 API와 연관성이 적은 std::string, std::vector 클래스 등 자료형이나 자료구조 관련 기능만 지원됨.
    - vcxproj이 개발 환경에 종속되기 때문에 VS 버전이 바뀌면 일일이 수작업으로 수정해줘야함.
    - x86에서 [이 코드](https://en.cppreference.com/w/cpp/language/throw#Example)를 실행하는 중 Hang이 발생함
    - Microsoft의 소스 코드를 그대로 복사하여 상단에 자신의 라이센스 주석만 넣어서 프로젝트에 포함시킨것으로 보이며 라이센스 문제에서 자유롭지 못해보임
      - 라이센스를 변조하는 것과 **(Microsoft CRT 및 STL 소스 코드를 재배포하는 행위는 허가되지 않은것으로 파악됨)**

- [KTL](https://github.com/DymOK93/KTL)
  - 장점
    - CMake 사용됨
      - 개발 환경에 맞는 프로젝트 파일 생성 및 자동화가 용이함
    - CRT를 포함한 모든 코드를 독자적으로 구현
      - 라이센스로부터 자유로우며, 예외처리 코드 부분이 경량화되어 스택을 절약할 수 있음
    - 커널에 특화된 템플릿 라이브러리 제공
      - 커널에서만 존재하는 매커니즘에 대한 템플릿 라이브러리를 제공하기 때문에, 실제 드라이버 제작 시에는 STL을 보다도 더 애용할만한 기능이 많음
    - 높은 품질의 코드
  - 단점
    - Microsoft STL을 사용하는 것을 고려해서 작성된것으로 보이나 실제로 Microsoft STL을 사용하도록 변경하는 것에 어려움을 느낌 (오탈자 및 ktl 만 사용하는 코드가 생각보다 많았음)
    - 러시아어 주석인데 파일이 UTF8+BOM로 인코딩되어있지 않아서 러시아 외 국가 환경에서 빌드를 실패함
    - x86 미지원

각 라이브러리의 단점을 보완하는 작업을 진행하던 중

1. UCXXRT은 Microsoft CRT 소스 코드를 그대로 복사하여 프로젝트에 포함시켰다는 점
2. KTL은 x86을 지원하지 않는 점

으로 인해서 새로운 라이브러리를 개발하게 되었습니다.

CRTSYS가 장점은 아래와 같습니다.

1. Micosoft CRT와 STL을 최대한 비슷하게 지원하기 위해서 Microsoft CRT 소스코드를 사용하긴 하지만 Microsoft Visual Studio가 설치되어있는 디렉토리 내의 소스를 직접 빌드하는 방법으로 처리하여, Visual Studio를 합법적으로 사용하는 사용자는 라이센스 문제 없이 사용 가능합니다.
2. Win32 API를 구현한 [Ldk](https://github.com/ntoskrnl7/Ldk)를 활용하여 많은 범위의 STL 기능을 지원합니다.
3. CMake로 프로젝트를 구성할 수 있으며, [CPM](https://github.com/cpm-cmake/CPM.cmake)을 사용하여 정말 간편하게 라이브러리를 사용할 수 있도록 지원합니다.

## Requirements

- Windows 8 or later
- Visual Studio 2017 or later

## Test Environments

- Windows 10
- Visual Studio 2017
  - Visual Studio 2017의 CRT 소스 코드는 일부 헤더가 누락되어 빌드가 되지 않아서, UCXXRT의 코드를 일부 사용하여 지원합니다.
  - 추후 누락된 헤더를 직접 작성하여 지원할 예정입니다.
- Visual Studio 2019
- Visual Studio 2022

## Goal

이 프로젝트는 커널 드라이벌르 작성할 때, 응용 프로그램에서 C++ 및 STL을 사용하는 것에 근접한 개발 경험을 제공하는것을 목표로합니다.

현재 지원되거나 앞으로 지원할 기능은 아래와 같습니다.

- 체크된 항목은 구현 완료된 항목입니다.
- 테스트 코드가 작성된 항목은 테스트 코드가 링크로 설정되어 있습니다.

### C++ Standard

- [ ] Initialization
  - [x] [Non-local variables](https://en.cppreference.com/w/cpp/language/initialization#Non-local_variables)
    - [x] [Static initialization](https://en.cppreference.com/w/cpp/language/initialization#Static_initialization)
    - [x] [Dynamic initialization](https://en.cppreference.com/w/cpp/language/initialization#Dynamic_initialization)
  - [ ] [Static local variables](https://en.cppreference.com/w/cpp/language/storage_duration#Static_local_variables)
    - [ ] thread_local
    - [ ] static
- [x] [catch](https://en.cppreference.com/w/cpp/keyword/catch)
  - [x] [try-block](./test/src/std_test.cpp#L357)
  - [x] [Function-try-block](https://en.cppreference.com/w/cpp/language/function-try-block)

#### STL

- [x] [std::chrono](./test/src/std_test.cpp#L69)
- [x] [std::thread](./test/src/std_test.cpp#L116)
- [x] [std::condition_variable](./test/src/std_test.cpp#L116)
- [x] [std::mutex](./test/src/std_test.cpp#L159)
- [x] [std::shared_mutex](./test/src/std_test.cpp#L206)
- [x] [std::future](./test/src/std_test.cpp#L232)
- [x] [std::promise](./test/src/std_test.cpp#278)
- [x] [std::packaged_task](./test/src/std_test.cpp#L345)
- [ ] std::cout
- [ ] std::cerr

### C Standard

- [x] math functions
  - 직접 구현하기에는 시간이 부족하여 [RetrievAL](https://github.com/SpoilerScriptsGroup/RetrievAL)의 코드를 그대로 사용하였습니다. (SpoilerScriptsGroup 감사합니다! :-))

### NTL (NT Template Library)

커널에서 더 나은 개발 환경을 지원하기 위한 기능을 제공합니다.

- ntl::expand_stack
  - 스택 크기를 확장하기 위한 함수
  - 기본적으로 커널 스택은 사용자 스레드 스택보다 훨씬 작은 크기를 할당받기 때문에 STL 기능을 사용하거나, 특히 throw를 수행할때 사용하는것이 좋습니다.
- ntl::status
  - NTSTATUS에 대한 클래스
- ntl::driver
  - DRIVER_OBJECT에 대한 클래스
  - 기능
    - [x] DriverUnload
    - [ ] DriverDispatch
- ntl::driver_main
  - C++ 용 드라이버 진입점
  - ntl::expand_stack 함수로 스택을 최대 크기로 확장하여 호출됩니다.

## Build

```Batch
git clone https://github.com/ntoskrnl7/crtsys
cd crtsys
cmake -S . -B build
cmake --build build --config Release
```

## Test

1. 아래 명령을 수행하여 라이브러리 및 테스트 코드를 빌드하시기 바랍니다.

    ```Batch
    git clone https://github.com/ntoskrnl7/crtsys
    cd crtsys/test
    cmake -S . -B build
    cmake --build build
    ```

2. build/Debug/crtsys_test.sys를 설치 및 로드하시기 바랍니다.
3. 정상적으로 로드 및 언로드가 되는지 확인하시기 바랍니다.

## Usage

CMake를 사용하는것을 권장합니다.

### CMake

CMake를 사용하신다면 아래와 같이 CMakeLists.txt를 만드시기 바랍니다.

#### CMakeLists.txt

```CMake
cmake_minimum_required(VERSION 3.14 FATAL_ERROR)

project(crtsys_test LANGUAGES C)

include(cmake/CPM.cmake)

set(CRTSYS_NTL_MAIN ON) # use ntl::main
CPMAddPackage("gh:ntoskrnl7/crtsys@0.1.0")
include(${crtsys_SOURCE_DIR}/cmake/CrtSys.cmake)

# add driver
crtsys_add_driver(crtsys_test main.cpp)
```

### main.cpp

간단한 샘플 코드입니다.

- 아래와 같이 CRTSYS_NTL_MAIN을 활성화한다면 ntl::main을 진입점으로 정의하시기 바랍니다. **(권장)**

    ```CMake
    set(CRTSYS_NTL_MAIN ON)
    ```

- 만약 아래와 같이 CRTSYS_NTL_MAIN을 비활성화한다면 기존과 깉이 DriverEntry를 진입점으로 정의하시기 바랍니다.

    ```CMake
    set(CRTSYS_NTL_MAIN OFF)
    ```

아래는 ntl::main를 진입점으로 설정한 프로젝트의 예제 코드입니다.

```C
#include <wdm.h>
#include <ntl/driver>

namespace ntl {
status main(ntl::driver &driver, const std::wstring &registry_path) {

  DbgPrint("load (registry_path : %ws)\n", registry_path.c_str());

  // TODO

  driver.on_unload([registry_path]() {
    DbgPrint("unload (registry_path : %ws)\n", registry_path.c_str());
  });

  return status::ok();
}
} // namespace ntl
```

## TODO

- 아직 구현되지 않은 C++ 및 STL 기능 구현
- Visual Studio 2017의 CRT 소스 코드 빌드
- 커널 드라이버와 사용자 프로세스 간 통신 기능
