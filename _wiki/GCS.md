---
layout: wiki 
title: GCS
last-modified: 2020/09/04 15:56:02
---

<!-- TOC -->

- [Google Cloud Storage](#google-cloud-storage)
    - [CLI](#cli)

<!-- /TOC -->

# Google Cloud Storage
Cloud Storage에 올려두고 로컬에 파일이 없을 경우 다운로드. BigQuery를 이용하면 다운로드 필요 없이 원할때 생성하는 형태로 활용 가능.
```python
import os
from google.cloud import storage

BUCKET = "bucket"
PATH = '/ml-experiments'
FILE = 'sample.csv'

client = storage.Client()
if not os.path.exists(PATH + '/' + FILE):
    bucket = client.get_bucket(BUCKET)
    blob = bucket.blob(FILE)
    blob.download_to_filename(PATH + '/' + blob.name)

    print('Downloaded to {}'.format(PATH + '/' + blob.name))
```

## CLI
hive에서 쿼리 조회 결과, hdfs에 저장. 이후 로컬에 가져와서 맥북을 경유하여 gcs에 업로드 하는 과정 정리
```console
# 원격 서버
$ ssh xxx@10.12.109.xxx

# 분석 결과 CSV export
$ hive -e "INSERT OVERWRITE DIRECTORY 'output6.csv'
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
select * from ignxxx.aas_6 limit 10000000"
# 위에 1천만건 5.3G
 
# CSV 결과 조회
$ hdfs dfs -ls output6.csv
 
# 로컬로 가져오기
$ hdfs dfs -get output6.csv

# Append header(Linux only)
$ sed -i '1i HEADER' aas_6.csv  # 헤더 파일은 hive 웹에서 export로 미리 확보하고 있어야 함

# 맥북에서 streaming으로 GCS 업로드(5.3G 약 5분 소요, 로컬 디스크 공간 필요 없음)
$ scp xxx@10.12.109.xxx:aas_6.csv /dev/stdout | gsutil cp - gs://stark-xxx/aas_6.csv
```

원래 회사 유선망으로 100MB/s가 나오는데, 50MB/s 정도였고 remote server에서 가져오는 동안 멈춰있는 것으로 보인다.
