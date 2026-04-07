---
title: 数学笔记流程
date: 2026-04-01
summary:
tags:
categories:
topics:
series:
draft: true
status:
---

# 数学笔记流程

## zotero

QuickNoteV5

```clip
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[QuickNoteV5]"
zoteroVersion: "8.0.5.SOURCE.bdd3b1df5"
pluginVersion: "3.0.3"
savedAt: "2026-04-01T05:44:32.622Z"
content: |-
  ${{
    let res = "# ";
    if (annotationItem.annotationComment) {
  	res += annotationItem.annotationComment;
    } else {
  	  res += "Blank";
    }
    res = await Zotero.BetterNotes.api.convert.md2html(res);
    return res;
  }}$

  ${{
    let res = "";
    res += await Zotero.BetterNotes.api.convert.annotations2html([annotationItem], {noteItem, ignoreComment: true});
    return res;
  }}$
```

ExportMDFileContent

```clip
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[ExportMDFileContent]"
zoteroVersion: "8.0.5.SOURCE.bdd3b1df5"
pluginVersion: "3.0.3"
savedAt: "2026-04-01T05:44:06.253Z"
content: |-
  ${{
    let init = mdContent;
    let rmspan = init.replace(/<\/?span.*?>/g,'');
    let pagelink = rmspan.replace(/(<a .*?open.*?>)“(.*?)”/g,'$2 $1(page)</a>');
    let itemlink = pagelink.replace(/<a href.*?zhref="(.*?)" ztype.*?>/g,'<a href="$1">');
    return itemlink;
  }}$
```

QuickInsertV3

```clip
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[QuickInsertV3]"
zoteroVersion: "8.0.5.SOURCE.bdd3b1df5"
pluginVersion: "3.0.3"
savedAt: "2026-04-01T05:43:39.252Z"
content: |-
  // @use-markdown
  [${linkText}](/zote/${subNoteItem.getNoteTitle ? subNoteItem.getNoteTitle().replace(/[/\\?%*:|"<> ]/g, "-") + "-":""}${subNoteItem.key})
  <a href="${link}">Zotero</a>
```

ExportMDFileHeaderV2

```clip
# This template is specifically for importing/sharing, using better
# notes 'import from clipboard': copy the content and
# goto Zotero menu bar, click Tools->New Template from Clipboard.
# Do not copy-paste this to better notes template editor directly.
name: "[ExportMDFileHeaderV2]"
zoteroVersion: "8.0.5.SOURCE.bdd3b1df5"
pluginVersion: "3.0.3"
savedAt: "2026-04-01T06:05:15.800Z"
content: |-
  ${{
    let header = {};
    header.title = noteItem.getNoteTitle(noteItem);
    header.tags = noteItem.getTags().map((_t) => _t.tag);
    header.parent = noteItem.parentItem
      ? noteItem.parentItem.getField("title")
      : "";
    header.categories = (
      await Zotero.Collections.getCollectionsContainingItems([
        (noteItem.parentItem || noteItem).id,
      ])
    ).map((c) => c.name);
    return JSON.stringify(header);
  }}$
```

## obsidian

## site
