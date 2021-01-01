---
layout: wiki 
title: Recommender System
last-modified: 2021/01/02 04:04:56
---

<!-- TOC -->

- [Books](#books)
    - [Python을 이용한 개인화 추천시스템 <sup>2020</sup>](#python을-이용한-개인화-추천시스템-2020)

<!-- /TOC -->

# Books
## Python을 이용한 개인화 추천시스템 <sup>2020</sup>
- CBF는 소개만 할 뿐 이 책에서는 언급하지 않는다.
- p19. MovieLens에서 전체 평균을 제시하면 RMSE 0.99가 나온다.
- train/test 구분으로 이후에는 성능이 떨어지는데, gender 집단 평균은 차이가 없고 occupation 집단 평균을 제시하면 1.12로(train/test로 인해 원래 이보다 더 높음) RMSE가 낮아진다. 이후 모든 결과 또한 당연히 train/test 구분 결과.

CF
- 전체 UBCF 기본은 1.017, ratings만으로 처리한다.
- KNN=30일때 1.011
- 평가경향 반영 0.9417
- IBCF는 1.017

MF
- 사용자와 아이템의 특성을 K개의 잠재요인 <sup>Latent Variable</sup>을 사용해 분석하는 모델. 『기계는 어떻게 생각하는가』를 보면 MF는 특징에 가중치를 곱한 결과의 합으로 최종 값을 표현한다고 했다. 여기서도, K개 특징을 추출해 연관 관계를 -1.0 ~ 1.0 으로 표현한 예제를 보여준다. 사용자요인 P와 아이템요인 Q를 선정하는데, 예제에서는 영화의 특성과 사용자의 특성을 각각 2개(K=2)의 잠재요인으로 분해하고 이 잠재요인 계산 값으로 점수를 예측한다. p61
- SGD로 학습 100번 iter로 0.88이 나오긴 하지만 여기서도 train/test를 나누지 않았다.
    - `surprise.KNNWithMeans` 결과는 0.9542. KNN 인원 제한과 평가경향 반영을 한 결과와 동일하다.
    - `surprise.SVD`는 `GridSearchCV`로 0.9117

Deep Learning
- Keras로 구현하면 RMSE 1.08. SGD 파라미터나 epoch, batch size 조건 등으로 인해 최적화가 안되어 있기 때문이라고 설명. MF는 은닉층 없는 신경망으로 볼 수 있다. p104
- DNN으로 구현하면 0.95 까지 떨어진다. 딥러닝 모델은 다양한 변수를 추가할 수 있으므로 직업을 추가했다. 그러나 거의 성능 향상이 없다 p124

하이브리드
- p133. 복수의 알고리즘이 결합되는 경우에, 한 알고리즘은 다른 알고리즘의 오류를 보정하는 역할을 하는 경우가 많다.
- p137. MF(88%)와 CF(12%)를 결합하여 0.9088 까지 영끌함.

Sparse Matrix
- 행렬이 매우 클때는 `from scipy.sparse import csr_matrix` 활용한다. MovieLens 20M 데이터를 sparse matrix로 계산하면 MF로 0.78까지 향상시킬 수 있다.