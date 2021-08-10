---
title: "Cannot create window: no screens available"
---

A user had a problem running a Qt5 application on a Mac M1 laptop, installed
via [conda](https://conda.io).

Only an empty dialog would show up.

We have a Mac mini with that CPU, accessible via ssh.

I tried to reproduce using X11 forwarding, but I could not get the
application to work at all. I was getting this error

```
PasteBoard: Error creating pasteboard: com.apple.pasteboard.clipboard [-4960]
PasteBoard: Error creating pasteboard: com.apple.pasteboard.find [-4960]
no screens available, assuming 24-bit color
Cannot create window: no screens available
```

My Google/DDG abilities prove insufficient to find any useful hints.

Finally I figured it out, so I put it here for future reference, and
in case someone else has the same problem:

The plugin `libqxcb` is missing on the build for macos, and cocoa can not
be used with forwarding.
