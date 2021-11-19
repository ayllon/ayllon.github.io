---
title: Keep your project clean of warnings
---

It is old news that compilation warnings can help
[catching bugs](https://www.cprogramming.com/tutorial/compiler_warnings.html)
early.

But sooner or later there will be "annoying" spurious warnings that
we can safely ignore. And we may do so. But it is a bad idea.
If the warning is a false positive, mark it as such so the linter (or compiler)
ignores it in the future. Or rework the code slightly to avoid the warning.

Even if the warning is harmless, just letting it be will eventually lead to
hundreds of warnings when compiling, and/or on the linter. And we end
learning to just ignore them. And then an important warning goes
unnoticed between the pile of other warnings, and a bug happens.

This happened to me with this small excerpt:

```c++
const std::size_t LIMIT = (2 << 30);
```

The observant reader, and the compiler!, will see `2 << 30` will overflow
(since 2 and 30 are `int`s, so is `2<<30` regardless of what's on the left
hand-side). But a wall of warnings will hide that.
