cmake_minimum_required(VERSION 3.14)
project(grpc_playground)

set(CMAKE_CXX_STANDARD 14)

option(ENABLE_CCACHE "Speed up incremental rebuilds via ccache" ON)

# Update submodules as needed
option(GIT_SUBMODULE "Check submodules during build" ON)

# CMake modules
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")

# Load modules
include(${CMAKE_MODULE_PATH}/git.cmake)
include(${CMAKE_MODULE_PATH}/include_vendor_pkg.cmake)
include(${CMAKE_MODULE_PATH}/include_guard.cmake)

# CCache

if(ENABLE_CCACHE AND (CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU"))
    find_program(CCACHE_FOUND ccache)
    if(CCACHE_FOUND)
        message(STATUS "Using ccache to speed up incremental builds")
        set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
        set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
        set(ENV{CCACHE_CPP2} "true")
    endif()
endif()

# Clang format

file(GLOB_RECURSE SRC_FILES src/*.cpp src/*.hpp)
file(GLOB_RECURSE TEST_FILES src/*.cpp src/*.hpp)
set(ALL_SOURCES ${SRC_FILES} ${TEST_FILES})

find_program(CLANG_FORMAT NAMES "clang-format" DOC "Path to clang-format executable")
if (NOT CLANG_FORMAT)
    message(STATUS "clang-format not found.")
else ()
    message(STATUS "clang-format found: ${CLANG_FORMAT}")
    add_custom_target(
            clang-format
            COMMAND ${CLANG_FORMAT}
            -style=file
            -i
            ${ALL_SOURCES}
    )
endif ()

# Clang tidy

find_program(CLANG_TIDY NAMES "clang-tidy" DOC "Path to clang-tidy executable")
if (NOT CLANG_TIDY)
    message(STATUS "clang-tidy not found.")
else ()
    message(STATUS "clang-tidy found: ${CLANG_TIDY}")
    set(DO_CLANG_TIDY "${CLANG_TIDY}" "-fix;-fix-errors")
endif ()

# Generated sources
add_subdirectory(proto)

# Main sources
add_subdirectory(src)