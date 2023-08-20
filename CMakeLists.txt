cmake_minimum_required(VERSION 3.10)
project(webview)

# set(CMAKE_CXX_STANDARD 17)
include(FetchContent)

message("--- fetching webview repository...")

FetchContent_Declare(
    webviewcc
    GIT_REPOSITORY https://github.com/webview/webview.git
    GIT_TAG master
)

FetchContent_GetProperties(webviewcc)
message("--- fetch webview repository completed")

if(NOT webview_POPULATED)
    # Library does not have a CMake build script
    # We have to do it ourselves
    FetchContent_Populate(webviewcc)
    add_library(webview INTERFACE ${webviewcc_SOURCE_DIR}/webview.cc)

    # target_sources(webview INTERFACE ${webview_SOURCE_DIR}/webview.h)
    target_include_directories(webview INTERFACE ${webviewcc_SOURCE_DIR})
    message(DEBUG "-------${webviewcc_SOURCE_DIR}")
    # Set compile options
    # See: https://github.com/webview/webview/blob/master/script/build.sh
    if(WIN32)
        message("--- platform is: Windows")
        message("--- fetching windows webview2...")
        FetchContent_Declare(
            webview_native
            URL https://www.nuget.org/api/v2/package/Microsoft.Web.WebView2
            DOWNLOAD_EXTRACT_TIMESTAMP TRUE
        )

        FetchContent_MakeAvailable(webview_native)
        message("--- fetch windows webview2 completed")
        target_compile_definitions(webview INTERFACE WEBVIEW_EDGE)
        target_include_directories(webview INTERFACE ${webview_native_SOURCE_DIR}/build/native/include)

        # See: https://github.com/webview/webview/blob/master/script/build.bat
        target_link_libraries(webview INTERFACE "-mwindows -ladvapi32 -lole32 -lshell32 -lshlwapi -luser32 -lversion")

    # target_link_libraries(webview INTERFACE "-mwindows -L./build/libs/webview2/build/native/x64 -lwebview -lWebView2Loader")

    # target_compile_options(...) ?
    elseif(APPLE)
        # not tested
        target_compile_definitions(webview INTERFACE WEBVIEW_COCOA)
        target_compile_definitions(webview INTERFACE "GUI_SOURCE_DIR=\"${CMAKE_CURRENT_SOURCE_DIR}\"")
        target_compile_options(webview INTERFACE -Wno-all -Wno-extra -Wno-pedantic -Wno-delete-non-abstract-non-virtual-dtor)
        target_link_libraries(webview INTERFACE "-framework WebKit")
    elseif(UNIX)
        message("--- build on unix like machine")
        EXECUTE_PROCESS(COMMAND pkg-config --cflags --libs gtk+-3.0 webkit2gtk-4.0
            TIMEOUT 5
            OUTPUT_VARIABLE WEB_LIBS
            OUTPUT_STRIP_TRAILING_WHITESPACE
        )
        target_compile_definitions(webview INTERFACE WEBVIEW_GTK)
        target_compile_options(webview INTERFACE -Wall -Wextra -Wpedantic)
        message(DEBUG "--- link librarys: ${WEB_LIBS}")
        target_link_libraries(webview INTERFACE ${WEB_LIBS})
    endif()
endif()