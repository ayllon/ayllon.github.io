---
title: But it doesn't work on my machine!
---

So we had a weird bug on gfal2. Trying to debug some crashes, when I run
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
for i in `ldd /usr/lib64/gfal2-plugins/libgfal_plugin_srm.so | sed -nr "s/.* => ([^[:space:]]+)(.*)/\1/p"`; do
  rpm -qf $i;
done | sort | uniq
```

So find the differences... I tend to just open two tabs on a text editor,
and switch fast. Makes easy to identify the difference without going through the
whole list.

And there it was, the difference, the single RPM with a different version:
`globus-gssapi-gsi`.

So granted, once I downgraded that RPM on development, the command would work as
expected. I upgraded again, and it would break.

Now to identify the change that broke this.

## Update
Doing basically a hand-made bisect, I found out that the commit that "broke" this
client/server interaction was [34813cc29eaa519482626a3c3576f5f7708653a6](https://github.com/globus/globus-toolkit/commit/34813cc29eaa519482626a3c3576f5f7708653a6).

Specifically, at `globus_i_gsi_gss_utils.c:474` it seems like something called
"empty fragments" were re-enabled with this commit (while removing SSlv3 support).

"Empty fragment" and a spurious `0` looks suspicious.

And indeed, when I built a patched version, disabling empty fragments,
everything would work just fine again.

These empty fragments happen to be a countermeasure against [a vulnerability of CBC
ciphersuites](https://www.openssl.org/~bodo/tls-cbc.txt), but Bestman has
precisely one of this "buggy SSL implementations" that do not play well with them.
Thus, the empty fragment ends being seen a "0" by the application.
