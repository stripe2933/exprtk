cmake_minimum_required(VERSION 3.27)
project(exprtk_module)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_CXX_EXTENSIONS OFF)

if(CMAKE_VERSION VERSION_LESS "3.27.0")
    set(CMAKE_EXPERIMENTAL_CXX_MODULE_CMAKE_API "2182bf5c-ef0d-489a-91da-49dbc3090d2a")
else()
    set(CMAKE_EXPERIMENTAL_CXX_MODULE_CMAKE_API "aa1f7df0-828a-4fcd-9afc-2dc80491aca7")
endif()

include(FetchContent)
FetchContent_Declare(
    std
    URL "file://${LIBCXX_BUILD}/modules/c++/v1/"
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
)
FetchContent_GetProperties(std)
if(NOT std_POPULATED)
    FetchContent_Populate(std)
    add_subdirectory(${std_SOURCE_DIR} ${std_BINARY_DIR} EXCLUDE_FROM_ALL)
endif()

#
# Adjust project compiler flags
#

add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-fprebuilt-module-path=${CMAKE_BINARY_DIR}/_deps/std-build/CMakeFiles/std.dir/>)
add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-nostdinc++>)
# The include path needs to be set to be able to use macros from headers.
# For example from, the headers <cassert> and <version>.
add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-isystem>)
add_compile_options($<$<COMPILE_LANGUAGE:CXX>:${LIBCXX_BUILD}/include/c++/v1>)

#
# Adjust project linker flags
#

add_link_options($<$<COMPILE_LANGUAGE:CXX>:-nostdlib++>)
add_link_options($<$<COMPILE_LANGUAGE:CXX>:-L${LIBCXX_BUILD}/lib>)
add_link_options($<$<COMPILE_LANGUAGE:CXX>:-Wl,-rpath,${LIBCXX_BUILD}/lib>)
# Linking against std is required for CMake to get the proper dependencies
link_libraries(std c++)

add_library(exprtk)
target_sources(exprtk
    PUBLIC
        FILE_SET cxx_modules TYPE CXX_MODULES FILES
            exprtk.cppm
)

add_executable(exprtk_test exprtk_test.cpp)
target_link_libraries(exprtk_test PRIVATE exprtk)

add_executable(exprtk_benchmark exprtk_benchmark.cpp)
target_link_libraries(exprtk_benchmark PRIVATE exprtk)

foreach(example_no RANGE 1 9)
    add_executable(exprtk_simple_example_0${example_no} exprtk_simple_example_0${example_no}.cpp)
    target_link_libraries(exprtk_simple_example_0${example_no} PRIVATE exprtk)
endforeach()
foreach(example_no RANGE 10 19)
    add_executable(exprtk_simple_example_${example_no} exprtk_simple_example_${example_no}.cpp)
    target_link_libraries(exprtk_simple_example_${example_no} PRIVATE exprtk)
endforeach()