---
layout: wiki 
title: RAPIDS
tags: ["Data Science & Engineering"]
last_modified_at: 2023/06/01 13:46:32
---

- [Dask-cuDF](#dask-cudf)

# Dask-cuDF
CLI 실행:
```
$ dask scheduler --dashboard-address :30001
$ dask-cuda-worker 127.0.0.1:8786
```
dask 및 jupyter등 여러 프로세스를 한 번에 구동하기 위해 supervisord 사용
