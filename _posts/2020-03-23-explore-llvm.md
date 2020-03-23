---
title:  "Exploring LLVM for iOS"
category: knowledge_base
tags: [knowledge_base]
date: 2020-03-22
---

> This page is work in progress

While browsing through YouTube, I found a video discussing a unknown yet important topic: `LLVM`

Here is the [link to the video](https://www.youtube.com/watch?v=IR_L1xf4PrU) if you'd like to watch it.

## What is LLVM?

By definition, LLVM is set of compiler and toolchain technologies, which can be used to develop a front end for any programming language and a back end for any instruction set architecture.

Basically, it is a middle-man between your high-level programming language and your computer's machine language, translating your code into machine code. Written in C++, it is used by many different platforms and tools, with one of them being Xcode. 

--- 
Xcode first introduced LLVM in Xcode 3.2 for Snow Leopard. Apple's LLVM compiler uses Clang as well as GCC, and have been improving its capability for the past few years. 

What's really interesting about LLVM is:
- It's design is very modular, allowing it to be adapted and reused very easily. This comes especially when you're looking to develop a compiler to target a new hardware or with a new language. 
- LLVM has its own intermediate representation of the code called LLVM IR. I found this especially interesting; we'll get back to it later. 

### So what is LLVM IR?