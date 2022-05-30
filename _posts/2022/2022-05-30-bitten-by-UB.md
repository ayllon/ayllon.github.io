---
title: Bitten by Undefined Behaviour
---

When packaging [Alexandria](https://github.com/astrorama/Alexandria) for
Fedora, starting with Fedora 35 I started having failures
only on the [`s390x`](https://en.wikipedia.org/wiki/Linux_on_IBM_Z) platform.

After pruning the failing test as much as I could, I reduced the problem to
a few lines like:

```cpp
  double variable = 123.;
  assert(Elements::isEqual(123., variable));
```

In fact, with a snippet like this it would start failing also
on `x86_64`, but only when compiling with link-time optimizations (`-flto`)!

Long story short, `isEqual` has [undefined behaviour](https://github.com/degauden/Elements/pull/16/files#diff-272628e77321098eed03e947247282c6b7b2f024cf6beba8581982d2415c179cL336)
when it casts from double to `UInt`:

```cpp
using Bits  = typename TypeWithSize<sizeof(RawType)>::UInt;
Bits x_bits = *reinterpret_cast<const Bits*>(&x);
```

Note that the UB behaviour is not the pointer cast from `double*` to `UInt*`,
but the *indirection* of the latter.

What is interesting is that this is an example of "nasal demons". Depending
on where and how the code is called, optimized and linked the results
vary wildly.

1. In some cases, when `isEqual` is visible on the same translation unit,
   the optimizer will be able to aggresively optimize away the
   call to `Elements::isEqual(123., variable)`, since it figures out
   it is a tautology and replaces it with `true`.
2. When it is not, on one translation unit the compiler has no idea what goes on
   inside the call, so it will push the two values into the stack and call the
   function. On the other side, the code will execute as one would (but should
   not) expect.
3. With link-time optimization, the compiler will be able to *peek* at what
   `isEqual` is doing. Due to [string aliasing rules](https://en.wikipedia.org/wiki/Aliasing_(computing)#Conflicts_with_optimization),
   it will asume that the pointer to `UInt` has nothing to do with the pointer
   to `double`. It will conclude that the `double` is not used, and just skip
   pushing the values into the stack.

Why did it originally fail only on `s390x`? The actual value being compared
was `0.`. Just our of sheer luck the stack happened to be 0 initialized on
other platforms and it didn't matter the caller was not pushing the values
into the stack.


I wonder why [`UndefinedBehaviorSanitizer`](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)
didn't see this, though...
My guess is that the pointer casting *is* defined and the pointer indirection *is*
also defined. What is undefined is what happens if two pointers with different
types point to the same address.
