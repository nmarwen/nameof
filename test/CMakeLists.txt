﻿set(SOURCES test.cpp)

include_directories(3rdparty/Catch2)
include_directories(${CMAKE_SOURCE_DIR}/include)

set(OPTIONS "")

set(HAS_CPP11_FLAG FALSE)
set(HAS_CPP14_FLAG FALSE)
set(HAS_CPP17_FLAG FALSE)
set(HAS_CPP20_FLAG FALSE)
set(HAS_CPPLATEST_FLAG FALSE)

if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    set(OPTIONS /W4)

    if(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19.00)
        set(HAS_CPP14_FLAG TRUE)
        set(HAS_CPPLATEST_FLAG TRUE)
    endif()

    if(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19.11)
        set(HAS_CPP17_FLAG TRUE)
        set(OPTIONS ${OPTIONS} /permissive-)
    endif()
elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    set(CMAKE_VERBOSE_MAKEFILE ON)
    set(OPTIONS -Wall -Wextra -pedantic-errors)

    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
        if(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.8.0)
            set(HAS_CPP11_FLAG TRUE)
        endif()
        if(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9.2)
            set(HAS_CPP14_FLAG TRUE)
        endif()
        if(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 7.1.0)
            set(HAS_CPP17_FLAG TRUE)
        endif()
    endif()

    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        if(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 3.3.0)
            set(HAS_CPP11_FLAG TRUE)
        endif()
        if(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 3.4.0)
            set(HAS_CPP14_FLAG TRUE)
        endif()
        if(NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 5.0.0)
            set(HAS_CPP17_FLAG TRUE)
        endif()
    endif()
endif()

function(make_target target std)
    add_executable(${target} ${SOURCES})
    target_compile_options(${target} PRIVATE ${OPTIONS})
    if(std)
        if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
            target_compile_options(${target} PRIVATE /std:${std})
        else()
            target_compile_options(${target} PRIVATE -std=${std})
        endif()
    endif()
endfunction()

if(HAS_CPP11_FLAG)
    make_target(nameof-cpp11.t c++11)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    make_target(nameof-cpp11.t "")
endif()
add_test(NAME nameof-cpp11.t COMMAND nameof-cpp11.t)

if(HAS_CPP14_FLAG)
    make_target(nameof-cpp14.t c++14)
    add_test(NAME nameof-cpp14.t COMMAND nameof-cpp14.t)
endif()

if(HAS_CPP17_FLAG)
    set(std17 c++17)
    if(CMAKE_CXX_COMPILER_ID MATCHES "AppleClang")
        set(std17 c++1z)
    endif()
    make_target(nameof-cpp17.t ${std17})
    add_test(NAME nameof-cpp17.t COMMAND nameof-cpp17.t)
endif()

if(HAS_CPPLATEST_FLAG)
    make_target(nameof-cpplatest.t c++latest)
    add_test(NAME nameof-cpplatest.t COMMAND nameof-cpplatest.t)
endif()
