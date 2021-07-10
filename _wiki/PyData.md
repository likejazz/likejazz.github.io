---
layout: wiki 
title: PyData
last_modified_at: 2021/06/30 22:21:06
---

<!-- TOC -->

- [Pandas](#pandas)
    - [제약](#제약)
- [Matplotlib](#matplotlib)
- [Seaborn](#seaborn)

<!-- /TOC -->

# Pandas
DataFrame 메모리 점유 확인
```python
df.info(memory_usage='deep')
```

『누구나 파이썬 통계분석』 ch4 성적 데이터 이용  
Scatter plot form dataframe with index on x-axis[^fn-df]
```python
df.reset_index().plot(kind='scatter', x='index', y='score')
plt.show()
```

[^fn-df]: <https://stackoverflow.com/a/49835113/3513266>

Pandas show all rows.
```python
pd.set_option('display.max_rows', 100)
df.nunique()
```

endpoint로 끝나는 모든 컬럼 제거
```python
df = df.drop(columns=df.filter(regex='^endpoint').columns)
```

## 제약
Pandas는 매우 훌륭한 tabular 데이터 manipulation 도구지만 매우 큰 파일을 학습 또는 preprocessing 용도로 읽어들일때 1 CPU/Memory(모든 데이터가 메모리에 상주) 제약이 있다.

CPU/Memory 제약에서 벗어나기 위해,
- dask: `compute()`시 시간이 오래 걸리는건 동일하다. 좀 더 스마트한 계산을 기대했으나 어차피 학습 데이터를 걸러내기 위해서는 모든 데이터를 한 번 이상 봐야 하기 때문에 O(n)이다. 이 부분은 최적화 할 수 없으며, dask는 여기에 적절하지 않다.
- cudf: GPU의 메모리가 16G 이기 때문에 큰 파일은 로드하지 못한다. manipulation에서 좋은 성능을 기대할 수 있으나 BigQuery에 비해 장점을 찾기 어렵고, Pandas의 모든 기능을 지원하는 것은 아니기 때문에 여러모로 불편함이 많다. 실제로 출력할때도 print를 하던지 매 번 `to_pandas()`를 해야 한다. 전체를 다 보는 용도로는 마찬가지로 적절하지 않다.
- modin: Pandas와 동일한 인터페이스를 제공하기 때문에 매력적이나, 내부적으로 dask를 쓰기 때문에 별 차이가 없고, 마찬가지로 전체 데이터를 다 보는 용도로는 활용이 어렵다. modin의 유효성은 좀 더 실험 필요.

결국 4G, 8G CSV를 한꺼번에 올리려면 그 만큼의 메모리가 필요하고, 이건 어떠한 라이브러리도 해결할 수 없는 부분. 따라서 아예 CSV를 미리 preprocessing하고 thread 갯수대로 실행할 수 있게끔 8개의 파일로 분리. 각각 combine 하지도 않고 그대로 두어 나중에 학습 데이터 추출시 파일을 하나씩 읽어서 조금씩 처리하도록 구성. 이래야 메모리 제약 문제를 벗어날 수 있다. 이렇게 해도 0901.pkl은 원래 파일 자체가 크기 때문에(8G) 8개의 쓰레드가 모두 돌아갈때는 23.6G의 메모리를 차지한다. 매 번 preprocessing 할때는 4시간 걸리던 작업을, 미리 preprocessing 하고(반나절), 이후에는 12분 정도만 추가로 preprocessing하면 된다.  
ie. 4시간 → 12분

Pandas는 파이썬 같다. 1CPU에 메모리 제약까지 있지만 대신 매우 강력하고, 안정적이다. 이를 효율적으로 병렬화 하던지 잘 활용하는건 개발자의 몫 ...

# Matplotlib
In Matplotlib, what does the argument mean in fig.add_subplot(111)?  
"111" means "1x1 grid, first subplot" and "234" means "2x3 grid, 4th subplot". [^fn-subplot]

```python
# 표본평균 Sample Mean
df = pd.read_csv('data/ch4_scores400.csv')
scores = np.array(df['score'])
sample_means = [np.random.choice(scores, 20).mean()
                for _ in range(10000)]

ax = plt.figure().add_subplot(111)
ax.hist(sample_means, bins=100, range=(0, 100), density=True)
ax.vlines(np.mean(scores), 0, 0.13, 'gray')  # 모평균 Population Mean
# Methods for subplots.
ax.set_xlim(50, 90)  # x축 간격을 50에서 90까지로
ax.set_xlabel('score')
ax.set_ylabel('relative frequency')
plt.show()
```
[^fn-subplot]: <https://stackoverflow.com/a/3584933/3513266>  
범위를 조정하고 레이블을 부여하는건 subplot만 가능하다.

<img src="https://user-images.githubusercontent.com/1250095/84562068-6c2aea80-ad8c-11ea-93bc-be1dbcd728a8.png" width="60%">

# Seaborn
regplot(), lmplot(), jointplot()등 단순 차트 외에 regression model과 결합된 플롯이 매우 강력하다.

```
>>> sns.jointplot(x="EngDispl", y="FE", data=cars10, kind="reg")
```

<img src="https://user-images.githubusercontent.com/1250095/99523276-309ae900-29da-11eb-82f2-2eafb1d27491.png" width="70%">