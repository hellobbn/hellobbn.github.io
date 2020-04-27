---
title: Write a simple compiler [Part 1]
date: 2020-04-27 1
category:
  - Programming
tags:
  - Compiler
  - LLVM
---

Building a C- compiler using bison, flex and LLVM, Part 1: Intro

<!-- more -->

## 1. Intro

The Lab of my _Compiler Principle_ class during 2019 Autumn is to write a compiler for a language called __C minus__, which is a subset of the C programming language. As it is the first year of the lab, the design is not perfect and we had many difficulties trying to understand and complete the tasks.

The good news is that we completed the compiler despite all the difficulties. And since the original code we wrote is messy, I decided to rewrite the compiler on my own, reusing some of the materials and codes provided by TAs as well as written by me and my teammates.

The source code of the compiler `cminusc` can be found in my GitHub: https://github.com/hellobbn/cminusc

This series is the record of my development process, and it might also be a tutorial on how to development a LLVM-backend compiler.

## 2. Prerequisites

- Knowledge on Compiler: lexer, parser, syntax tree, code gen ... etc
- Makefile, GNU make system
- C, C++
- (Optional) LLVM

## 3. Getting started

In this section, I am going to show you different parts in my compiler, so that you can understand the structure of the compiler.

### 3.1 Dir structure

The dir structure of my compiler looks like this:

```bash
.
├── cminusc
├── docs
├── helper
├── include
├── lexer
├── lib
├── parser
├── syntax_tree
├── testcase
└── test_src
```

- The `cminusc` dir contains the "driver" of the compiler (command line parser.. etc)
- The `docs` dir contains the design docs and some notes of the compiler
- The `helper` dir contains come help functions, which are more general
- The `include` dir contains header files for both C and C++ files
- The `lexer` dir contains the source code needed to construct the "Lexer"
- The `parser` dir contains the source code needed to construct the "Parser"
- The `syntax_tree` dir contains the source code to generate the "Syntax Tree" needed for visiter pattern
- The `testcase` dir contains C- testcases (I have not completed this part, yet)
- The `test_src` dir contains some `main` folder that are used to generate testing binaries for Lexer, Parser and Syntax Tree.

### 3.2 Build my source code

The only Makefile of the compiler lies in the root directory. SO to compile the project, simply run the command:

```bash
-> make
```

And the generated compiler binary is in `build/cminus`. In the dir `build` there are also test binarys for different parts of the compiler. We will get to them later as we develop the compiler.

### 3.3 Development process

The process of developing a compiler is very similar to the that of learning the structure of a compiler. We are going to build the "LEXER", followed by "Parser", "Syntax Tree". And then we are going to enter the world of LLVM, where we will be using the LLVM API and "Visiter Pattern" to transfrom the syntax tree to LLVM IR, and build the object and executable files. In the end, we are going to implement a very simple System V calling interface so that our program does not need the libc to work.

## 4. Design Notes & TODO

- No library support or built-in functions
  - meaning that we don't have functions like printf, scanf... etc
- No char support
  - meaning that the interface for System V is not complete
  - We are not able to process string, or character
- No pointer support

However, the structure of the compiler makes it quite easy to expend types and functions. So maybe in the future I will be able to implement those functions. Actually, I have initiated the support for char.