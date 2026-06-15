cmake_minimum_required(VERSION 3.22.0)

project(vs-nlm-hip VERSION 0.1 LANGUAGES HIP CXX)

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

target_link_libraries(vsnlm_hip PRIVATE hip::host)

find_package(PkgConfig QUIET MODULE)
find_package(Python3 REQUIRED COMPONENTS Interpreter)

if(PKG_CONFIG_FOUND)
    pkg_search_module(VS vapoursynth)
    if(VS_FOUND)
        message(STATUS "Found VapourSynth R${VS_VERSION}")
        execute_process(
            COMMAND "${Python3_EXECUTABLE}" -c "import vapoursynth; print(vapoursynth.get_plugin_dir())"
            OUTPUT_VARIABLE VS_PLUGIN_DIR
            OUTPUT_STRIP_TRAILING_WHITESPACE
        )
        target_include_directories(vsnlm_hip PRIVATE ${VS_INCLUDE_DIRS})
        install(TARGETS vsnlm_hip LIBRARY DESTINATION ${VS_PLUGIN_DIR})
    endif()
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