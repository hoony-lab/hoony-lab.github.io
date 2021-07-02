---
title: [minimal-mistakes] TOC (table-of-contents)
description:
search: true
categories:
  - minimal-mistakes
tags:
  - minimal-mistakes
permalink:

last_modified_at:
---

> 현재 페이지의 목차를 나타내주는 TOC (table-of-contents)를 만들어보자

**[minimal-mistakes docs](https://mmistakes.github.io/minimal-mistakes/docs/layouts/#table-of-contents)** 를 확인해보면 TOC를 손쉽게 생성할 수 있다. 그저 `Front Matter`에 `toc: true` 만 추가하면 된다.

TOC를 만들고, TOC를 sticky 하게 할 것인지 결정할 것이다.

## 각 Front Matter에 값 추가하기

`toc: true`를 추가하고 `commit & push & reload` 하고 확인해보자.

```yaml
---
title: 이것은 제목
...

toc: true
---
```


## 스크롤해도 따라다니게 해줘 !

`toc: true` 를 추가하고 `toc_sticky: true` 를 추가로 넣어주면 TOC가 둥둥 떠다녀서 스크롤해도 우측에 떠있게 된다. 참고로 `toc_sticky`의 default 값은 `false` 라고 한다.

`toc_sticky: true`를 추가하고 `commit & push & reload` 하고 확인해보자.

```yaml
---
title: 이것은 제목
...

toc: true
toc_sticky: true
---
```

## 매 페이지마다 추가해야해 ?

`_config.yaml` 에 우리는 매 포스팅의 설정값을 위한 `defaults` 를 설정했을 것이다. 매 포스팅 마다 `Front Matter`에 추가하지 말고, `_config.yaml` 하단 부분에 `toc: true` 와 `toc_sticky: true`를 추가해준다.

```yaml
defaults:
  - scope:
      path: ""
      type: posts
    values:
      toc: true
      toc_sticky: true
      ...
```


## 마무으리

~~솔직히 한번에 끝내는게 좋지~~
