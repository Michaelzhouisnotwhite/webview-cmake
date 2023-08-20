# Webview CMake

A cross-platform cmake interface of <https://github.com/webview/webview.git>

## Get Started

1. Create a project directory use `mkdir webview_hello` and `cd webview_hello`.
2. Create `CMakeLists.txt` and `webview_hello.cpp` file.
3. Add following to `CMakeLists.txt`.
```cmake
cmake_minimum_required(VERSION 3.20)
project(webview_hello)
set(CMAKE_CXX_STANDARD 17)
include(FetchContent)
FetchContent_Declare(
    webview
    GIT_REPOSITORY https://github.com/Michaelzhouisnotwhite/webview-cmake.git
    GIT_TAG main
)
FetchContent_MakeAvailable(webview)

add_executable(webview_hello webview_hello.cpp)

target_link_libraries(webview_hello webview)

```

4. add following c++ code to `webview_hello.cpp` see [webview example](https://github.com/webview/webview/blob/master/examples/bind.cc)
```c++
#include "webview.h"
#ifdef WIN32
#include <windows.h>
#include <dwmapi.h>
#endif
#include <chrono>
#include <filesystem>
#include <fstream>
#include <string>
#include <thread>


using namespace std::literals;

constexpr const auto html =
    R"html(
<button id="increment">Tap me</button>
<div>You tapped <span id="count">0</span> time(s).</div>
<button id="compute">Compute</button>
<div>Result of computation: <span id="compute-result">0</span></div>
<script>
  const [incrementElement, countElement, computeElement, computeResultElement] =
    document.querySelectorAll("#increment, #count, #compute, #compute-result");
  document.addEventListener("DOMContentLoaded", () => {
    incrementElement.addEventListener("click", () => {
      window.increment().then(result => {
        countElement.textContent = result.count;
      });
    });
    computeElement.addEventListener("click", () => {
      computeElement.disabled = true;
      window.compute(6, 7).then(result => {
        computeResultElement.textContent = result;
        computeElement.disabled = false;
      });
    });
  });
</script>)html";


#ifdef _WIN32
int WINAPI WinMain(HINSTANCE hInt, HINSTANCE hPrevInst, LPSTR lpCmdLine, int nCmdShow) {
#else
int main() {
#endif
    unsigned int count = 0;
    webview::webview w(true, nullptr);
    w.set_title("Example");
    w.set_size(800, 800, WEBVIEW_HINT_MIN);

    w.bind("increment", [&](const std::string& /*req*/) -> std::string {
        auto count_string = std::to_string(++count);
        // std::this_thread::sleep_for(1s);
        return "{\"count\": " + count_string + "}";
    });

    // An binding that creates a new thread and returns the result at a later time.
    w.bind(
        "compute",
        [&](const std::string& seq, const std::string& req, void* /*arg*/) {
            // Create a thread and forget about it for the sake of simplicity.
            std::thread([&, seq, req] {
                // Simulate load.
                std::this_thread::sleep_for(std::chrono::seconds(1));
                // json_parse() is an implementation detail and is only used here
                // to provide a working example.
                auto left = std::stoll(webview::detail::json_parse(req, "", 0));
                auto right = std::stoll(webview::detail::json_parse(req, "", 1));
                auto result = std::to_string(left * right);
                w.resolve(seq, 0, result);
            }).detach();
        },
        nullptr);

    w.set_html(html);
    w.run();

    return 0;
}
```
4. Make sure your current dir is in webview_hello. Create a folder named `build` and `cd build`. Run `cmake ..` and `cmake build`. 
5. You will see your app use webview.