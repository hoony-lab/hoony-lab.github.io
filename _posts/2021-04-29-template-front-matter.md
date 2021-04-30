---
title: this is title
description: this is description
search: false
categories:
  - pages
tags:
  - hello world
  - template
permalink: /posts/template-front-matter
date: 2020-01-01
last_modified_at: 2020-01-02

foo: '{'
bar: '}'
some_variable: haha
---


> this is front matter template and 'liquid' examples


-----

```{{page.foo}}{{page.foo}} page.title }}```

{{ page.title }}

-----

```{{page.foo}}{{page.foo}} page.description }}```

{{ page.description }}

-----

```{{page.foo}}{{page.foo}} "now" | date: "%Y-%m-%d %H:%M" }}```

{{ "now" | date: "%Y-%m-%d %H:%M" }}

-----

```{{page.foo}}{{page.foo}} page.some_variable }}```

{{ page.some_variable }}

-----
