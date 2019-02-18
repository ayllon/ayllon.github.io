---
title: Where is my memory II
---

[Where is my memory I]({%post_url 2019/2019-02-15-where-is-my-memory%})

## Short story
SExtractor is doing just fine! This behavior is caused by glibc malloc.

## Long story

When running with the same number of threads, and the same configuration,
but switching the malloc implementation, we get different behavior.

<figure>
  <a href="{{baseurl}}/img/2019/2019-02-15-where-is-my-memory/heaptrack.png">
    <img src="{{baseurl}}/img/2019/2019-02-15-where-is-my-memory/heaptrack.png" alt="Heaptrack consumed memory"/>
  </a>
  <figcaption>Reminder of the amount of allocated memory as seen from *within*</figcaption>
</figure>

Please, note that, by no means, am I an expert on malloc implementations, so
here I am mostly guessing.

<figure>
  <a href="{{baseurl}}/img/2019/2019-02-15-where-is-my-memory/multi-thread.png">
    <img src="{{baseurl}}/img/2019/2019-02-15-where-is-my-memory/multi-thread.png" alt="Using glibc"/>
  </a>
  <figcaption>Using glibc</figcaption>
</figure>

As far as I can tell, SExtractor can be quite allocation-heavy.

Probably when multi-threading kicks-in, the detection stage is still working
on the detection image looking for sources, so it is allocating stuff.

The measurement threads start trying to allocate as well (i.e. image stamps to
take measures). To avoid contention, glibc will spawn a new allocation arena, and
get the memory chunks from there.

Since the threads are using different heaps, even though the tile manager
is keeping the used memory (as far as it can tell) below the limit, the
resident memory peaks at twice the configured limit, since glibc is allocating
on multiple heaps, and **not** returning unused memory to the system,

<figure>
  <a href="{{baseurl}}/img/2019/2019-02-15-where-is-my-memory/tcmalloc-multi.png">
    <img src="{{baseurl}}/img/2019/2019-02-15-where-is-my-memory/tcmalloc-multi.png" alt="Using tcmalloc"/>
  </a>
  <figcaption>Using tcmalloc</figcaption>
</figure>

[TCMalloc does not return memory to the system](http://goog-perftools.sourceforge.net/doc/tcmalloc.html)
 either (see Caveats), but large allocations are done on the central heap. For
 SExtractor, this is probably the case. Tiles are configured to be, in this case,
 256x256 pixels, 4 bytes each = 256 KiB, which is what TCMalloc considers "large".

 As all these large allocations are done on the same heap, the resident memory
 is kept under the expected value.

<figure>
  <a href="{{baseurl}}/img/2019/2019-02-15-where-is-my-memory/jemalloc-multi.png">
    <img src="{{baseurl}}/img/2019/2019-02-15-where-is-my-memory/jemalloc-multi.png" alt="Using jemalloc"/>
  </a>
  <figcaption>Using jemalloc</figcaption>
</figure>

Jemalloc also uses arenas, and that is probably why there is a very similar
memory increase when multi-threading kicks-in. However, jemalloc
**does** return unused memory to the system via [`madvise`](http://man7.org/linux/man-pages/man2/madvise.2.html).
That is obviously visible on the graph.

As far as I can tell, there are multiple configuration parameters for telling
jemalloc when to return memory to the system (returning straight away can be
wasteful if more allocations are coming later). The defaults seem to be 10 seconds.

## Summary
As the heaptrack graph showed, SExtractor tile manager is behaving
properly, and there are no leaks. The amount of allocated memory - from
the point of view of the tile manager - is what is expected, but the amount
of resident memory depends on how the underlying malloc/free are dealing with
the allocations when running with multiple threads.
