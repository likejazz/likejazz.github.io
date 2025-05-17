---
layout: wiki 
title: Distance
tags: ["Machine Learning"]
last_modified_at: 2025/05/17 19:55:07
---

<!-- TOC -->

- [거리<sup>distance measures</sup>](#거리distance-measures)
    - [유클리드 공간 거리에 따른 점수](#유클리드-공간-거리에-따른-점수)

<!-- /TOC -->

# 거리<sup>distance measures</sup>
- 유클리드 거리 L2-Norm 제곱  
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/45aec22918ea0fc3c1821cccee33c34ee3b3352f" />  

- 맨하탄 거리 L1-Norm 절대값  
<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/6909908a18e848414a32a6310c5c7fed3f18e7b6" />  

도시의 구획이 직각으로 나뉘어 있을 때 이 도시의 두 지점 사이의 거리를 측정하는 것과 같다. (핸즈온 머신러닝, 2017)
- 자카드 거리
    - 자카드 유사도<sup>jaccard index, jaccard similarity coefficient</sup>: 
    <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/eaef5aa86949f49e7dc6b9c8c3dd8b233332c9e7" />  

    - 자카드 거리<sup>jaccard distance</sup>: `1 - 자카드 유사도` 
    <img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/3d17a48a5fb6cea57b076200de6edbccbc1c38f9" />
    
- 코사인 거리 [코사인 유사도의 의미](http://likejazz.com/cosine-sim/)  
코사인 유사도 = 1 - 코사인 거리
- 편집 거리<sup>edit distance, levenshtein distance</sup>
    - LD is measure of the similarity between two strings. The distance is the **number of deletions, insertions, or substitutions** required to transform s into t.
    - 편집 거리<sup>edit distance</sup>를 정의하고 추정하는 또 다른 방법은 x와 y의 최장 공통 부분열<sup>LCS, longest common subsequence</sup>을 계산하는 것이다. x와 y의 LCS는 x와 y에 공통으로 포함되지 않는 문자를 삭제해가면서 만들어지는 문자열로서, 이 방식으로 구성될 수 있는 문자열 중 가장 긴 문자열이다. 편집 거리 d(x,y)는 x의 길이와 y의 길이의 합에서 그들의 LCS 길이의 두 배를 뺀 값으로 계산된다. (빅데이터 마이닝, 2011)
    
## 유클리드 공간 거리에 따른 점수
- 데이터: 700차원 $$\mathbb{R}^{700}$$ Sent2Vec
- [Distance in Euclidean space](https://en.wikipedia.org/wiki/Distance#Distance_in_Euclidean_space)
- scipy [Compute distance](https://docs.scipy.org/doc/scipy/reference/generated/scipy.spatial.distance.cdist.html)

| 거리 | TOP 1 | TOP 3 | TOP 5 |
| --- | ----- | ----- | ----- |
| cosine | **4099(0.8316)** | 4583(0.9298) | **4702(0.9539)** |
| correlation | 4094(0.8306) | **4586(0.9304)** | 4702(0.9539) |
| minkowski | 3866(0.7842) | 4321(0.8765) | 4443(0.9012) |
| cityblock | 3851(0.7811) | 4311(0.8744) | 4447(0.902) |
| seuclidean | 3883(0.7876) | 4334(0.8791) | 4461(0.9049) |
| sqeuclidean | 3866(0.7842) | 4321(0.8765) | 4443(0.9012) |
| hamming | - | | |
| jaccard | - | | |
| chebyshev | 3363(0.6822) | 3883(0.7876) | 4045(0.8205) |
| canberra | 3934(0.798) | 4492(0.9112) | 4613(0.9357) |
| braycurtis | 3986(0.8085) | 4532(0.9193) | 4669(0.9471) |
| mahalanobis | - | | |
| yule | - | | |
| dice | - | | |
| kulsinski | - | | |
| rogerstanimoto | - | | |
| russellrao | - | | |
| sokalmichener | - | | |
| sokalsneath | - | | |
| wminkowski | - | | |
