---
title: zotero
date: 2023-11-13
tags:
  - pkm
  - zotero
  - tools
  - math
categories: handbook
---

# Zotero

## Install and basic setting

## Plugins

**zotero 7:**

常用：

- Zotero attanger
- Del item with Attachment
- Ethereal Reference

好用：

- zotero better notes

没必要但可以：

- Ethereal Style
- Attachment Scanner
- arXiv Workflow for zotero
- Zoplicate
- Linter for zotero

## Sync

using ~~zotmoov~~ attanger to move files

zotero 7 to rename:

```
{{ title case="snake" }}{{ creators case="snake" prefix="_" max="1"}}{{ year prefix="_" }}
```

use any cloud drive to sync. I'm using koofr.

## zotero better notes

用来[导出笔记为markdown](/doc/dev/3in1wiki)

**templates:** you can copy these:

```quickinsert
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[QuickInsertV2]"
zoteroVersion: "7.0.9.SOURCE.fadbf3d2d"
pluginVersion: "2.0.18"
savedAt: "2024-11-07T07:59:45.835Z"
content: |-
  // @use-markdown
  <a href="${link}">${linkText}</a> [md](/wiki/math/zotero/${subNoteItem.getNoteTitle ? subNoteItem.getNoteTitle().replace(/[/\\?%*:|"<> ]/g, "-") + "-":""}${subNoteItem.key})

```

```quickimport
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[QuickImportV2]"
zoteroVersion: "7.0.9.SOURCE.fadbf3d2d"
pluginVersion: "2.0.18"
savedAt: "2024-11-07T05:54:37.696Z"
content: |-
  <blockquote>
  ${{
    return await Zotero.BetterNotes.api.convert.link2html(link, {noteItem, dryRun: _env.dryRun});
  }}$
  </blockquote>

```

```quicknote
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[QuickNoteV5]"
zoteroVersion: "7.0.9.SOURCE.fadbf3d2d"
pluginVersion: "2.0.18"
savedAt: "2024-11-07T05:54:55.255Z"
content: |-
  ${{
    let res = "";
    if (annotationItem.annotationComment) {
      res += await Zotero.BetterNotes.api.convert.md2html(
        annotationItem.annotationComment
      );
    } else {
  	res += "No comment";
    }
    return res;
  }}$

  // @use-markdown
  ***
  ${{
  	let res = "";
    res += await Zotero.BetterNotes.api.convert.annotations2html([annotationItem], {noteItem, ignoreComment: true});
    return res;
  }}$
```

```exportmdfilename
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[ExportMDFileNameV2]"
zoteroVersion: "7.0.9.SOURCE.fadbf3d2d"
pluginVersion: "2.0.18"
savedAt: "2024-11-07T05:55:24.890Z"
content: |-
  ${(noteItem.getNoteTitle ? noteItem.getNoteTitle().replace(/[/\\?%*:|"<> ]/g, "-") + "-" : "")}${noteItem.key}.md
```

```exportmdfileheader
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[ExportMDFileHeaderV2]"
zoteroVersion: "7.0.9.SOURCE.fadbf3d2d"
pluginVersion: "2.0.18"
savedAt: "2024-11-07T05:55:34.047Z"
content: |-
  ${{
    let header = {};
    header.tags = noteItem.getTags().map((_t) => _t.tag);
    header.parent = noteItem.parentItem
      ? noteItem.parentItem.getField("title")
      : "";
    header.collections = (
      await Zotero.Collections.getCollectionsContainingItems([
        (noteItem.parentItem || noteItem).id,
      ])
    ).map((c) => c.name);
    return JSON.stringify(header);
  }}$
```

```exportmdfilecontent
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[ExportMDFileContent]"
zoteroVersion: "7.0.9.SOURCE.fadbf3d2d"
pluginVersion: "2.0.18"
savedAt: "2024-11-07T05:56:22.434Z"
content: |-
  ${{
    let start = mdContent;
    let rmspan = start.replace(/<\/?span.*?>/g,'');
    let pdflink = rmspan.replace(/(<a .*?open.*?>)“(.*?)”/g,'$2 $1(pdf)</a>');
    let dir2zotero = pdflink.replace(/<a href.*?zhref="(.*?)" ztype.*?>/g,'<a href="$1">');
    return dir2zotero;
  }}$
```

annotation will be exported as

```
<span class="highlight" data-annotation="<data-annotation>" ztype="zhighlight"><a href="zotero://open/library/items/G4BKVA2X?page=2&#x26;annotation=LZLEYYRJ">“<content>”</a></span> <span class="citation" data-citation="<citation>" ztype="zcitation">(<span class="citation-item"><a href="zotero://select/library/items/GLXUZZJT"></a></span>)</span>
```

the color of annotation is coded as `%23<rgb>` in `<data-annotation>`, for
example blue (#2ea8e5) annotation is `%232ea8e5`
