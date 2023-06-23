---
title: Memory corruption in Windows with `ReadProcessMemory`
---

I had once to debug a funny issue that took me ages to reproduce. As the title indicates,
it involved [`ReadProcessMemory`](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-readprocessmemory),
a Windows API that can read memory from another process.

The code was trying to read a string from another process from a known offset, although it did not know the size of the string.
To know how much to read, it would first call [`VirtualQueryEx`](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualqueryex)
to obtain how much memory was available at that offset.

So far, so good. However, one user reported what looked like a memory corruption around this area. No matter how much we tried, we could not
reproduce the issue... until I tried Windows Server 2016. The corruption *only* happened with that particular version of Windows.
Any other would be fine.

The bug happened because the code did not account for the offset passed to `VirtualQueryEx` being rounded to a page boundary.
Therefore, the call to `ReadProcessMemory` could ask for a bit more memory than was accessible.

For most Windows versions, this was OK. Like when reading a file, you would get a partial read, including the string null-terminator, as expected,
so all was good.

However, for some reason, in Windows 2016, if you ask for a bit more memory than you can read, you get back considerably *less* than what you can read.
For instance:

1. The program asks `VirtualQueryEx` how much memory it is readable at `0x0000f010`.
2. The size we get back (say, 16K) applies to `0x0000f000` (page boundary).
3. The program asks to read 16K at `0x0000f010`, so 10 bytes more than can be read at that address.
   1. In Windows 10, 11, or any other, that's ok, we get back (16K - 10) bytes.
   2. But in Windows Server 2016, we get 8K!

This means that only in Windows Server 2016 could we not get the null-terminator from the string back from the other process!
Hence the memory corruption.

Curiously, the truncation at 8K bytes was consistent, regardless of the particular offset.
