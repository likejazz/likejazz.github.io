---
layout: wiki 
title: BigQuery
last-modified: 2020/09/04 19:22:57
---

<!-- TOC -->

- [Python 설치](#python-설치)
- [인증](#인증)
- [DataFrame](#dataframe)
- [Jupyter Notebook](#jupyter-notebook)
- [Python 코드](#python-코드)

<!-- /TOC -->

bigquery에서 create native table from gcs로 schema는 auto detect. `Header rows to skip:`은 1. 컬럼이 string으로 잡히면 에러가 거의 안나는데, 이번에는 float으로 잡혀서 문자열에 대해 모두 에러 발생. invalid columns가 많을 경우 `Number of errors allowed:`를 충분히 늘려주면 도움이 된다. (1000 이상) bigquery dataset은 위치를 default로 한다. seoul(asia-northeast3)로 강제 지정했더니 gcs에서 import시,

```
Cannot read and write in different locations: source: asia, destination: asia-northeast3
```
오류 발생. raw file은 gsutil을 이용해 gcs로 업로드 하는데, 사내 유선망은 100MB/s가 나와서 5.3G도(1천만건) 어렵지 않게 업로드 완료.

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

## 인증
인증 문제는 vm 내에서,

```
$ gcloud auth application-default login
```
로 직접 처리. CLI 기본 인증,

```
$ gcloud auth list
``` 
다르기 때문에 주의 필요.

## DataFrame
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

## Python 코드
코드에서 BigQuery 호출 코드

```python
import google.auth
from google.cloud import bigquery
from google.cloud import bigquery_storage_v1beta1


# Explicitly create a credentials object. This allows you to use the same
# credentials for both the BigQuery and BigQuery Storage clients, avoiding
# unnecessary API calls to fetch duplicate authentication tokens.
def bigquery_auth(project_id: str = 'edith-xxx') -> None:
    logging.info('[AUTH] Create a credentials.')
    credentials, _ = google.auth.default(
        scopes=["https://www.googleapis.com/auth/cloud-platform"]
    )

    # Make clients.
    bqclient = bigquery.Client(
        credentials=credentials,
        project=project_id,
    )
    bqstorageclient = bigquery_storage_v1beta1.BigQueryStorageClient(
        credentials=credentials
    )
    logging.info('[AUTH] Done.')

    globals()['bqclient'] = bqclient
    globals()['bqstorageclient'] = bqstorageclient


def bigquery_results(query_string: str, idx='N/A') -> pd.core.frame.DataFrame:
    # Download query results.
    logging.info(f'[SQL] #{idx} BigQuery runs.')
    dataframe = (
        globals()['bqclient'].query(query_string)
            .result()
            .to_dataframe(bqstorage_client=globals()['bqstorageclient'])
    )
    return dataframe

bigquery_auth()
df = bigquery_results("""
SELECT
  vin,
  COUNT(DISTINCT triplength) AS triplengths,
  COUNT(*) AS datas
FROM
  xxxds.xxxs_0831
GROUP BY
  vin
HAVING
  COUNT(*) BETWEEN 300 AND 7200
""", 'VIN')
```