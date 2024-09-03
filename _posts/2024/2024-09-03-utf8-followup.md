---
title: UTF-8 in C++, followup
---

As a follow-up on the post "[Wait What... (UTF-8 in gcc)]({% post_url 2022/2022-09-27-wait-what %})", it turns out that since [P1949R7](https://wg21.link/p1949r7) "C++ Identifier Syntax using Unicode Standard Annex 31" the snipped I showed is not allowed anymore in C++, retroactively, since it is a

> Defect Report against C++ 20 and earlier

Clang already refused the use of these characters since [version 14](https://github.com/llvm/llvm-project/issues/54732). Gcc, since [version 12](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=100977), although it seems to complain only when `-Wpedantic` is used.
