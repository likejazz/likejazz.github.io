---
layout: wiki 
title: Scikit Learn
tags: ["Machine Learning"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [개요](#개요)
- [팩키지](#팩키지)
- [클래스](#클래스)
- [기타](#기타)
    - [상관 계수<sup>correlation coefficient</sup>](#상관-계수correlation-coefficient)

<!-- /TOC -->

# 개요
- [scikit-learn의 분류기 비교](http://scikit-learn.org/stable/auto_examples/classification/plot_classifier_comparison.html)  
랜덤 포레스트는 선형에 가까운 모습을, RBF SVM은 비선형에 가까운 모습을 보여준다. SVM의 정확도가 가장 높다. 의외로 NN이 선형으로 구분되는 모습을 보여주는 점이 특이하다. kNN도 잘 동작하는데 아마 오버피팅이 매우 심할 것 같다.
- [scikit-learn의 적절한 평가기 선택하기](http://scikit-learn.org/stable/tutorial/machine_learning_map/)  
스무고개 형태로 적절한 평가기를 선택할 수 있는 가이드를 제공한다.

# 팩키지
[데이터 사이언스 스쿨의 Scikit-Learn 패키지 소개](https://datascienceschool.net/view-notebook/293ece8b0d124fbaa4d4d52bb8f1cb42/)와 scikit-learn 공식 홈페이지의 [전체 API 레퍼런스](http://scikit-learn.org/stable/modules/classes.html)

# 클래스

- 전처리용 클래스
  - `fit()`: 학습
  - `transform()`: 추론할때 사용. 처리된 모델내에서 적용한다.
  - `fit_transform()`: 학습하면서 결과를 함께 리턴한다. 증분 학습이 되는건 아니다.
- 머신러닝 모형 클래스
  - `fit()`: 학습
  - `predict()`: 예측 또는 추론
  - `predict_proba()`: 확률 표시
  - `score()`
- Pipeline 클래스
  - 복수의 Preprocessor와 Model을 연결하여 하나의 Model처럼 행동
  - Model 클래스가 제공하는 공통 메서드를 모두 제공
  - pipeline 내부에서 Preprocessor에서 자료를 계속 변형한 후 마지막으로 Model에 입력

# 기타
scikit-learn은 학습 데이타에서 파생된 속성은 맨 뒤에 _를 붙여 사용자가 지정한 파라미터와 구분한다. 예를 들어 LinearRegression에서 `coef_`, `intercept_`.

## 상관 계수<sup>correlation coefficient</sup>
표준 상관 계수<sup>standard correlation coefficient</sup>는 `corr()`를 이용해 쉽게 계산할 수 있다.

```python
corr_matrix = housing.corr()

>>> corr_matrix["median_house_value"].sort_values(ascending=False)
median_house_value    1.000000
median_income         0.687170
total_rooms           0.135231
housing_median_age    0.114220
households            0.064702
total_bedrooms        0.047865
population           -0.026699
longitude            -0.047279
latitude             -0.142826
Name: median_house_value, dtype: float64
```
