---
layout: wiki 
title: BigQuery
last-modified: 2020/09/02 16:57:20
---

<!-- TOC -->

- [권한](#권한)
- [Jupyter Notebook](#jupyter-notebook)

<!-- /TOC -->

bigquery에서 create native table from gcs로 schema는 auto detect. `Header rows to skip:`은 1. 컬럼이 string으로 잡히면 에러가 거의 안나는데, 이번에는 float으로 잡혀서 문자열에 대해 모두 에러 발생. invalid columns가 많을 경우 `Number of errors allowed:`를 충분히 늘려주면 도움이 된다. (1000 이상) bigquery dataset은 위치를 default로 한다. seoul(asia-northeast3)로 강제 지정했더니 gcs에서 import시 `Cannot read and write in different locations: source: asia, destination: asia-northeast3` 오류 발생. raw file은 gsutil을 이용해 gcs로 업로드 하는데, 사내 유선망은 100MB/s가 나와서 5.3G도(1천만건) 어렵지 않게 업로드 완료.

## 권한
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
conda는 여전히 설치되지 않음. 인증 문제가 있는데 vm 내에서 `$ gcloud auth application-default login`로 직접 처리.

```python
>>> %load_ext google.cloud.bigquery
>>> %%bigquery df --use_bqstorage_api
select * from ds.1m where vin = 'KMTHA81BBxxx'
>>> type(df)
pandas.core.frame.DataFrame
```
쿼리 결과가 dataframe에 맵핑된다. invalid value 때문에 모든 컬럼이 STRING이 되었다.

## Jupyter Notebook
```python
>>> %%bigquery df --use_bqstorage_api
select temperature, count(temperature) from ds.10m 
group by temperature having 
temperature != '-40.0' and
temperature != '\\N'
>>> df = df.sort_values(by='temperature')
>>> df.plot.bar(x='temperature', y='f0_', figsize=(20,10))
```
<img src="https://user-images.githubusercontent.com/1250095/89176000-c3e31680-d5c3-11ea-9d73-9a47ba9aefc5.jpg" width="80%">

data manipulation 과정(column type 변경, sorting) 정리 필요