---
date: 2020-10-01
title: NES CPU Emulator
weight: 2
tags: ["gamedev", "c++"]
image: "nes1.gif"
---

I built a NES CPU Emulator called [SCONES](https://github.com/Zephilinox/SCONES) using C++17 (well, it was meant to be a full emulator, but I only got as far as the CPU 😂)

It uses an instruction table that leverages a templated function pointer as a non-type template parameter to create multiple different functions for each type of instruction, and has tests to ensure the CPU is working correctly
