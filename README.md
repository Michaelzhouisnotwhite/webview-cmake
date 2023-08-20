# Webview Cmake

A cross-platform cmake interface of <https://github.com/webview/webview.git>

## Usage

1. create a file named CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.20)
project(webview_test)
set(CMAKE_CXX_STANDARD 17)
include(FetchContent)

```