cmake_minimum_required(VERSION 3.16..3.30)

############################################
# Project setup
############################################
include(cmake/compilers.cmake)

if(DEFINED ENV{CMAKE_C_COMPILER} AND DEFINED ENV{CMAKE_CXX_COMPILER})
    message(STATUS "Setting C and C++ compiler from environment variables")
    set(CMAKE_C_COMPILER $ENV{CMAKE_C_COMPILER})
    set(CMAKE_CXX_COMPILER $ENV{CMAKE_CXX_COMPILER})
endif()

if(CMAKE_CXX_COMPILER AND CMAKE_C_COMPILER)
    message(STATUS "Using specifed C++ compiler: ${CMAKE_CXX_COMPILER}")
    message(STATUS "Using specifed C compiler: ${CMAKE_C_COMPILER}")
else()
    message(STATUS "No C or C++ compiler specified, defaulting to Clang-17")
    FIND_AND_SET_CLANG17()
endif()

project(
    MyFooProject
    VERSION 0.1.0
    LANGUAGES CXX
)

add_executable(my_foo example.cpp)

if(NOT DEFINED ENV{TT_METAL_HOME})
    message(FATAL_ERROR "Please set TT_METAL_HOME environment variable")
endif()

if(NOT DEFINED ENV{ARCH_NAME})
    message(FATAL_ERROR "Please set ARCH_NAME to grayskull, wormhole_b0, or blackhole")
endif()

############################################
# Dependencies setup
############################################
list(PREPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
set(ENV{CPM_SOURCE_CACHE} "${PROJECT_SOURCE_DIR}/.cpmcache")
include(CPM)

if(CMAKE_VERSION VERSION_LESS 3.25)
    add_subdirectory(dependencies EXCLUDE_FROM_ALL)
else()
    add_subdirectory(dependencies EXCLUDE_FROM_ALL SYSTEM)
endif()

find_library(NUMA_LIBRARY NAMES numa REQUIRED)

############################################
# Target configuration
############################################

set_target_properties(my_foo PROPERTIES
    CXX_STANDARD 20
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
)

target_compile_definitions(my_foo PRIVATE
    ARCH_$<UPPER_CASE:$ENV{ARCH_NAME}>
    FMT_HEADER_ONLY
)

target_compile_options(my_foo PRIVATE
    -Werror
    -Wno-deprecated-declarations
    -Wdelete-non-virtual-dtor
    -Wreturn-type
    -Wswitch
    -Wuninitialized
    -Wno-unused-parameter
    -mavx2
    -fPIC
    -fvisibility-inlines-hidden
    -fno-lto
    # Clang-specific options
    $<$<CXX_COMPILER_ID:Clang>:-Wsometimes-uninitialized>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-c++11-narrowing>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-error=local-type-template-args>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-delete-non-abstract-non-virtual-dtor>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-c99-designator>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-shift-op-parentheses>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-non-c-typedef-for-linkage>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-deprecated-this-capture>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-deprecated-volatile>
    $<$<CXX_COMPILER_ID:Clang>:-Wno-deprecated-builtins>
    # GCC-specific options
    $<$<CXX_COMPILER_ID:GNU>:-Wno-deprecated>
    $<$<CXX_COMPILER_ID:GNU>:-Wno-attributes>
    $<$<CXX_COMPILER_ID:GNU>:-Wno-stringop-overread>
    $<$<CXX_COMPILER_ID:GNU>:-Wno-stringop-overflow>
    $<$<CXX_COMPILER_ID:GNU>:-Wno-maybe-uninitialized>
    $<$<CXX_COMPILER_ID:GNU>:-Wno-missing-requires>
    $<$<CXX_COMPILER_ID:GNU>:-Wno-narrowing>
    $<$<CXX_COMPILER_ID:GNU>:-Wno-non-template-friend>
    $<$<CXX_COMPILER_ID:GNU>:-Wno-error=non-template-friend>
)

target_include_directories(my_foo PRIVATE
    $ENV{TT_METAL_HOME}
    $ENV{TT_METAL_HOME}/tt_metal
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/device/api/
    $ENV{TT_METAL_HOME}/tt_metal/hw/inc/$ENV{ARCH_NAME}
    $ENV{TT_METAL_HOME}/tt_metal/hw/inc/
    $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/src/firmware/riscv/$ENV{ARCH_NAME}
    $ENV{TT_METAL_HOME}/tt_metal/detail
    $ENV{TT_METAL_HOME}/tt_metal/hostdevcommon
    $ENV{TT_METAL_HOME}/tt_metal/include
    $ENV{TT_METAL_HOME}/tt_metal/third_party/
    $ENV{TT_METAL_HOME}/tt_metal/third_party/fmt
    $ENV{TT_METAL_HOME}/tt_metal/third_party/pybind11/include

    # TTNN
    $ENV{TT_METAL_HOME}/ttnn/cpp
    $ENV{TT_METAL_HOME}/ttnn/cpp/ttnn/deprecated
    $ENV{TT_METAL_HOME}/tt_metal/third_party/magic_enum

)

target_link_directories(my_foo PRIVATE
    $ENV{TT_METAL_HOME}/build/lib
)

target_link_libraries(my_foo PRIVATE
    fmt
    range-v3
    magic_enum
    Reflect::Reflect
    yaml-cpp
    Boost::core
    Boost::smart_ptr
    Boost::container
    dl
    z
    pthread
    atomic
    hwloc
    numa
    tt_metal
    $ENV{TT_METAL_HOME}/build/lib/_ttnn.so
    $ENV{TT_METAL_HOME}/build/lib/libdevice.so
    $ENV{TT_METAL_HOME}/build/lib/libuv.so.1
    $ENV{TT_METAL_HOME}/build/lib/libnng.so.1
    $ENV{TT_METAL_HOME}/build/lib/libtt_metal.so
)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(my_foo PRIVATE -O0 -g -DDEBUG)
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_options(my_foo PRIVATE -O3)
elseif(CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
    target_compile_options(my_foo PRIVATE -O3 -g -DDEBUG)
else()
    message(WARNING "Defaulting to Release build")
    target_compile_options(my_foo PRIVATE -O3)
endif()

if(ENABLE_LIBCXX)
    target_compile_options(my_foo PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-stdlib=libc++>)
    if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        target_link_options(my_foo PRIVATE -lc++ -lc++abi)
    endif()
else()
    target_compile_options(my_foo PRIVATE -fsized-deallocation)
endif()
