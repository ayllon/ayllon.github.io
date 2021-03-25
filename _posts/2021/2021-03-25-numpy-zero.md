---
title: Memory consumption with numpy.zeros
---

I was profiling the memory consumption of a project I want to optimize
for the reasons described [here]({% post_url 2021/2021-03-23-numpy-structured-fields-as-flat %})
and there was a continuous growth that was driving me mad.

{% include image.html file="2021/2021-03-25-numpy-zeros.png"
    description="I am looking at you!"
    alt="Plot of memory consumption"
%}

Why would this line

```python
self.__pdz[idx] += ref_pdz * neighbor.weight
```

steadily increase the memory footprint?


It would seem that `numpy.zeros` calls `calloc` directly, and since the size of
`self.__pdz` is quite noticeable (order of GiB), that triggers directly
a `mmap` call into the kernel. The kernel does *not* give straight away the
memory, however. The previous line will trigger page faults as `idx` moves
around, causing the physical allocation.

TL;DR `calloc` is obviously smart enough to avoid initializing to zeros the
memory when it already knows the kernel will...
