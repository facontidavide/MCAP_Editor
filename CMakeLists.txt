# Copyright (C) 2022 The Qt Company Ltd.
# SPDX-License-Identifier: BSD-3-Clause

cmake_minimum_required(VERSION 3.16)
project(mcap_editor LANGUAGES CXX)

set(ZSTD_MULTITHREAD_SUPPORT OFF CACHE BOOL "MULTITHREADING SUPPORT")
add_subdirectory(3rdparty/zstd-1.5.5)

# find dependencies
find_package(ament_cmake REQUIRED)

set(LZ4_BUILD_LEGACY_LZ4C OFF CACHE BOOL "Build legacy lz4c")
set(LZ4_BUILD_CLI OFF CACHE BOOL "Build lz4 program")
add_subdirectory(3rdparty/lz4-1.9.4/build/cmake)

option(COMPILING_TO_WASM "Set to ON if compiling to WASM" OFF)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(QT_WASM_EXTRA_EXPORTED_METHODS specialHTMLTargets)

find_package(Qt6 REQUIRED COMPONENTS Widgets)

qt_add_executable(mcap_editor
    src/main.cpp
    src/mainwindow.cpp
    src/mainwindow.h
    src/mainwindow.ui
    src/mcap_impl.cpp
    src/bytearray_writable.hpp
    src/resources.qrc)

target_link_libraries(mcap_editor PRIVATE
    Qt::Core
    Qt::Gui
    Qt::Widgets
    libzstd_static
    lz4_static
)


if(EMSCRIPTEN)
    target_compile_definitions(mcap_editor PRIVATE USING_WASM=1)
endif()

target_include_directories(mcap_editor PRIVATE
     $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/3rdparty>
     $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/3rdparty/lz4-1.9.4/lib>
     $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/3rdparty/zstd-1.5.5>
     $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/3rdparty/mcap-1.3.0/include>
)

if(COMPILING_TO_WASM)
  target_link_options(mcap_editor PUBLIC -sASYNCIFY)
endif()

install(
    TARGETS mcap_editor
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    DESTINATION lib/${PROJECT_NAME}
)
install(DIRECTORY
  3rdparty
  DESTINATION share/${PROJECT_NAME}
)


ament_package()