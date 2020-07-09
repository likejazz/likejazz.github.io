---
layout: wiki 
title: Santander Product Recommendation
last-modified: 2020/07/09 16:17:31
---

<!-- TOC -->

- [개요](#개요)
- [EDA](#eda)
- [Baseline](#baseline)
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
TQDM을 이용해 progress bar를 표기할 수 있다. 단, 여러 트랜잭션으로 나뉘어져 있을때. `pd.read_csv()`는 단일 트랜잭션이라 표시가 쉽지 않다. `pd.read_csv(chunksize=10000)`로 가능하다.

# Winner's Code