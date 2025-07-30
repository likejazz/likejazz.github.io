---
layout: wiki 
title: Front-End
tags:  ["Languages & Framework"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [NPM](#npm)
- [jQuery](#jquery)
    - [시작 함수](#시작-함수)
    - [event 제한](#event-제한)
- [CSS](#css)
    - [clear:both](#clearboth)

<!-- /TOC -->

# NPM
`npm`으로 dist 버전만 또는 minified 버전만 설치할 수 있을까.  
`npm` was primary created for backend dependencies. [^fn-npm] Bower has also been deprecated. [^fn-bower]

[^fn-npm]: <https://stackoverflow.com/a/25447302/3513266>
[^fn-bower]: <https://bower.io/blog/2017/how-to-migrate-away-from-bower/>

하지만 그나마 `bower`가 제일 깔끔하다.
```console
$ bower install jquery
```

# jQuery
## 시작 함수
jQuery의 시작 함수,
```
$(document).ready(function () {

});
```
간단하게 사용할 수 있는 형태를 함께 제공한다.
```
$(function () {

});
```
(모던 웹을 위한 JavaScript + jQuery 입문, 2011)

## event 제한
```
event.stopPropagation();
event.preventDefault();
```
는 간단히 `return false`로 대체 가능하다.

# CSS

새로운 레이아웃: `flex`
`column-width` 속성

`@media` 쿼리
pseudo_class
pseudo_elements

## clear:both
더 이상 의미 없는 div clear:both를 사용하지 말고 아래 clear fix 이용하라.
```
.clearfix:after {
	content: " ";
	display: block;
	clear: both;
}
```

Less 보다는 Sass를 주로 사용한다고. (전문가를 위한 CSS3, 2013)
