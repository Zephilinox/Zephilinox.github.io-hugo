---
date: 2021-09-01
title: WASM C++ & CMake Template
weight: 2
tags: ["c++", "library"]
---

[Demo](https://ricardoheath.co.uk/emscripten-cpp-cmake-template/). I released [this](https://github.com/Zephilinox/emscripten-cpp-cmake-template) Github Template to set up a new repo that had full support for running C++ on the web via WASM, using Emscripten, alongside all the required libraries, integrations, best practices, and build systems with CI/CD

<!--more-->

A template for using emscripten with C++20 and CMake, and using `GLAD` and `SDL2`

- Emscripten web build, deployed to `gh-pages`. [View demo here](https://ricardoheath.co.uk/emscripten-cpp-cmake-template/)
- Linux, Mac, and Windows native builds via `Github Actions`
- Code coverage via `CodeCov`
- Code formatting via `clang-format`
- Static Analysis using `cppcheck` and `clang-tidy`
- Dynamic Analysis using `Address Sanitizer`, `Leak Sanitizer`, `Thread Sanitizer`, and `Undefined Behaviour Sanitizer`
- Unit tests ran using `Valgrind` for runtime memory analysis
- Caching builds via `ccache`
- Caching CMake dependencies via [CPM](https://github.com/cpm-cmake/CPM.cmake)
- Multi-project (Monorepo) setup, with `MyLib` for e.g. a game engine, and `MyApp` for e.g the gameplay code
- Unit tests via [Doctest](https://github.com/onqtam/doctest)
- Benchmarks via [Google Benchmark](https://github.com/google/benchmark)
- Strict compiler warnings due to [cmake/CompilerWarnings](https://github.com/Zephilinox/emscripten-cpp-cmake-template/blob/main/cmake/CompilerWarnings.cmake), taken from [cpp_starter_project](https://github.com/lefticus/cpp_starter_project/blob/master/cmake/CompilerWarnings.cmake)
- No third-party warnings due to [cmake/SetSystemIncludes](https://github.com/Zephilinox/emscripten-cpp-cmake-template/blob/main/cmake/SetSystemIncludes.cmake) and [cmake/CompilerWarnings](https://github.com/Zephilinox/emscripten-cpp-cmake-template/blob/main/cmake/CompilerWarnings.cmake)
- Building [SDL2](https://github.com/libsdl-org/SDL) from source
