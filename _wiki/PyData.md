---
layout: wiki 
title: PyData
tags: ["Data Science & Engineering"]
last_modified_at: 2025/06/08 19:30:33
---

<!-- TOC -->

- [Pandas](#pandas)
- [Matplotlib](#matplotlib)
- [Seaborn](#seaborn)
- [Anaconda](#anaconda)
  - [GCE](#gce)
- [Numba](#numba)
- [PyCaret](#pycaret)

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

Pandas는 매우 훌륭한 tabular 데이터 manipulation 도구지만 매우 큰 파일을 학습 또는 preprocessing 용도로 읽어들일때 1 CPU/Memory(모든 데이터가 메모리에 상주) 제약이 있다.

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

# Anaconda
```console
# How to Install
$ brew install --cask anaconda
```
PATH 확인 후 `$ conda init zsh`

```console
# Create a new environment
$ conda create --name pbv-data-analysis python=3.9

# Remove a old environment
$ conda env remove --name numba
# or
$ conda remove --name numba --all
```
If you want to install a lot of packages in default `base` env, be careful and think about it one more time because it makes the installation of additional packages very slow.

신규 conda 환경 설치
```console
$ conda create --name pycaret -c conda-forge python=3.8 pycaret
```
base에서 실행. conda-forge 기반이 빨리 설치되는걸 본적이 없다. 여전히 Solving environment에서 한참 걸린다. 이 때문에 conda로 환경 관리 외에는 패키지 설치가 꺼려진다.
```console
Solving environment: failed with repodata from current_repodata.json, will retry with next repodata source.
Solving environment: /  # It take up to 5 mins or more.
```

## GCE
GCE에서 Conda 설치 관련해 삽질한 내용

conda가 설치된 상태에서 추가 설치 및 업그레이드가 필요하다. 특히 RAPIDS는 conda로만 설치된다. C++14 및 CUDA 제약 사항으로 인해[^fn-conda] 그런데, `$ conda update --all --verbose` 조차 제대로 실행 안됨. 채널에서 정보를 가져오는데 상당한 제약 사항이 있다. 다음과 같이 수정 필요.

```console
$ conda config --show-sources
==> /home/gcp-user/.condarc <==
channel_priority: flexible
channels:
  - conda-forge
  - defaults
```

채널 설정을 `flexbile`로 두는게 핵심이다. 이 사소한 설정으로 인해 `Solving environment:`가 hang up되어 이후 과정이 진행 안됨. 대부분의 문서에서,

```console
$ conda config --set channel_priority strict
```
를 가이드 하지만 실제로는,
```console
$ conda config --set channel_priority flexible
```
에서 동작했다. 추가 확인이 필요하다. 

느리고, 설정에 sensitive한 설치로 conda 설치에 대한 신뢰가 많이 떨어져 있다. (특히 conda는 base에 패키지를 추가하면 새 환경 구성시 매우 느려지므로 주의가 필요하다) 다행히 Deep Learning VM으로 이미 RAPIDS가 설치된 이미지로 부팅할 수 있다.

[^fn-conda]: <https://medium.com/rapids-ai/rapids-0-7-release-drops-pip-packages-47fc966e9472>

conda 버전을 낮추면 된다는 얘기가 있어 따라해봤으나 여전히 안된다. 특히 4.6.x에서는 아예 killed 되어 버렸다. 여전히 RAPIDS conda 설치는 실패.

```console
$ conda install conda=4.6.14
$ wget https://repo.anaconda.com/pkgs/misc/conda-execs/conda-4.7.5-linux-64.exe
$ ./conda-4.7.5-linux-64.exe install -p /opt/conda conda=4.7.5
$ conda install -c rapidsai -c nvidia -c conda-forge -c defaults rapids=0.14 python=3.7 cudatoolkit=10.1
```

# Numba
Numba ~5 min tutorial[^fn-tutor] was very helpful. I've installed numba to new conda env using `$ conda install numba`. I've changed a part of the code to run randomly and it was also executed as quickly as before.

[^fn-tutor]: <https://numba.pydata.org/numba-doc/latest/user/5minguide.html>

# PyCaret
scikit-learn기반의 AutoML 라이브러리. 모델간 비교, k-fold CV(기본 10개로 보임), hyperparameters 튜닝, 시각화를 제공한다. XGBoost도 지원하며 GPU도 지원한다. sklearn은 GPU를 지원하지 않기 때문에 `use_gpu=True` 설정을 하면 cuML을 사용한다. ML 모든 절차를 대행해줘서 매우 편리하다.