project(test_package)
cmake_minimum_required(VERSION 3.0)

set(CMAKE_CXX_STANDARD 11)

set(CMAKE_VERBOSE_MAKEFILE TRUE)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup(NO_OUTPUT_DIRS)

message(STATUS "Configuring Flex test package targets")

set(FLEX_TARGET_COMPILE_FLAGS "")
if(MSVC)
    set(FLEX_TARGET_COMPILE_FLAGS "--wincompat")
endif()

find_package(FLEX)

set(FLEX_VARS
    FLEX_FOUND
    FLEX_EXECUTABLE
    FLEX_LIBRARIES
    FLEX_INCLUDE_DIRS
)

foreach(FLEX_VAR ${FLEX_VARS})
    message("${FLEX_VAR}: ${${FLEX_VAR}}")
    if(NOT ${FLEX_VAR})
        message(WARNING "${FLEX_VAR} NOT FOUND")
    endif()
endforeach()


FLEX_TARGET(TestParser basic_nr.l "${CMAKE_CURRENT_BINARY_DIR}/lexer.cpp"
    COMPILE_FLAGS "${FLEX_TARGET_COMPILE_FLAGS}")

add_executable(flex_test_package
    ${FLEX_TestParser_OUTPUTS}
)

enable_testing()
add_test(TestLexer
    flex_test_package "${CMAKE_CURRENT_LIST_DIR}/basic_nr.txt"
)

message(STATUS "Configuring Bison test package targets")

find_package(BISON)

set(BISON_VARS
    BISON_FOUND
    BISON_EXECUTABLE
    BISON_VERSION
)

foreach(BISON_VAR ${BISON_VARS})
    message("${BISON_VAR}: ${${BISON_VAR}}")
    if(NOT ${BISON_VAR})
        message(WARNING "${BISON_VAR} NOT FOUND")
    endif()
endforeach()

bison_target(bison_parser_target
    mc_parser.yy "${CMAKE_CURRENT_BINARY_DIR}/mc_parser.cpp"
)

add_executable(bison_test_package
    "${CMAKE_CURRENT_BINARY_DIR}/mc_parser.cpp"
    dummy_lex.cpp
)
target_include_directories(bison_test_package
    PRIVATE
        "${CMAKE_CURRENT_BINARY_DIR}"
)

enable_testing()
add_test(TestBison
    bison_test_package
)
