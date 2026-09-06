---
title: hugo
date: 2025-12-02
summary: 自制hugo主题后做一些记录
tags:
  - geek
categories: handbook
topics:
status: outline
ai: true
---

# hugo

hugo is a SSG. Using golang.

## hugo site structure

### kinds

There are mainly two kinds of site pages:

- `page` for all markdown files like `content/subfolder/something.md` and goes to `https://example.com/subfolder/somethinng`
- `section` for all `_index.md` in folder `content/another/_index.md` and goes to `https://example.com/another`

there are some special `section`, for example `content/_index` is `home`. And `taxonomy`, `term`, which are associated to taxonomy feature of hugo.

also you can make a bundle of a `page` with some resources. For example, `content/subfolder/something/index.md` goes to `https://example.com/subfolder/something`, and corresponding resources at `content/cubfolder/something/pic.jpg`

## hugo theme structure

```
custom-theme
├── assets
├── data
├── exampleSite
├── layouts
└── static
```

### layouts

```tree layouts
layouts
├── _markup
├── _partials
├── baseof.html
├── home.html
├── index.json
├── page.html
├── rss.xml
├── section.html
├── taxonomy.html
└── term.html
```

### render

```tree _markup
_markup
├── render-blockquote.html
├── render-codeblock-mermaid.html
├── render-codeblock.html
├── render-heading.html
├── render-image.html
└── render-link.html
```

### others

assets

i18n

data

## go templates

transforms data into HTML.

### context

Hugo makes extensive use of the Go Template language for designing its themes. "Context" refers to the data that is passed to a template when it is rendered. Templates can access context to display dynamic content.

**Example Context Access:**

```
<title>{{ .Title }}</title>
<p>{{ .Content }}</p>
```

### range, with, if, index

- **Range**: Used to iterate over collections such as a slice or map.
  **Example:**
  ```
  {{ range .Data.Pages }}
    <p>{{ .Title }}</p>
  {{ end }}
  ```
- **With**: Equivalent to an `if` statement that changes the default context.
  **Example:**
  ```
  {{ with .Site.Menu.main }}
    <ul>
      {{ range . }}
        <li>{{ .Name }}</li>
      {{ end }}
    </ul>
  {{ end }}
  ```
- **If**: Conditional rendering.
  **Example:**
  ```
  {{ if isset .Site.Params "author" }}
    <p>Author: {{ .Site.Params.author }}</p>
  {{ end }}
  ```
- **Index**: Retrieves specific content from a slice or map.
  **Example:**
  ```
  {{ index .Site.Params "social" "twitter" }}
  ```

Sorting, grouping, and accessing the first elements of collections can refine the data display.

### partials

```go
{{ partial "header.html" . }}
```

where `layouts/partials/header.html` is the partial template file.

### hugo special

#### slice, scratch

Hugo provides a `slice` function that helps in working with lists and arrays, enabling you to append items or create slices dynamically. The `scratch` object in Hugo allows for temporary data storage and manipulation during template rendering through methods like `add`, `set`, and `get`.

**Example of Using Slice:**

```
{{ $list := slice "apple" "banana" "cherry" }}
{{ range $list }}
  <p>{{ . }}</p>
{{ end }}
```

**Example of Using Scratch:**

```
{{ .Scratch.Set "count" 0 }}
{{ range .Pages }}
    {{ .Scratch.Add "count" 1 }}
{{ end }}
<p>Total Pages: {{ .Scratch.Get "count" }}</p>
```

## hugo feature

### taxonomies

### menus

### outputs

### assets

The `assets` folder in Hugo is a central place for managing your static assets, such as CSS, JavaScript, and images. You can preprocess your assets using Hugo Pipes, enabling features like minification, fingerprinting, and transpiling. This allows you to optimize your site's performance and maintain organized asset management.

**Example of Minification Using Hugo Pipes:**

```
{{ $styles := resources.Get "css/styles.css" | minify }}
<link rel="stylesheet" href="{{ $styles.Permalink }}">
```

**Example of Fingerprinting for Cache Busting:**

```
{{ $js := resources.Get "js/script.js" | fingerprint }}
<script src="{{ $js.Permalink }}"></script>
```

### misc

i18n

## trick

### backlinks

### local partials

### misc

related

prev next
