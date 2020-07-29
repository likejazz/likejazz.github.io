---
layout: wiki 
title: GPU Data Science
last-modified: 2020/07/29 15:40:44
---

<!-- TOC -->

- [NumPy vs PyTorch](#numpy-vs-pytorch)
- [Pandas vs cuDF](#pandas-vs-cudf)
- [메모리](#메모리)
- [Dask](#dask)

<!-- /TOC -->

# NumPy vs PyTorch
```python
import numpy as np
import torch

x = np.random.random((10000, 10000))  # 1억
y = torch.from_numpy(x).float().to(torch.device("cuda:0"))
z = torch.from_numpy(x).float().to(torch.device("cpu"))

'''
>>> %timeit (np.sin(x) ** 2 / np.cos(x) ** 2).sum()
4.12 s ± 23.1 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

>>> %timeit (torch.sin(y) ** 2 / torch.cos(y) ** 2).sum()
18.9 ms ± 30 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)

>>> %timeit (torch.sin(z) ** 2 / torch.cos(z) ** 2).sum()
738 ms ± 5.64 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
'''
```
GPU에서 계산할때 220배 더 빠르다.[^fn-gpu-pydata] GCP n1-standard-4/Tesla T4 기준으로 V100이었으면 훨씬 더 빠를 것이다.

[^fn-gpu-pydata]: <https://medium.com/rapids-ai/first-impressions-of-gpus-and-pydata-348194660e40>

# Pandas vs cuDF
```python
import pandas as pd, numpy as np, cudf

pdf = pd.DataFrame({'x': np.random.random(10000000),
                    'y': np.random.randint(0, 10000000, size=10000000)})
gdf = cudf.DataFrame.from_pandas(pdf)

'''
>>> %timeit pdf.x.mean()
14.8 ms ± 78.1 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
>>> %timeit gdf.x.mean()
1.55 ms ± 109 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
'''
```
cuDF가 9.5배 더 빠르다. cuDF는 Python Console에서 디버깅이 되지 않는다. 반드시 turn off `Show Variables`. 대부분의 dataframe manipulation 작업에서 cuDF가 10배 이상 빠르다.

# 메모리
Tesla T4의 available memory는 14.726G로 나온다. 메모리가 꽉 차서 더 이상 할당이 안되는 경우 다음과 같은 오류가 발생한다.
```
CudaAPIError: [1] Call to cuMemcpyHtoD results in CUDA_ERROR_INVALID_VALUE
```

python REPL을 나가거나 하면 모든 메모리를 반환한다. T4도 14.7G의 메모리를 data manipluation 작업 등으로 초과할 경우 방법이 없기 때문에 Dask를 활용해야 한다. `dask_cudf.compute()`가 매우 불안하다. 계산 도중 실패하면 메모리를 반환하지 못하고 점유하고 있으며, 아무런 task 없이 `compute()` 할때도 저절로 메모리가 증가한다. 결국 메모리가 꽉 차서 오류를 발생한다.

아래는 XGBoost를 `gpu_hist`로 학습하는 모습이다.
<img src="https://user-images.githubusercontent.com/1250095/88621747-35125d80-d0dc-11ea-8ed8-a0d5832dfd2d.png" width="70%">

# Dask
큰 json을 읽어들이는데는 cuDF가 pandas 보다 훨씬 더 빠르다. dask는 `inplace=True`를 지원하지 않아서 많이 불편하다. 반면 cudf는 지원하기 때문에 큰 차이 없이 그대로 사용 가능하다. `dask.dataframe`은 pandas는 읽어 들이는 invalid 데이터를 제대로 처리해주지 못한다.

Dask의 시각화 너무 훌륭하다.
```python
import numpy as np
import dask.array as da

x = np.ones((10000, 1000, 1000))
# MemoryError: Unable to allocate 74.5 GiB for an array ...
x = da.ones((10000, 1000, 1000))
x
```
<img src="https://user-images.githubusercontent.com/1250095/88758890-84bc5c00-d1a4-11ea-995c-0ede3bc7fb14.png" width="50%">
```python
x.sum()
```
<img src="https://user-images.githubusercontent.com/1250095/88758896-86861f80-d1a4-11ea-8ead-91318ec8b867.png" width="30%">
```python
from dask.diagnostics import ProgressBar
with ProgressBar():
    print(x.sum().compute())
# [########################################] | 100% Completed | 11.7s
# 10000000000.0
```

CuPy를 이용해 gpu 연산을 시도해봤으나 `cudaErrorMemoryAllocation: out of memory`로 실패. 시간도 오래 걸렸다.
```python
import cupy
x = x.map_blocks(cupy.asarray)
x.sum().compute()
```