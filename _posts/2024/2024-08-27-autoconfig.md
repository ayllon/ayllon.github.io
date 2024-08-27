---
title: AutoConfig in SonarQube
---

After [SonarCloud's automatic analysis for C++]({% post_url 2023/2023-08-17-autoscan-c++ %}),
SonarQube now has, since 10.6, a similar feature called [AutoConfig](https://www.sonarsource.com/blog/autoconfig-cpp-code-analysis-redefined/) for C++.

Unlike "Automatic Analysis", "AutoConfig" allows the user to manually define
macros, set the target architecture, or to point to their own set of dependencies.
Other than that, most of the heavy lifting is shared between both: computing the set of
non-conflicting macros that  cover the most cost possible (measured in tokens) and a
hardened analyzer capable of handling incomplete code (i.e., missing types or functions 
declarations).

**Yes, but why?**

Adding support for a compiler is burdensome and time consume. Some times it is not even
possible unless some agreement is reached (propietary compilers with non public 
documentation).

This work is necessary to figure which macros are predefined by the compiler, for instance.
Or to understand the flags in order to properly handled type sizes (`long` has not the same 
size in Linux than in Windows, as a trivial example; and the size of a pointer depends
on the architecture).

AutoConfig objective is to allow users to get some level of analysis without having
to wait for compiler-specific logic to be added.
