cmake_minimum_required(VERSION 3.22.0)

project(vs-nlm-hip VERSION 0.1 LANGUAGES HIP CXX)

# Find the HIP package so we can use its exported targets for the CXX compiler
find_package(hip REQUIRED)

add_library(vsnlm_hip SHARED
    source/source.cpp
    source/kernel.hip
)

set_target_properties(vsnlm_hip PROPERTIES
    HIP_EXTENSIONS OFF
    HIP_STANDARD 20
    HIP_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
    CXX_STANDARD 20
    CXX_STANDARD_REQUIRED ON
)

# This is the critical line: it passes the HIP include directories (like /opt/rocm/include) 
# to the CXX compilation phase of source.cpp
target_link_libraries(vsnlm_hip PRIVATE hip::host)

find_package(PkgConfig QUIET MODULE)

if(PKG_CONFIG_FOUND)
    pkg_search_module(VS vapoursynth)

    if(VS_FOUND)
        message(STATUS "Found VapourSynth r${VS_VERSION}")

        cmake_path(APPEND install_dir ${VS_LIBDIR} vapoursynth)
        target_include_directories(vsnlm_hip PRIVATE ${VS_INCLUDE_DIRS})

        install(TARGETS vsnlm_hip LIBRARY DESTINATION ${install_dir})
    endif()
endif()

if(NOT VS_FOUND)
    set(VS_INCLUDE_DIR "" CACHE PATH "Path to VapourSynth headers")

    if(VS_INCLUDE_DIR STREQUAL "")
        message(WARNING "VapourSynth not found")
    endif()

    target_include_directories(vsnlm_hip PRIVATE ${VS_INCLUDE_DIR})

    install(TARGETS vsnlm_hip LIBRARY RUNTIME)
endif()

find_package(Git QUIET)

if(GIT_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} describe --tags --long --always
        WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
        OUTPUT_VARIABLE VCS_TAG
    )
    if(VCS_TAG)
      string(STRIP ${VCS_TAG} VCS_TAG)
    endif()
endif()

if(VCS_TAG)
    message(STATUS "vs-nlm-hip ${VCS_TAG}")
else()
    message(WARNING "unknown plugin version")
    set(VCS_TAG "unknown")
endif()

configure_file(source/config.h.in config.h)

target_include_directories(vsnlm_hip PRIVATE ${CMAKE_CURRENT_BINARY_DIR})