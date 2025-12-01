---
title: 豆瓣记录导入obsidian
date: 2025-05-28
tags:
    - obsidian
    - pkm
    - geek
categories: handbook
---

# obsidian 和豆瓣

## plugins

发现一些插件可以把豆瓣的信息导入到obsidian，可以作为阅读管理统计。

首先是豆瓣的[插件](https://github.com/Wanxp/obsidian-douban)，可以登陆豆瓣账户，然后可以同步自己的书影音。

[weread](https://github.com/zhaohongxuan/obsidian-weread-plugin/)导出划线，这个和豆瓣的不能是同一个文件，会有同步的问题。可以看完了再把划线移动到其他文档。

用bookshelf来查看，和记录阅读进度。这个似乎可以用base之类的方法代替。

## 模板配置

### 豆瓣

book

```temple
---
title: {{title}}
lists: {{type}}
score: {{score}}
rating: {{myRating}}
originalTitle: {{originalTitle}}
published: {{datePublished}}
pages: {{totalPage}}
author: {{author}}
tags: {{myTags}}
state: {{myState}}
url: {{url}}
country: {{country}}
language: {{language}}
time: {{time}}
comment: {{desc}}
cover:
---

# {{title}}

![image]({{image}})

comment: {{desc}}

{{myComment}}

doubanId: {{id}}

## log

```

movie

```template
---
doubanId: {{id}}
title: {{title}}
lists: {{type}}
score: {{score}}
rating: {{myRating}}
originalTitle: {{originalTitle}}
published: {{datePublished}}
director: {{director}}
actor: {{actor}}
author: {{author}}
tags: {{myTags}}
url: {{url}}
aliases: {{aliases}}
country: {{country}}
language: {{language}}
time: {{time}}
comment: {{desc}}
cover:
---

# {{title}}
![]({{image}})
IMDb: {{IMDb}}
```

### weread

这个是只导出划线和高亮没有评论，因为我不怎么在微信阅读写评论。

```template
---
title: {{ metaData.title }}
lists: book
author: {{ metaData.author }}
time: {{ metaData.publishTime }}
comment: {{ metaData.intro }}
cover: {{ metaDate.cover }}
---

![{{metaData.title}}|200]({{metaData.cover}})

作者： {{metaData.author}}
简介： {{metaData.intro}}
类型： {{metaData.category}}

## highlights

{% for chapter in chapterHighlights %}### {{chapter.chapterTitle}}

{% for highlight in chapter.highlights %}- {{ highlight.markText |trim }}
{% endfor %}
{% endfor %}
```

### bookshelf

这个json可以直接放在`obsidian-repo/.obsidian/plugins/bookshelf/data.json`中。

```json
{
    "settingsVersion": 20250408,
    "booksFolder": "gallery",
    "bookProperties": {
        "cover": "cover",
        "author": "author",
        "published": "published",
        "pages": "pages",
        "tags": "tags",
        "rating": "rating",
        "lists": "lists",
        "comment": "comment",
        "links": "links",
        "series": "series",
        "positionInSeries": "position-in-series",
        "duration": "duration"
    },
    "bookNote": {
        "enabled": true,
        "heading": "log",
        "dateFormat": "yyyy-MM-dd",
        "patterns": {
            "started": "{date}: Started",
            "finished": "{date}: Finished",
            "abandoned": "{date}: Abandoned",
            "absoluteProgress": "{date}: {start}-{end}",
            "relativeProgress": "{date}: {end}"
        }
    },
    "dailyNote": {
        "enabled": true,
        "heading": "Reading",
        "patterns": {
            "started": "Started {{book}}",
            "finished": "Finished {book}",
            "abandoned": "Abandoned {book}",
            "absoluteProgress": "Read {book}: {start}-{end}",
            "relativeProgress": "Read {book}: {end}"
        }
    },
    "readingProgress": {
        "newEntryLocation": "bookNote"
    },
    "previousVersion": "0.18.0",
    "showReleaseNotes": true
}
```

## 解释

我的文件放在`obsidian-repo/gallery/`目录下，用bookshelf的lists来区分书、电影和音乐
