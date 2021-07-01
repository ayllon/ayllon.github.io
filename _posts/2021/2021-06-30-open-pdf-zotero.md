---
title: Keeping Notes (Or Open PDFs via Zotero)
---

Following a recommendation from a colleague, I have been trying
[Obsidian](https://obsidian.md/) to keep notes, link them, etc.

I installed its [citation plugin](https://github.com/hans/obsidian-citation-plugin)
to write literature notes and the like from the bibliography I keep
in [Zotero](https://www.zotero.org/).

Pretty cool, I have to say. Personally, I like Obsidian keeps the content
in plain text files (markdown), so you are not irreversibly bound to
a product.

For the extra mile, I also installed [Zotfile](http://zotfile.com/) to extract
into Zotero the notes I add to the PDFs, and [Mdnotes](https://github.com/argenos/zotero-mdnotes)
to convert those notes to markdown, so I can copy them into the notes
in Obsidian.

Definitely do not handle confidential stuff this way, too many plugins involved.
But for my purposes, I do not worry.

Anyway, the missing bit is that the markdown notes will include links
that look like `zotero://open-pdf/library/items/ID?page=p`, so from the
notes you can open directly on your PDF viewer the page where the note is.
But those links may not work out of the box, at least not in Fedora.

In order to fix this, first we need a `.desktop` file for Zotero, specifying
the MimeType `x-scheme-handler/zotero`, which tells that `zotero://` URLs
are to be open with Zotero.

```bash
cd ~/.local/share/applications
cat zotero.desktop
```
```ini
[Desktop Entry]
Comment=
Terminal=false
Name=Zotero
Exec=/home/aalvarez/Tools/Zotero_linux-x86_64/zotero -url %u
Type=Application
Icon=/home/aalvarez/Tools/Zotero_linux-x86_64/chrome/icons/default/default256.png
MimeType=x-scheme-handler/zotero;
```

Note the [annoyingly undocumented `-url` parameter](https://forums.zotero.org/discussion/78345/zotero-executable-doesnt-handle-url-parameter-as-documented).

Last, we need to register the MimeType

```bash
xdg-mime default zotero.desktop x-scheme-handler/zotero
```

And, with that, we can open the `zotero` URLs as folllows:

```bash
xdg-open "zotero://open-pdf/library/items/ZQRC5HT9?page=5"
```

## Still doesn't work from Obsidian?

May need to refresh `mimeinfo.cache`

```bash
cd $HOME/.local/share/applications
update-desktop-database .
```
