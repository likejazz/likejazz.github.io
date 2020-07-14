---
layout: wiki 
title: Santander Product Recommendation
last-modified: 2020/07/14 17:04:52
---

<!-- TOC -->

- [개요](#개요)
- [EDA](#eda)
- [Baseline](#baseline)
    - [Feature Engineering](#feature-engineering)
    - [Training](#training)
    - [Evaluation](#evaluation)
- [Winner's Code](#winners-code)

<!-- /TOC -->

# 개요
[Kaggle Competition](https://www.kaggle.com/c/santander-product-recommendation)  
고객이 은행의 24개의 금융 상품 중 새롭게 가입하는 상품 추천하기

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
20,000개를 추리고 XGBoost로 학습하고, actual이 비어 있는 값은 bypass 처리하면(기존에는 0점 처리가 평균을 까먹게 되는 요인이 되었음) 0.79 점수가 나온다. 물론 이 점수는 아무 의미가 없는 값이다.

## Evaluation
이 competition의 평가 방식은 Map@7이다. 그런데, 상품 번호 순으로 소팅해서 비교한다. 그렇게 하면 확률 점수가 의미가 없게 되므로 확률 점수 순으로 7개를 자르고 결과를 비교하는게 맞다.


# Winner's Code