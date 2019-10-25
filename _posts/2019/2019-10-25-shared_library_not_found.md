---
title: libQt5Core.so.5 not found
---

There is one tool we ship to our users inside a Docker container, mainly
because we, the developers, are Linux users, and they are MacOSX users.

So, we build the rpms, and install them inside a Docker image based on Fedora,
and upload it to Docker Hub.

However, one of the users come back saying they get an error, something
along the lines

```
error while loading shared libraries: libQt5Core.so.5: cannot open shared object file: No such file or directory
```

I tried to reproduce locally, with exactly the same Docker image, but it
would work.

We tried to make sure Qt5 was properly installed, and, indeed,
`/usr/lib64/libQt5Core.so.5` is present.

Let's try ldd:

```bash
ldd /usr/bin/<app> | grep -i qt
libPhzQtUI.so => /usr/bin/../lib64/libPhzQtUI.so (0x00007f06385ad000)
libQt5Widgets.so.5 => /usr/bin/../lib64/libQt5Widgets.so.5 (0x00007f0637f3a000)
libQt5Core.so.5 => not found
libQt5Xml.so.5 => /usr/bin/../lib64/../lib64/libQt5Xml.so.5 (0x00007f063787a000)
libQt5Gui.so.5 => /usr/bin/../lib64/../lib64/libQt5Gui.so.5 (0x00007f0636ff4000)
```

The file is there, but ldd complains about it... 🤔

After a bit of Googling, this did the magic:

```bash
objdump -s -j .note.ABI-tag /usr/lib64/libQt5Core.so.5

/usr/lib64/libQt5Core.so.5:     file format elf64-x86-64

Contents of section .note.ABI-tag:
 4cdf58 04000000 10000000 01000000 474e5500  ............GNU.
 4cdf68 00000000 04000000 0b000000 00000000  ................
```

So, as it turns out, those last three bytes are saying that `libQt5Core.so`
requires a kernel 4.11 or higher. Docker for MacOSX ships a kernel 4.9.

And that's why the file is there, but it refuses to load, while it runs
just fine "in my machine".

Docker does not isolate as much as one would think...

P.S Interestingly, while for Fedora 29 it asks for a kernel 4.11,
in Fedora 30 only 3.17 is required.
