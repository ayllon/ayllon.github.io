---
title: But it doesn't work on my machine!
---

So we had a weird bug on gfal2. Trying to debug some crash, when I run
gfal-ls towards the endpoint, I would get an error like this:

```
[CGSI-GSOAP] HTTP/1.1 501 Method 0POST is not defined in RFC 2068 and is not supported by the Servlet API
```

At first I thought it was a bug on the remote storage side, but no one else
could reproduce. Even I couldn't if I went to a production machine at CERN.

Ok, it seems like the development version of gfal2 could be buggy, right?
Let's checkout master, rebuild, rerun... and there is the bug again.

No reason to panic. Let's go to a development machine, and downgrade the rpms
to have the production ones.

Still there. But not on an actual production machine.

Doesn't seem to be in gfal2 then. Some external dependency broke something?
Which one?

Since this was affecting the SRM protocol, I run this on both the production
machine, and the development one:

```bash
for i in `ldd /usr/lib64/gfal2-plugins/libgfal_plugin_srm.so | cut -d'>' -f2 | cut -d' ' -f 2 | grep -v "(" |grep -v "^$" `; do
  rpm -qf $i;
done | sort | uniq
```

So find the difference... I tend to just open two tabs on a text editor,
and switch fast. Makes easy to identify the difference without going through the
whole list.

And there it was, the difference, the single RPM with a different version:
`globus-gssapi-gsi`.

So granted, once I downgraded that RPM on development, the command would work as
expected. I upgraded again, and it would break.

Now to identify the change that broke this.
