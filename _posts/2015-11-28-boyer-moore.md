---
layout: post
title: ! '문자열 검색: Boyer-Moore 알고리즘'
tags: ["NLP"]
last_modified_at: 2025/05/17 19:55:07
---

<div class="message">
보이어-무어 알고리즘을 이용해 문자열 검색 기능을 만들어 본다.
</div>

<small>
*2015년 11월 27일 1차 개정*  
*2014년 7월 1일 초안 작성*  
</small>

<!-- TOC -->

- [서론](#서론)
- [본론](#본론)
- [결론](#결론)
- [참고](#참고)

<!-- /TOC -->

## 서론
[검색엔진을 만든](http://likejazz.com/search-engine/) 직후였다. 한참동안 검색 기능을 테스트 하다 우연히 데이타 파일을 직접 grep 해봤다. 그런데, 내가 만든 엔진보다 grep이 훨씬 더 빨랐다. 물론 grep은 단순히 검색된 문자열을 순서대로 나열하는 것이고 엔진은 가중치가 높은 순으로 랭킹을 거친다는 차이가 있지만 그래도 검색 엔진이 모든 문자열을 일일이 검색하는 것보다 훨씬 더 느리다는 점은 다소 굴욕적이다.

GNU grep을 만든 Mike Haertel이 [why GNU grep is fast](http://lists.freebsd.org/pipermail/freebsd-current/2010-August/019310.html)에서 밝힌 바에 따르면 GNU grep은 보이어-무어(Boyer-Moore) 알고리즘을 사용한다고 한다. 물론 이외에도 여러가지 hacks를 적용해 성능을 극대화했지만, 우선 근간이 되는 보이어-무어 알고리즘부터 구현해보기로 한다.

## 본론
[보이어-무어(Boyer-Moore) 알고리즘](http://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_string_search_algorithm)은 크게 세 가지 rule로 구성된다.

<img src="https://31.media.tumblr.com/42306da7fc5626c1afdd273ca8dc5647/tumblr_inline_n7z7xlN6L51qzgoac.png" width="350" />

1. [right to left scan](http://java.dzone.com/articles/algorithm-week-boyer-moore)
2. bad character rule
3. good suffix rule

후미에서부터 역순으로 문자열 비교를 수행하고 mismatch가 발생할때 bad charater rule 또는 good suffix rule 중 max 값에 대한 shift를 수행한다. rule은 단순하지만 막상 암산으로 계산하려면 쉽지 않다. 그래서 연습장에 적어나가며 알고리즘을 계산했다.

![image](https://33.media.tumblr.com/2003356869d9d8e53f07525cc42d3616/tumblr_inline_n7z77oZld31qzgoac.jpg)

사실 알고리즘이 전부인 코드라 이번에는 코딩보다는 알고리즘을 이해하는데 대부분의 시간을 할애했다. 워낙에 유명한 알고리즘이다 보니 예제 코드도 포럼, GitHub 뿐만 아니라 gist에 여러가지가 있는데 대부분은 C&P 한 것에 불과하고(함수명이 모두 동일) 그 중 일부는 잘못된 코드도 더러 있었다.

*   1977년 출판된 원본 논문 PDF: [A Fast String Searching Algorithm](http://www.akira.ruc.dk/~keld/teaching/algoritmedesign_f05/Artikler/09/Boyer77.pdf)

그나마 위키피디어에 있는 샘플이 논문에 가장 가까웠고, 실제로 위키피디어 C 코드는 변수명을 `string`, `stringlen`, `pat`, `patlen` 그리고 rule 테이블을 `delta1`, `delta2`로 명시하는등 논문과 명칭이 동일하고 논문에 명시된 알고리즘과 동일한 순서대로 작성되어 있다.

원래 논문은 PDP-10 어셈블리로 구현되었으며(1977년에 쓰여진) 위키피디어에 C로 작성한 코드외에는 Python과 Java 버전은 엉뚱하게도 논문과 다른 순서로 작성되어 있다. 애당초 이번에는 성능에 포커싱해 C 구현외에 다른 언어는 생각 해보지도 않았기에 망설임없이 C 코드를 fork해서 구현했다.

## 결론
그렇게 해서 아래와 같은 결과를 얻었다.

![](https://33.media.tumblr.com/cd17538fecd89306a93bf0b295708f73/tumblr_inline_n7z82875Wy1qzgoac.png)

애당초 고성능이 가장 큰 목표이기에 shift 횟수와, match 판단을 위해 몇 개의 문자열<sup>chars</sup>을 비교했는지<sup>compared</sup>를 디버그로 출력하도록 했다. `#define DEBUG`를 하면 디버그가 출력되며 기본적으로 `define`한 상태로 커밋했다.

chars compared 값이 적을수록 더 최적화된 연산을 수행했다고 보면 된다. 물론 여기에는 원래 계산해야 하는 preprocessing 즉, delta table 계산 시간은 빠져있다. 이 점은 각 예문별 연산 확인시 참고하기 바란다.

## 참고
이번에도 주말을 활용했는데 마침 바닷가에 캠핑을 가기로 한 날이라 노트북을 들고가서 좀 더 수정해야 할지 한참을 고민했다.

![](https://33.media.tumblr.com/d5f16684ca98441fea4b105e4d55646b/tumblr_inline_n7z8fgUBuS1qzgoac.jpg)

노트북을 가져갔고 바닷가에선 열심히 수영하면서 놀아주고 애들 잘때 밤에, 낮에 애들 놀때 틈틈히 코딩했다. 다행히 만족스런 결과물을 얻을 수 있었다.

사실 문자열 검색은 이미 좋은 라이브러리가 많이 나와있기 때문에 실제로 구현할 일이 거의 없다. 어쩌면 이번이 마지막이 될지도 모르는 일인데 깊이 있게 짚어보고 넘어 갈 수 있는 좋은 기회였다.

- [boyer-moore-string-search - GitHub](https://github.com/likejazz/boyer-moore-string-search)
