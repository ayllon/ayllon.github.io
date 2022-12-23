---
title:  No valid patches in input 
---

I am a Windows noob. I was doing some development in Windows, and I wanted to get
a patch file.

I did what I usually do

```cmd
git show <commit> > .../file.patch
```

Add the patch file to the `rpm` build, run a test, and...

```
error: No valid patches in input (allow with "--allow-empty")
```

It turns out that [Out-File](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_character_encoding),
which handles the redirection, creates files with a default UTF-16LE encoding.

The patch is not empty; it "just" has an unsupported encoding.
