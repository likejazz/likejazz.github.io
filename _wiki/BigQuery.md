---
layout: wiki 
title: BigQuery
tags: ["Cloud"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [생성](#생성)
    - [테이블 생성](#테이블-생성)
    - [테이블 이동](#테이블-이동)
- [연동](#연동)
    - [Python 설치](#python-설치)
    - [데이터 연동](#데이터-연동)
- [활용](#활용)
    - [인증](#인증)
    - [DataFrame 맵핑](#dataframe-맵핑)
    - [Jupyter Notebook, Python](#jupyter-notebook-python)
    - [성능](#성능)
    - [GIS 함수, 지리 데이터 연동](#gis-함수-지리-데이터-연동)
- [BigQuery ML](#bigquery-ml)
    - [피어슨 상관 계수](#피어슨-상관-계수)
    - [모델](#모델)

<!-- /TOC -->

# 생성
## 테이블 생성
bigquery에서 create native table from gcs로 schema는 auto detect. `Header rows to skip:`은 1. 컬럼이 string으로 잡히면 에러가 거의 안나는데, 이번에는 float으로 잡혀서 문자열에 대해 모두 에러 발생. invalid columns가 많을 경우 `Number of errors allowed:`를 충분히 늘려주면 도움이 된다. (1000 이상) bigquery dataset은 위치를 default로 한다. seoul(asia-northeast3)로 강제 지정했더니 gcs에서 import시,

```
Cannot read and write in different locations: source: asia, destination: asia-northeast3
```
오류 발생. raw file은 gsutil을 이용해 gcs로 업로드 하는데, 사내 유선망은 100MB/s가 나와서 5.3G도(1천만건) 어렵지 않게 업로드 완료.

## 테이블 이동
multi-region에서 US와 EU간 테이블 조인이 되지 않는다. 동일 위치로 이동해야 하는데, dataset 이동에는 Transfer API를 활성화 해야 한다. 이후 copy dataset을 하면 Transfers에 추가되고 비동기로 진행된다. 2G 테이블 이동에 약 1분 이상 소요됐다.


# 연동
## Python 설치
BigQuery 쿼리 결과를 로컬 pandas로 내려서 분석 시도. Data Studio는 사용법도 다르고 무엇보다 data source connection 오류가 있어서 데이터를 부르지도 못했다. 로컬 분석은 가이드[^fn-guide]를 참고했다.

[^fn-guide]: <https://cloud.google.com/bigquery/docs/bigquery-storage-python-pandas#pip> 
```console
# Linux
$ pip install --upgrade google-cloud-bigquery[bqstorage,pandas]

# OSX
$ pip install --upgrade google-cloud-bigquery
# AttributeError: module 'grpc.experimental.aio' has no attribute 'Call' 오류로 인해
$ pip install --upgrade grpcio
```

pip는 sudo 설치해도 사용할 수 없기 때문에, 다음과 같이 owner 조정
```
$ cd opt/
$ chown -R gcp-user anaconda3/
$ pip install google-cloud-bigquery google-cloud-bigquery-storage
```
conda는 여전히 설치되지 않음. 

## 데이터 연동
<img src="https://user-images.githubusercontent.com/1250095/102486006-be3a2900-40ab-11eb-9dcf-dad8eadb6813.jpg" width="50%">

(구글 빅쿼리 완벽 가이드, 2020)

# 활용
## 인증
인증 문제는 GCE 내에서,

```
$ gcloud auth application-default login
```
로 직접 처리. CLI 기본 인증,

```
$ gcloud auth list
``` 
다르기 때문에 주의 필요.

## DataFrame 맵핑
```python
>>> %load_ext google.cloud.bigquery
>>> %%bigquery df
select * from ds.1m where vin = 'KMTHA81BBxxx'
>>> type(df)
pandas.core.frame.DataFrame
```
쿼리 결과가 dataframe에 맵핑된다. invalid value 때문에 모든 컬럼이 STRING이 되었다.

## Jupyter Notebook, Python
GCP의 AI Platform에서 노트북을 생성하면 별도 설정 없이 바로 활용 가능하다.

```python
>>> %%bigquery df
select temperature, count(temperature) from ds.10m 
group by temperature having 
temperature != '-40.0' and
temperature != '\\N'
>>> df = df.sort_values(by='temperature')
>>> df.plot.bar(x='temperature', y='f0_', figsize=(20,10))
```
<img src="https://user-images.githubusercontent.com/1250095/89176000-c3e31680-d5c3-11ea-9d73-9a47ba9aefc5.jpg" width="80%">

data manipulation 과정(column type 변경, sorting) 정리 필요

[Python에서 BigQuery 호출 코드](https://gist.github.com/likejazz/01e76b10364a47bf9c4aa67c8ab49b33)

## 성능
BigQuery는 columnar-based이기 때문에 컬럼을 최소화 할수록 성능이 개선된다. 일반적인 DB와 마찬가지로 `SELECT *` 보다 `SELECT id1, id2`가 훨씬 더 성능이 좋다.

## GIS 함수, 지리 데이터 연동
ST_GeoPoint 위경도 POINT()로 표현, Geo Viz에서 시각화 가능

```sql
SELECT
  name,
  ST_GeogPoint(longitude, latitude) AS location
FROM
  `bigquery-public-data`.london_bicycles.cycle_stations
WHERE
  id BETWEEN 1 AND 5000
```

# BigQuery ML
<img width="80%" src="https://user-images.githubusercontent.com/1250095/99152817-7e1c0b00-26e7-11eb-901a-25e301f8a48d.png">

(구글 빅쿼리 완벽 가이드, 2020)

## 피어슨 상관 계수
자전거 수와 평균 대여 시간의 피어슨 상관 계수 확인

```sql
SELECT
  CORR(bikes_count, duration) AS corr
FROM
  `bigquery-public-data`.london_bicycles.cycle_hire
JOIN
  `bigquery-public-data`.london_bicycles.cycle_stations
ON
  cycle_hire.start_station_name = cycle_stations.name
```

> 피어슨 상관 계수는 선형 의존성이 있으면 1.0의 절대값 을 가지며, 선형적으로 독립적이면 0.0을 가진다. 따라서 -0.0039이라는 결과는 bikes_count와 duration 컬럼 사이의 관계가 서로 독립적임을 뜻한다.

(p427, 구글 빅쿼리 완벽 가이드, 2020)

## 모델
`dayofweek`, `hourofday`를 입력값으로 하는 머신러닝 모델 생성  
(london_bicycles 데이터셋이 EU에 있기 때문에 EU 데이터셋을 별도로 생성했음)

```sql
CREATE OR REPLACE MODEL
  ch09eu.bicycle_model OPTIONS(input_label_cols=['duration'], model_type='linear_reg') AS

SELECT
  duration,
  start_station_name,
  CAST(EXTRACT(dayofweek FROM start_date) AS STRING) AS dayofweek,
  CAST(EXTRACT(hour FROM start_date) AS STRING) AS hourofday
FROM
  `bigquery-public-data`.london_bicycles.cycle_hire
```

예측
```sql
SELECT * FROM
  ML.PREDICT(MODEL `ch09eu.bicycle_model`,
    (SELECT
      'Rodney Street, Angel' AS start_station_name,
      '7' AS dayofweek,
      '0' AS hourofday
    ) 
  )
```

가중치
```sql
SELECT * FROM ML.WEIGHTS(MODEL ch09eu.bicycle_model)
```
