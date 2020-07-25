---
title:  "How are comments processed in the compiler?"
category: knowledge_base
tag: [programming, compilers]
date: 2020-07-26
---

Comments are important. They are essential. They are useful. And, they should be used frequently. 

We write and read a lot of comments when programming. It is used to explains parts of a code, or explain how certain interfaces are used. 

Sometimes, they may be too many comments, even more so if you add something like javadocs, that it starts to concern us, "is this being processed in the compiler as well and slowing down the build time?" 

Not really.

## Compiling process

Before we learn how comments are treated in the build process, we must first know how our source code is turned into an executable code. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200726/compilation-process.png" alt="">

#### Preprocessor

First, the source code is passed into the preprocessor. Preprocessor expands the code, handling the preprocessor directives. These include `#include`, `#define` and other macro syntaxes. The preprocessor replaces the macros with the corresponding files and selects appropriate code lines using the `#if` directives. 

In this phase, the preprocessor also runs through the code, taking out the useful code and ignoring the useless ones. It removes all newlines and whitespaces from the code. 

Here, when the preprocessor encounters the start of a comment, such as `//` or `/*`, it *throws out all the following code **until** it meets a newline character(for `//`) or a closing comment character `*/`*. 

Also, compilation process may include a `Lexical analyzer` that is used to divide the code into tokens, and feed it to the parser for creating a symbol table. 

When breaking the source code into tokens, the lexical analyzer detects comments that matches correct syntax (`//` or `/* */`) and ignores them until a newline or `*/`. The comments basically become a whitespace, and they are effectively ignored from the lexical analyzer. 

When the source code goes through the first process of the compilation, **Preprocessor**, the comments are already removed from the code, and as they go through the later stages of compilation, comments will not create any delay in the process. 

Even though te above process applies to most language and compilers, *some* language and compilers have special options that embeds information inside the comments and include the comments throughou the compilation process. 

---

There are so many more to the process of compiling, and linking. We'll look more into the techniques and processes that the compilers use in order to optimize our code for usage. 