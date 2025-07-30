---
layout: wiki 
title: GCS
tags: ["Cloud"]
last_modified_at: 2021/12/18 01:13:51
---

<!-- TOC -->

- [Google Cloud Storage](#google-cloud-storage)

<!-- /TOC -->

# Google Cloud Storage
Cloud Storage에 올려두고 로컬에 파일이 없을 경우 다운로드. 
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

업로드 과정도 동일. blob에 gcs 파일명, upload에 로컬 파일명
```python
client = storage.Client()
bucket = client.get_bucket(BUCKET)
blob = bucket.blob('X.pkl')
blob.upload_from_filename(os.getcwd() + '/X.pkl')
```
