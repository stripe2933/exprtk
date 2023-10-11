cmake_minimum_required(VERSION 3.26)
project(exprtk)

set(CMAKE_CXX_STANDARD 23)

# Check compiler compatibility.
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 17.0)
    # On Clang 17, you should manually set the LLVM build dir (LIBCXX_BUILD) with prebuilt Standard Library Modules.
    # See https://libcxx.llvm.org/Modules.html .

    # Turn on experimental module features.
    if (CMAKE_VERSION VERSION_LESS "3.27.0")
        set(CMAKE_EXPERIMENTAL_CXX_MODULE_CMAKE_API "2182bf5c-ef0d-489a-91da-49dbc3090d2a")
    elseif (CMAKE_VERSION VERSION_LESS "3.28.0")
        set(CMAKE_EXPERIMENTAL_CXX_MODULE_CMAKE_API "aa1f7df0-828a-4fcd-9afc-2dc80491aca7")
    endif()

    set(CMAKE_EXPERIMENTAL_CXX_MODULE_DYNDEP 1)
    set(CMAKE_CXX_STANDARD_REQUIRED YES)
    set(CMAKE_CXX_EXTENSIONS OFF)

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

    add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-fprebuilt-module-path=${CMAKE_BINARY_DIR}/_deps/std-build/CMakeFiles/std.dir/>)
    add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-nostdinc++>)
    add_compile_options($<$<COMPILE_LANGUAGE:CXX>:-isystem>)
    add_compile_options($<$<COMPILE_LANGUAGE:CXX>:${LIBCXX_BUILD}/include/c++/v1>)

    add_link_options($<$<COMPILE_LANGUAGE:CXX>:-nostdlib++>)
    add_link_options($<$<COMPILE_LANGUAGE:CXX>:-L${LIBCXX_BUILD}/lib>)
    add_link_options($<$<COMPILE_LANGUAGE:CXX>:-Wl,-rpath,${LIBCXX_BUILD}/lib>)

    link_libraries(std c++)
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 19.36)
    # In MSVC, there's nothing to do manually.
    # Set CMAKE_CXX_STANDARD to 23 will enable Standard Library Modules automatically.
else()
    message(FATAL_ERROR "C++20 module is not supported with given compiler.")
endif()

add_library(exprtk)
target_sources(exprtk
    PUBLIC
        FILE_SET cxx_modules TYPE CXX_MODULES FILES
            exprtk.cppm
)

# Set EXPRTK_STANDALONE_BUILD flag if it is unset and project is top-level project.
if (NOT DEFINED EXPRTK_STANDALONE_BUILD)
    set(EXPRTK_STANDALONE_BUILD (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR))
endif()

if (EXPRTK_STANDALONE_BUILD)
    add_subdirectory(test)
    add_subdirectory(benchmark)
    add_subdirectory(examples)
endif()