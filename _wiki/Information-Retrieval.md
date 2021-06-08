---
layout: wiki 
title: Information Retrieval
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [강의](#강의)
- [PageRank](#pagerank)
- [BM25](#bm25)

<!-- /TOC -->

# 강의
- [스탠포드 CS276](https://web.stanford.edu/class/cs276/) IR 강의 Syllabus by Chris Manning, ~~Dan Jurafsky~~ Pandu Nayak

# PageRank
- [PageRank](https://sungmooncho.com/2012/08/26/pagerank/)
- HITS: 초기에 아카데믹으로만 접근하고 사업화 하지 않아 페이지랭크에 비해 유명하지 않음. 또한 미리 계산하는 페이지랭크와 달리 검색시 계산해야 하는 부담이 있음

# BM25
수식을 그래프로 [표현한 문서](http://opensourceconnections.com/blog/2015/10/16/bm25-the-next-generation-of-lucene-relevation/) 단순 수식 나열보다 시각화로 보기 편함  
<img src="http://opensourceconnections.com/blog/uploads/2015/IDF1.png" width="50%" />

<img src="http://opensourceconnections.com/blog/uploads/2015/TF1.png" width="50%" />

<img src="http://opensourceconnections.com/blog/uploads/2015/NORMS1.png" width="50%" />

BM25는 문서 길이 정규화<sup>document length normalization</sup> 수식을 포함

기존에 블로그에 정리했던 글
- [BM25 overview](http://dev.likejazz.com/post/120399548878/bm25-overview)
- [BM25](http://dev.likejazz.com/post/40647975988/bm25)
