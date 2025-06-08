---
layout: wiki 
title: Information Retrieval
tags: ["Information Retrieval"]
last_modified_at: 2025/06/08 19:29:39
---

<!-- TOC -->

- [검색엔진](#검색엔진)
  - [검색을 위한 딥러닝](#검색을-위한-딥러닝)
- [강의](#강의)
- [PageRank](#pagerank)
- [BM25](#bm25)

<!-- /TOC -->

# 검색엔진
Vertical Search Engine: 온라인에 있는 전체 내용이 아니라 특정 문서 형식, 특정 주제에 한정한 검색. 더 자세한 결과를 더 정확하게 검색할 수 있다. e.g. 논문 검색시 구글 스칼라 검색

- Vector Space Model, VSM: 문서, 쿼리 벡터 간의 유사도 판단. TF-IDF도 이에 해당.

## 검색을 위한 딥러닝
- 동의어<sup>synonym</sup> 확장 소개 (p46, 검색을 위한 딥러닝) DNN에 넣어 유사한 단어 추출. word2vec의 knn 결과로 동의어 추출. automatic query expansion: 동의어 확장은 쿼리 시간에 사용하는 경우 자동 쿼리 확장의 특수한 사례로 볼 수 있다. (p92) 결국 신경망은 QA에서 주로 사용. Seq2Seq로 쿼리 확장하는 방법까지 소개.
- Query Suggestion에 딥러닝 적용: Lucene의 lookup API 예를 드는데, 사용해보진 않았지만 쿼리 자동완성 기능을 제공하는 것으로 보인다. 제목은 suggestion이지만 실제로는 autocomplete에 대한 내용을 다루고 있다. (p141)

# 강의
[CS276](https://web.stanford.edu/class/cs276/) IR and Web Search by Chris Manning, ~~Dan Jurafsky~~ Pandu Nayak

# PageRank
HITS: 초기에 아카데믹으로만 접근하고 사업화 하지 않아 페이지랭크에 비해 유명하지 않음. 또한 미리 계산하는 페이지랭크와 달리 검색시 계산해야 하는 부담이 있다.

# BM25
수식을 그래프로 표현[^fn-bm25]해서 설명. 루씬 BM25 도입 이후 소개.

[^fn-bm25]: <https://opensourceconnections.com/blog/2015/10/16/bm25-the-next-generation-of-lucene-relevation/>

<img src="https://user-images.githubusercontent.com/1250095/148515353-beb9e13c-2b9f-4adb-88fe-96b4a6a724b8.png" width="45%" style="float: left; margin-right: 5px">

<img src="https://user-images.githubusercontent.com/1250095/148515357-5b6ba0bf-6c9e-4784-8166-60a6ebb06b00.png" width="45%">

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==" style="clear: both">

<img src="https://user-images.githubusercontent.com/1250095/148515360-f06c9aa1-2de6-4fc2-b8b0-a112e17687c1.png" width="50%" />

BM25는 문서 길이 정규화<sup>document length normalization</sup> 수식을 포함하기 때문. `b`는 document length normalize degree를 제어한다. 낮을수록 문서 길이에 영향을 받지 않고 score 일정(사실상 tfidf와 동일)
