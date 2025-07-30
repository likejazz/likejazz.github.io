---
layout: wiki 
title: Dask
tags: ["Data Science & Engineering"]
last_modified_at: 2022/11/03 19:44:08
---

<!-- TOC -->

- [Pandas 제약](#pandas-제약)
- [Dask](#dask)

<!-- /TOC -->

# Pandas 제약
Pandas의 1CPU/Memory 제약에서 벗어나기 위해,
- Pandas: Single CPU
- Dask: Multiple CPUs (사실상 Apache Spark와 동일한 역할을 한다)
- cuDF: Single GPU
- Dask-cuDF: Multiple GPUs in the Cluster

기타
- modin: Pandas와 동일한 인터페이스를 제공하기 때문에 매력적이나, 내부적으로 dask를 사용하며, modin의 유효성은 좀 더 실험 필요.

예전에 Dask를 처음 사용했을 때 학습 데이터를 걸러내기 위해서는 모든 데이터를 한 번 이상 봐야 하기 때문에 $$O(n)$$이라고 오해했으나 Dask는 필요한 부분만 로딩하거나 메모리를 초과하는 파일도 읽을 수 있다. 물론 `persist()`로 메모리에 고정하면 더 빠르게 구동된다.

`dask.dataframe`은 Pandas가 읽어 들이는 invalid 데이터를 제대로 처리해주지 못하는 경우가 있었다.

# Dask
그러나 이제 제법 쓸만하고(2022/11 기준) Cluster 구성 후 `persist()`로 메모리에 고정하면 제법 큰 파일도 빠르게 잘 실행된다.

```console
# Scheduler
$ dask scheduler

# Worker #1
$ dask worker tcp://x.x.x.x:8786 --nworkers 128
# Worker #2
$ dask worker tcp://x.x.x.x:8786 --nworkers 128
```

<img src="https://user-images.githubusercontent.com/1250095/199700691-b416c963-b5dc-48ad-a95b-25948220cc8f.png" width="70%">
위 대시보드 포트는 8787이며, Jupyter에서는 다음과 같이 접속한다.
```python
from dask.distributed import Client
client = Client('tcp://x.x.x.x:8786')
```