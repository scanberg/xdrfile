cmake_minimum_required(VERSION 3.0)

INCLUDE (CheckTypeSize)

project(xdrfile VERSION 1.1.4 LANGUAGES C)

if (POLICY CMP0063)
    # Use of `<LANG>_VISIBILITY_PRESET` in OBJECT libraries
    cmake_policy(SET CMP0063 NEW)
endif()

check_type_size("int" INT_SIZE)
if (NOT "${INT_SIZE}" MATCHES "^(2|4|8|16)$") # in Bytes; 16, 32, 64, 128 bit
    # See PR #2 for details
    message(FATAL_ERROR "Int size ${INT_SIZE} is not supported")
endif()

set(SOURCES
    src/xdrfile.c
    src/xdrfile_trr.c
    src/xdrfile_xtc.c
    include/xdrfile.h
    include/xdrfile_trr.h
    include/xdrfile_xtc.h
)

add_library(xdrfile OBJECT ${SOURCES})
target_include_directories(xdrfile PUBLIC include)

if (${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
    enable_testing()

    add_executable(xdrfile_test src/xdrfile_c_test.c $<TARGET_OBJECTS:xdrfile>)
    target_include_directories(xdrfile_test PRIVATE include)
    if (UNIX)
        target_link_libraries(xdrfile_test m)
    endif()

    add_test(xdrfile xdrfile_test)
    if(NOT EXISTS "${CMAKE_CURRENT_BINARY_DIR}/test_data")
        file(COPY test_data DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
    endif()
endif()
