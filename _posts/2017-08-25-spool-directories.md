---
title: Spool directories
---

FTS3 uses a message queue based on directories. This is, a spool dir, basically.

Until version 3.5 or so, it used to write all messages into the same directory,
and this have one issue: when the consumer stops working for a long while, the
number of files written can explode.

The issue is not as immediate as it may seem, though. The problem is not just
having a big directory taking a lot of disk space (the directory itself), but
the fact that even after consuming all pending messages, the directory size
will *not* decrease.

It is a bit counter intuitive, but in Linux, at least with EXT filesystems, you
have a watermark effect on the size of a directory: the OS never reclaims back
the space used by a directory, even if it just a lot of empty entries.

This causes problems on the long run, of course, since long after the issue was
solved, the directory will remain occupying all that much space.

Now, FTS3 uses "DirQ", an implementation from the messaging team that uses
subdirectories, which are gone when empty, reducing considerably the problem.

P.S `fsck.ext* -D` can solve the issue without deleting the directory. 
