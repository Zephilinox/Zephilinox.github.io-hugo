---
date: 2021-11-01
title: Element Interpreter
weight: 1
tags: ["c++", "c", "csharp", "library"]
---

While at Ultraleap I was responsible for developing an interpreter over a period of 6 months for the Functional Programming language that we open sourced, [Element](https://github.com/ultraleap/Element) - designed to execute highly performant and optimised code on hardware embedded devices within deterministic bounded space and time constraints

<!--more-->

- C++11 Wrapper API (source not available)
- [Stable C89 API](https://github.com/ultraleap/Element/tree/main/libelement/include/element)
- [C++17 library internals](https://github.com/ultraleap/Element/tree/main/libelement/src)
- [Command Line Interface](https://github.com/ultraleap/Element/tree/main/libelement.CLI) to compile and evaluate the language using the interpreter 
- Lexer & Parser
- Type Checker
- AST Optimisations (Constant Folding, Dead Code Elimation, Peephole Optimisation)
- AST Tree walking expression evaluation
- Extensive documentation, testing, code coverage, static analysis, Github CI/CD, and build systems (CMake)
- A number of internal supporting libraries that used or integrated with the libelement interpreter, such as our Handtracking SDK
- Language design discussions
- Reviewing PR's by coworkers using the interpreter or creating other libraries and tools for the language, such as those written for C# & Unity
