---
layout: wiki 
title: Santander Product Recommendation
last-modified: 2020/07/21 17:27:23
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
XGBoost                       : 0.8421 (1min 46s)
LightGBM(boosting='dart')     : 0.8373 (15.56s)
Neural Network(20, 20, scaled): 0.8367 (13.31s)
SVM(LinearSVC, scaled)        : 0.8191 (5min 6s)
Random Forest(n_estimators=50): 0.8142 (5.44s)
Random Forest(n_estimators=10): 0.7786 (1.21s)
Bernoulli Naive Bayes         : 0.7508 (0.09s)
Decision Tree                 : 0.6506 (0.8s)
SVM(kernel='rbf')             : N/A (52min 47s)
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
그러나 prob가 False일때는 그나마 1시간에 되던 학습이 하루 종일 걸린다. 이마저도 제대로 된 결과가 나오지 않고, 한가지 결과가 나오는 형태로 잘못 학습됨. scale 하니까 학습 속도가 훨씬 빨라진 느낌이다. LinearSVC도 scaling 이후 6% 정도 오른다.

### Neural Network
SVM과 유사하게 동일한 값만 나오며 학습이 되지 않는다. 네트워크를 깊게 해도 마찬가지. but MLP is sensitive to feature scaling(매우 중요!)[^fn-tips] 이후 학습이 잘 된다.

[^fn-tips]: <https://scikit-learn.org/stable/modules/neural_networks_supervised.html#tips-on-practical-use>

### LightGBM
XGBoost 보다 훨씬 빠르지만 성능이 더 떨어진다. `boosting_type='gbrt'` 보다 `dart`일때 성능이 훨씬 좋다. 속도는 2배 느려진다.

# Winner's Code