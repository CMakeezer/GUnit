#
# Copyright (c) 2016-2017 Kris Jusiak (kris at jusiak dot net)
#
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE_1_0.txt or copy at
# http://www.boost.org/LICENSE_1_0.txt)
#
cmake_minimum_required(VERSION 3.4)
project(GUnit CXX)

option(ENABLE_MEMCHECK "Run the unit tests and examples under valgrind if it is found." OFF)
option(ENABLE_COVERAGE "Run coverage." OFF)

add_custom_target(style)
add_custom_command(TARGET style COMMAND find ${CMAKE_CURRENT_LIST_DIR}/benchmark ${CMAKE_CURRENT_LIST_DIR}/example ${CMAKE_CURRENT_LIST_DIR}/include ${CMAKE_CURRENT_LIST_DIR}/test -iname "*.h" -or -iname "*.cpp" | xargs clang-format -i)

set(CMAKE_CXX_STANDARD 17)

#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wextra")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pedantic")
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pedantic-errors")

if (ENABLE_COVERAGE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --coverage")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0")
endif()

enable_testing()

find_package(GTest REQUIRED)
find_package(GMock REQUIRED)
find_package(gherkin REQUIRED)
find_package(fmem REQUIRED)
find_package(gherkin-cpp REQUIRED)

find_program(MEMORYCHECK_COMMAND valgrind)
if (ENABLE_MEMCHECK AND MEMORYCHECK_COMMAND)
  function(test name scenario)
    string(REPLACE "/" "_" out ${name})
    add_executable(${out} ${CMAKE_CURRENT_LIST_DIR}/${name}.cpp)
    add_test(${out} ${MEMORYCHECK_COMMAND} --leak-check=full --error-exitcode=1 ./${out})
    target_link_libraries(${out} GTest::Main ${GMOCK_BOTH_LIBRARIES} gherkin-cpp::gherkin-cpp)
    target_include_directories(${out} PRIVATE include)
    add_custom_command(TARGET ${out} COMMAND ${scenario} ./${out})
  endfunction()
else()
  function(test name scenario)
    string(REPLACE "/" "_" out ${name})
    add_executable(${out} ${CMAKE_CURRENT_LIST_DIR}/${name}.cpp)
    add_test(${out} ./${out})
    target_link_libraries(${out} GTest::Main ${GMOCK_BOTH_LIBRARIES} gherkin-cpp::gherkin-cpp)
    add_custom_command(TARGET ${out} COMMAND ${scenario} ./${out})
    target_include_directories(${out} PRIVATE include)
  endfunction()
endif()

#test(example/GMock SCENARIO=)
test(example/GTest SCENARIO=)

test(example/GTest-Lite SCENARIO=)
test(example/GSteps SCENARIO=${CMAKE_CURRENT_SOURCE_DIR}/example/GSteps.feature)

#include_directories(test)
test(test/GMake SCENARIO=)
test(test/GMock SCENARIO=)
test(test/GSteps SCENARIO=)
test(test/Features/Calc/Steps/CalcSteps SCENARIO=${CMAKE_CURRENT_SOURCE_DIR}/test/Features/Calc/addition.feature:${CMAKE_CURRENT_SOURCE_DIR}/test/Features/Calc/division.feature)
test(test/Features/Data/Steps/DataSteps SCENARIO=${CMAKE_CURRENT_SOURCE_DIR}/test/Features/Data/data.feature)
test(test/Features/Error/Steps/ErrorSteps SCENARIO=${CMAKE_CURRENT_SOURCE_DIR}/test/Features/Calc/addition.feature:${CMAKE_CURRENT_SOURCE_DIR}/test/Features/Error/error.feature)
test(test/Features/Table/Steps/TableSteps SCENARIO=${CMAKE_CURRENT_SOURCE_DIR}/test/Features/Table/table.feature)
test(test/Features/Tags/Steps/TagsSteps SCENARIO=${CMAKE_CURRENT_SOURCE_DIR}/test/Features/Tags/tags.feature)
test(test/GTest SCENARIO=)
test(test/GTest-Lite SCENARIO=)
test(test/Detail/FileUtils SCENARIO=)
test(test/Detail/Preprocessor SCENARIO=)
test(test/Detail/ProgUtils SCENARIO=)
test(test/Detail/RegexUtils SCENARIO=)
test(test/Detail/StringUtils SCENARIO=)
test(test/Detail/TypeTraits SCENARIO=)
test(test/Detail/Utility SCENARIO=)

include_directories(benchmark)
#test(benchmark/GUnit/test SCENARIO=)
#test(benchmark/gtest/test SCENARIO=)