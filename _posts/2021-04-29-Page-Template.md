---
title: this is title
description: this is description
search: false
categories:
  - pages
tags:
  - hello world
  - template
permalink: /posts/template/front-matter
date: 2020-01-01
last_modified_at: 2020-01-02
---


> this is front matter template with 'liquid'


-----

```{{ page.title }}```

{{ page.title }}

-----

```{{ page.description }}```

{{ page.description }}

-----

```{{ "now" | date: "%Y-%m-%d %H:%M" }}```

{{ "now" | date: "%Y-%m-%d %H:%M" }}

-----
