---
layout: wiki 
title: GPU Data Science
tags: ["Data Science & Engineering"]
last_modified_at: 2022/11/03 23:55:17
---

<!-- TOC -->

- [NumPy vs PyTorch](#numpy-vs-pytorch)
- [메모리](#메모리)
- [nvtop](#nvtop)
- [Low Precision Inference on GPU](#low-precision-inference-on-gpu)

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
GPU에서 계산할때 40배 더 빠르다.[^fn-gpu-pydata] GCP n1-standard-4/Tesla T4 기준으로, V100이었으면 훨씬 더 빠를 것이다.

[^fn-gpu-pydata]: <https://medium.com/rapids-ai/first-impressions-of-gpus-and-pydata-348194660e40>

# 메모리
Tesla T4의 available memory는 14.726G로 나온다. 메모리가 꽉 차서 더 이상 할당이 안되는 경우 다음과 같은 오류가 발생한다.
```
CudaAPIError: [1] Call to cuMemcpyHtoD results in CUDA_ERROR_INVALID_VALUE
```

python REPL을 나가거나 하면 모든 메모리를 반환한다. T4도 14.7G의 메모리를 data manipluation 작업 등으로 초과할 경우 방법이 없기 때문에 Dask를 활용해야 한다. `dask_cudf.compute()`가 매우 불안하다. 계산 도중 실패하면 메모리를 반환하지 못하고 점유하고 있으며, 아무런 task 없이 `compute()` 할때도 저절로 메모리가 증가한다. 결국 메모리가 꽉 차서 오류를 발생한다. 

예전에는 클러스터도 구동하지 않고 돌려서 그랬던거 같다(2022/11).

아래는 XGBoost를 `gpu_hist`로 학습하는 모습이다.
<img src="https://user-images.githubusercontent.com/1250095/88621747-35125d80-d0dc-11ea-8ed8-a0d5832dfd2d.png" width="70%">

# nvtop
htop과 유사한 nvtop 설치[^fn-nvtop] apt에는 없어서 예전에는 소스 컴파일했다.

```console
$ sudo apt install cmake libncurses5-dev libncursesw5-dev
$ git clone https://github.com/Syllo/nvtop.git
$ mkdir -p nvtop/build && cd nvtop/build
$ cmake .. -DNVIDIA_SUPPORT=ON -DAMDGPU_SUPPORT=OFF -DINTEL_SUPPORT=OFF
$ make
$ sudo make install
```

[^fn-nvtop]: <https://github.com/Syllo/nvtop#nvtop-build>

빌드 오류가 발생하면 `$ conda deactivate` 후 다시 cmake를 실행한다. 요즘은 apt로 설치 가능하다고 가이드에 나와 있다.
```
$ sudo apt install nvtop
```

# Low Precision Inference on GPU
<img src="https://user-images.githubusercontent.com/1250095/64407402-2e8f8780-d0bf-11e9-8eb9-ff3aa8ceb348.jpg" width="80%">

T4 GPU에서 Quantized 연산으로 8.1 TFLOPS를 130 TOPS, 16배 개선할 수 있다. 그러나, BERT SQuAD 1.1(F1) 점수가 Int8에서 6.43%나 낮다. Activation으로 GeLU10을 사용해 0.67% 이내로 줄일 수 있다. [^fn-pdf] MRPC는 두 문장의 유사 문장 판별 태스크로 마찬가지로 성능 차이를 0.80% 이내로 줄일 수 있다.

[^fn-pdf]: [LOW PRECISION INFERENCE ON GPU(PDF)](https://developer.download.nvidia.com/video/gputechconf/gtc/2019/presentation/s9659-inference-at-reduced-precision-on-gpus.pdf) 

<img src="https://user-images.githubusercontent.com/1250095/64407574-9c3bb380-d0bf-11e9-908c-83783fcb6692.jpg" width="80%">
