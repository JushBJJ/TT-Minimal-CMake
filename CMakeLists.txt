# Bookmarks
# (1) Project Name
# (2) Executable

cmake_minimum_required(VERSION 3.16)
cmake_policy(VERSION 3.16)

############################################
# Check whether TT_METAL_HOME and ARCH_NAME
# environemnt variables are set
############################################

if("$ENV{TT_METAL_HOME}" STREQUAL "")
    message(FATAL_ERROR "TT_METAL_HOME environment variable is not set. Please do:\n\t1) export TT_METAL_HOME=<path_to_tt_metal>\n\t2) source <path_to_tt_metal>/setup.sh")
endif()
if("$ENV{ARCH_NAME}" STREQUAL "")
    message(FATAL_ERROR "ARCH_NAME environment variable is not set. Please do:\n\texport ARCH_NAME=<arch_name>")
endif()

############################################
# Project setup
############################################
include(cmake/compilers.cmake)

if (DEFINED ENV{CMAKE_C_COMPILER} AND DEFINED ENV{CMAKE_CXX_COMPILER})
    message(STATUS "Setting C and C++ compiler from environment variables")
    set(CMAKE_C_COMPILER $ENV{CMAKE_C_COMPILER})
    set(CMAKE_CXX_COMPILER $ENV{CMAKE_CXX_COMPILER})
endif()

if (CMAKE_CXX_COMPILER AND CMAKE_C_COMPILER)
    message(STATUS "Using specifed C++ compiler: ${CMAKE_CXX_COMPILER}")
    message(STATUS "Using specifed C compiler: ${CMAKE_C_COMPILER}")
else()
    message(STATUS "No C or C++ compiler specified, defaulting to Clang-17")
    FIND_AND_SET_CLANG17()
endif()

# Set project after compiler setup'
# (1) Set the name of your tt project name
project(example-tt-project LANGUAGES CXX C)
CHECK_COMPILERS()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(cmake/dependencies.cmake)

add_library(stdlib INTERFACE)
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_link_libraries(stdlib INTERFACE c++ c++abi)
    target_compile_options(stdlib INTERFACE -stdlib=libc++)
else()
    target_link_libraries(stdlib INTERFACE stdc++)
endif()

add_library(metal_common_libs INTERFACE)
target_link_libraries(metal_common_libs INTERFACE
    dl z pthread atomic hwloc numa stdlib # system libraries, hwloc has no cmake support, find_package won't find it
)

add_library(linker_flags INTERFACE)

add_library(compiler_warnings INTERFACE)
target_compile_options(compiler_warnings INTERFACE -Werror -Wdelete-non-virtual-dtor -Wreturn-type -Wswitch -Wuninitialized -Wno-unused-parameter)

ADJUST_COMPILER_WARNINGS()

add_library(compiler_flags INTERFACE)
target_link_libraries(compiler_flags INTERFACE compiler_warnings stdlib)
target_compile_options(compiler_flags INTERFACE -mavx2 -fPIC -DFMT_HEADER_ONLY -fvisibility-inlines-hidden -fno-lto)

add_library(metal_header_directories INTERFACE)
target_include_directories(metal_header_directories INTERFACE $ENV{TT_METAL_HOME}/tt_metal/hw/inc)
target_include_directories(metal_header_directories SYSTEM INTERFACE ${reflect_SOURCE_DIR})

if ("$ENV{ARCH_NAME}" STREQUAL "wormhole_b0")
    target_include_directories(metal_header_directories INTERFACE tt_metal/hw/inc/wormhole
        tt_metal/hw/inc/wormhole/wormhole_b0_defines
        ${UMD_HOME}/device/wormhole
        ${UMD_HOME}/src/firmware/riscv/wormhole
    )
else()
    target_compile_options(compiler_flags INTERFACE -DDISABLE_ISSUE_3487_FIX)
    target_include_directories(metal_header_directories INTERFACE
        tt_metal/hw/inc/$ENV{ARCH_NAME}
        ${UMD_HOME}/device/$ENV{ARCH_NAME}
        ${UMD_HOME}/src/firmware/riscv/$ENV{ARCH_NAME}
    )
endif()


############################################################################################################################
# Our actual project
############################################################################################################################

# (2)
# Don't forget to rename "example-tt-project" to whatever you want to call your project Name
# Rename `main.cpp` to the .cpp files you want to compile

add_executable(example-tt-project
                main.cpp
                # others.cpp
)

target_include_directories(example-tt-project PRIVATE
                            metal_header_directories
                            ${reflect_SOURCE_DIR}
                            $ENV{TT_METAL_HOME}
                            $ENV{TT_METAL_HOME}/build/lib
                            $ENV{TT_METAL_HOME}/tt_metal
                            $ENV{TT_METAL_HOME}/tt_metal/common
                            $ENV{TT_METAL_HOME}/tt_metal/third_party/umd
                            $ENV{TT_METAL_HOME}/tt_metal/hw/inc/$ENV{ARCH_NAME}
                            $ENV{TT_METAL_HOME}/tt_metal/hw/inc/
                            $ENV{TT_METAL_HOME}/tt_metal/third_party/umd/src/firmware/riscv/$ENV{ARCH_NAME}
                            $ENV{TT_METAL_HOME}/tt_metal/detail
                            $ENV{TT_METAL_HOME}/tt_metal/hostdevcommon
                            $ENV{TT_METAL_HOME}/tt_metal/include
                            $ENV{TT_METAL_HOME}/tt_metal/third_party/
                            $ENV{TT_METAL_HOME}/tt_metal/third_party/fmt
                            $ENV{TT_METAL_HOME}/tt_metal/third_party/pybind11/include
)

target_link_directories(example-tt-project PRIVATE
    $ENV{TT_METAL_HOME}/build/lib
)

target_link_libraries(example-tt-project PRIVATE
                        stdlib
                        metal_common_libs
                        linker_flags
                        compiler_flags
                        tt_metal
                        yaml-cpp
)
