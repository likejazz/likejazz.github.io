---
layout: wiki 
title: Santander Product Recommendation
tags: ["Machine Learning"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [개요](#개요)
- [EDA](#eda)
- [Baseline](#baseline)
    - [Feature Engineering](#feature-engineering)
    - [Training](#training)
    - [Evaluation](#evaluation)
        - [XGBoost](#xgboost)
        - [Random Forest](#random-forest)
        - [Gradient Boosting](#gradient-boosting)
        - [Naive Bayes](#naive-bayes)
        - [SVM](#svm)
        - [Neural Network](#neural-network)
        - [LightGBM](#lightgbm)
        - [CatBoost](#catboost)
- [Winner's Code](#winners-code)

<!-- /TOC -->

# 개요
[Kaggle Competition](https://www.kaggle.com/c/santander-product-recommendation)  
고객이 은행의 24개의 금융 상품 중 새롭게 가입하는 상품 추천하기[^fn-code]

[^fn-code]: <https://github.com/jsrimr/kaggle_santaner/tree/master/kaggle_santander_product_recommendation>

# EDA
```python
# 나이를 정렬하여 가로 표현
plt.figure(figsize=(4, 12))
sns.countplot(y="age", data=a, order=sorted(a['age'].unique()))
plt.show()
```

<img src="https://user-images.githubusercontent.com/1250095/86990723-308f0f00-c1d8-11ea-8c11-2c6bca06eac9.png" width="20%">

```python
b[b.columns[:5]].describe()
```

number인 경우 `describe()`는 mean,std,min등을 보여주지만 object는 다음과 같이, unique와 top의 freq를 보여준다.
```
Out[8]: 
        fecha_dato ind_empleado pais_residencia     sexo  fecha_alta
count      1859727      1859727         1859727  1859717     1859727
unique           2            5             118        2        6750
top     2016-05-28            N              ES        V  2014-07-28
freq        931453      1858687         1851744  1009363        6846
```

# Baseline
```python
for col in tqdm.tqdm(categorical_cols):
    df[col], _ = df[col].factorize(na_sentinel=-99)
```
Encode the object as an enumerated type or categorical variable.  
TQDM을 이용해 progress bar를 표기할 수 있다. 단, 여러 트랜잭션으로 나뉘어져 있을때. `pd.read_csv()`는 단일 트랜잭션이라 표시가 쉽지 않다. `pd.read_csv(chunksize=10000)`로 가능하다.

## Feature Engineering
`df_lag`은 한달 후로 셋팅하고 컬럼명을 `_prev`로 바꾼다. 나중에 merge할 때 한 달 후 row에 df_lag 데이터가 prev 값으로 들어갈 것이다.

```
df[(df['ult_fec_cli_1t_year_prev'] == -99) & (df['fecha_dato'] == '2016-04-28') ].count()['ncodpers']
```
multiple conditions에 대한 갯수. 컬럼명을 지정하지 않으면 모든 컬럼에 대한 갯수가 나온다.

```
pd.set_option("display.max_rows", 999)
trn[(trn['ult_fec_cli_1t_year_prev'] == -99) & (trn['fecha_dato'] == '2016-04-28')].head(2).transpose()
```
columns 제한 때문에 가로로 모두 출력되지 않을때 세로 출력.

## Training
20,000개를 추리고 XGBoost로 학습하고, actual이 비어 있는 값은 bypass 처리하면(기존에는 0점 처리가 평균을 까먹게 되는 요인이 되었음) 0.79가 나온다. 물론 이 점수는 큰 의미가 없다.

## Evaluation
이 competition의 평가 방식은 MAP@7이다. 그런데, 상품 번호 순으로 소팅해서 비교한다. 그렇게 하면 확률 점수가 의미가 없게 되므로 확률 점수 순으로 7개를 자르고 결과를 비교하는게 맞다.

MAP@7에 비어있는 값에 대한 처리도 추가하려 했으나, 편의상 비어있는 값은 모두 제외, 실제 값이 있는 actual 3만여개(한달치)에 대해 validation 진행. prob 점수 기준 없이 7개 추출 후 비교. 7개가 많다고 생각되지만 어차피 뒷쪽에 위치한다면 그 만큼 penalty를 받게 되어 무방하다고 판단.

학습: 2016-03, 04 2개월치  
검증: 2016-05 1개월치

```
Gradient Boosting             : 0.8434 (3min 34s)
CatBoost                      : 0.8429 (2min 6s)
XGBoost(GPU)                  : 0.8406 (9.37s) 10x faster(Tesla T4)
XGBoost                       : 0.8405 (1min 46s)
CatBoost(GPU)                 : 0.84   (12.21s)
LightGBM(boosting='dart')     : 0.8373 (15.56s)
Neural Network(20, 20, scaled): 0.8367 (13.31s)
SVM(LinearSVC, scaled)        : 0.8191 (5min 6s)
Random Forest(n_estimators=50): 0.8142 (5.44s)
Random Forest(n_estimators=10): 0.7786 (1.21s)
Bernoulli Naive Bayes         : 0.7508 (0.09s)
SVM(kernel='rbf', scaled)     : 0.6821 (7hr 31min 13s)
Decision Tree                 : 0.6506 (0.8s)
```

각 실행 코드는 [노트북](https://nbviewer.jupyter.org/github/likejazz/jupyter-notebooks/blob/master/machine-learning/titanic.ipynb) 참고.

### XGBoost

XGBoost의 feature importances 7개 결과는 다음과 같다.  
```
Feature importance:
0 ('renta', 8958)
1 ('antiguedad', 7359)
2 ('age', 7223)
3 ('age_prev', 5929)
4 ('antiguedad_prev', 5727)
5 ('nomprov', 4836)
6 ('fecha_alta_month', 4613)
```

`xgboost.XGBClassifier`로 scikit-learn like wrapper를 사용할 수 있다. matrix 변환이 필요 없어 편하다. 그러나, `verbose=True`에서도 아무것도 출력되지 않으며, `model.get_fscore().items()`가 되지 않음. `model.get_booster().get_fscore().items()`로 가능하다.

GPU로 속도 개선 효과가 크다.

### Random Forest
dt 결과가 rf 결과와 동일하게 나와서 당황. rf의 `n_estimators=10`이 default인데, 이 정도는 충분한 성능이 나오지 않는다. dt와 동일한 성능이 나옴. 50으로 조정했고, 괜찮은 성능을 끌어낼 수 있었다.
```python
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(n_estimators=50)
rf.fit(XY_trn[features], XY_trn['y'])
```

### Gradient Boosting
GradientBoostingClassifier는 상당히 오래 걸린다. dt는 10초면 끝나는데, gb는 3min 34s가 걸린다. 그러나 결과는 매우 좋다.

### Naive Bayes
[Multinomial](/multinomial-naive-bayes/)은 negative number가 있다고 학습이 되지 않는다. Bernoulli는 잘 동작함. 매우 빠른 속도로 학습되며 나쁘지 않은 성능을 보여준다.

### SVM
너무 오래 걸린다. gb보다 훨씬 더 오래 걸림. 그나마 LinearSVC가 가장 빠르지만 생각보다 성능이 좋지 않다. LinearSVC는 predict시 `._predict_proba_lr()` 사용. SVM 학습 옵션은 다음과 같다.
```
clf = svm.SVC(kernel='rbf', gamma=0.7, C=1.0, verbose=True, probability=True)
```
그러나 prob가 False일때는 그나마 1시간에 되던 학습이 하루 종일 걸린다. 이마저도 제대로 된 결과가 나오지 않고, 한가지 결과가 나오는 형태로 잘못 학습됨. scale 하니까 학습 속도가 훨씬 빨라진 느낌이다. 성능은 나오지만 CPU를 1개 밖에 사용하지 않아 학습이 매우 늦다. LinearSVC도 scaling 이후 6% 정도 오른다.

### Neural Network
SVM과 유사하게 동일한 값만 나오며 학습이 되지 않는다. 네트워크를 깊게 해도 마찬가지. but MLP is sensitive to feature scaling(매우 중요!)[^fn-tips] 이후 학습이 잘 된다.

[^fn-tips]: <https://scikit-learn.org/stable/modules/neural_networks_supervised.html#tips-on-practical-use>

### LightGBM
XGBoost 보다 훨씬 빠르지만 성능이 더 떨어진다. `boosting_type='gbrt'` 보다 `dart`일때 성능이 훨씬 좋다. 속도는 2배 느려진다. GPU를 사용하려면 컴파일을 새로 해야한다. 반면 GPU utilization 비율이 낮고, 속도 개선 효과도 없다.

### CatBoost
progress bar가 default로 나오는게 아름답다. 학습은 가장 오래 걸렸지만 그만큼 성능은 잘 나온다. GPU로 속도 개선 효과가 크다.

# Winner's Code
(머신러닝 탐구생활, 2018)
1. 데이터 전처리: 날짜 관련 결측값을 채우려고 시도했으며, age 변수와 antiguedad 변수를 수정했고, lag 5 이상의 데이터를 만들기 위해 2014년도 데이터를 생성하려고 했지만 소용 없었다.
1. 피처 엔지니어링: lag 5, lag 데이터에 대한 기초 통계(최솟값, 최댓값, 표준편차 등)가 주된 피처 엔지니어링이었다. 5 이상의 lag데이터로 성능 개선을 보지 못했다.
    - 결측값을 0.0으로 대체하고, 정수형으로 변환한다.
    - `LabelEncoder`를 이용해(`label_encode()` 함수) 범주형 변수를 수치형으로 변환. 예를 들어 `canal_entrada` 처럼 알파벳 세글자로 암호화된 변수이고 상위 5개가 대부분을 차지하는 값이다. 수치형으로 변환. `pais_residencia` 고객 거주 국가 알파벳 두 자리도 마찬가지.
    - 빈도 상위 100개의 데이터에 고유값을 순위로 대체하고 그외 빈도가 낮은 값은 모두 0으로 변환(`encode_top()` 함수) 데이터 전체가 아닌 고빈도 데이터에 대한 정보를 추출한다.
    - 날짜를 숫자로 변환한다.
    - 범주형 변수를 이진 변수로 생성하는 one-hot 인코딩을 수행한다. 범주형 데이터를 하나의 열에서 label encoding 하는 것 보다 표현력이 높아지지만 고유값의 숫자만큼 데이터의 열이 늘어나기 때문에 고유값이 적은 데이터에서 선호되는 피처 엔지니어링 기법이다.
    - 과거 데이터를 lag_x 형태로 구성. lag-5 변수가 가장 중요한 역할을 했다고.
1. 기본 모델 및 모델 통합: mlogloss 기준으로 학습된 부스팅 기본 모델을 다수 만들었다. 초기 예측값 대비 피어슨 상관관계가 가장 낮은 모델 결과물을 통합했다.
    - 1명의 고객이 같은 날짜에 2개 이상의 금융 제품을 '신규 구매'할 경우, 해당 고객의 데이터에서는 유효한 데이터 2개가 인식되어 2줄의 데이터를 차지하게 된다. 이럴 경우 신규 구매 건구사 많은 고객 데이터의 분포가 필요 이상으로 많아져, 건수가 적은 고객들에 대한 정확도가 낮아질 우려기 있다. XGBoost와 LightGBM은 이러한 데이터 분포를 고려해 각 고객 데이터의 가중치를 부여하는 weight 변수를 지원한다. 신규 구매 건수가 많은 고객의 weight를 낮게 배정하여, 올바른 데이터 분포를 유지한다.  
    ```python
    weights = np.exp(1/counts - 1)
    ```

링크: 또 다른 실험[^fn-kaggle], Slideshare[^fn-slide]

[^fn-kaggle]: <https://yseon99.tistory.com/category/Kaggle%20%EB%8C%80%ED%9A%8C/3.%20Santander%20Product%20Recommendation>
[^fn-slide]: <https://www.slideshare.net/HyunJinJung6/3-santander-72072254>
