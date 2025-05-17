---
layout: wiki 
title: llama2.c
tags: ["Transformer"]
last_modified_at: 2024/10/29 18:57:55
---

<!-- TOC -->

- [실행](#실행)
- [Quantization](#quantization)
- [OpenMP](#openmp)
- [실행 속도 정리](#실행-속도-정리)
  - [llama2-7b.bin](#llama2-7bbin)
  - [stories110M.bin](#stories110mbin)
  - [CUDA](#cuda)
- [M1에서 clang 빌드](#m1에서-clang-빌드)
- [convert.py](#convertpy)

<!-- /TOC -->

# 실행
M1 Pro:
```shell
$ ./run stories15M.bin -t 0
Once upon a time, there was a little girl named Lily.
...
achieved tok/s: 112.314709
```
DGX Station A100:  
`achieved tok/s: 72.649573`  
128개 코어 중 하나만 사용하며 M1에 비해 늦다.

# Quantization
quantized model은 DGX에서 multithreads로 실행할 때 3x ↑

`matmul()` 함수를 비교하면 다음과 같다.
```c
// run.c
void matmul(float* xout, float* x, float* w, int n, int d) {
    // W (d,n) @ x (n,) -> xout (d,)
    for (i = 0; i < d; i++) {
        float val = 0.0f;
        for (int j = 0; j < n; j++) {
            val += w[i * n + j] * x[j];
        }
        xout[i] = val;
    }
}

// runq.c
void matmul(float* xout, QuantizedTensor *x, QuantizedTensor *w, int n, int d) {
    // W (d,n) @ x (n,) -> xout (d,)
    for (i = 0; i < d; i++) {
        float val = 0.0f;
        int32_t ival = 0;
        int in = i * n;
        for (int j = 0; j <= n - GS; j += GS) {
            for (int k = 0; k < GS; k++) {
                ival += ((int32_t) x->q[j + k]) * ((int32_t) w->q[in + j + k]);
            }
            val += ((float) ival) * w->s[(in + j) / GS] * x->s[j / GS];
            ival = 0;
        }
        xout[i] = val;
    }
}
```
run.c는 float 곱셈 결과를 모두 더하는 naive 구현이고, runq.c는 `GS` 크기(여기서는 `group_size=64`)만큼 점프하면서 int8을 int32로 변환한 곱셈 결과에 scale을 곱한 값을 더해나간다. runq.c의 연산이 더 많지만 naive float 곱셈보다 임베딩 값인 `x`도 quantized value인 int32 곱셈이 더 빠르다. 참고로 PyTorch doesn’t allow INT8 matrix multiplication by default.

DGX 실행 결과:  
```shell
$ OMP_NUM_THREADS=16 ./runq llama2_7b_q80.bin -i "Once upon a time" -n 10 -t 0.0
```
- `float` 연산: 9.336100 tokens/s
- `int32` 연산: 11.658031 tokens/s

# OpenMP
`$ make runomp`로 DGX에서 openmp로 128코어를 모두 사용할 수 있다.

<img src="/images/2024/runc-multithreads.png" width="70%">

M1:
```shell
# brew install llvm libomp
# echo 'export PATH="/opt/homebrew/opt/llvm/bin:$PATH"' >> ~/.zshrc
# CC = clang  # Makefile
# make runomp
#   clang -Ofast -fopenmp -march=native runq.c -lm -o runq
$ OMP_NUM_THREADS=10 ./runq llama2_7b_q80.bin -i "Once upon a time" -n 10 -t 0.0
```

# 실행 속도 정리
## llama2-7b.bin
llama2-7b FP32(25G) / Q8(6.7G), DGX: gcc 11.4.0, M1: clang 17.0.6  
```shell
$ ./run llama2_7b.bin -i "Once upon a time" -n 10 -t 0.0
```

| machine  | build        | run                 | quantized | tokens/s  |
| ------   | ------------ | ------------------- | --------- | --------- |
| DGX      | make         |                     |           | 0.163970  |
|          | make runfast |                     |           | 0.479004  |
|          | make runomp  | OMP_NUM_THREADS=1   |           | 0.675068  |
|          | make runomp  | OMP_NUM_THREADS=64  |           | 4.782147  |
|          | make runomp  | OMP_NUM_THREADS=128 |           | 4.668050  |
|          | make         |                     | o         | 0.763294  |
|          | make runfast |                     | o         | 0.741229  |
|          | make runomp  | OMP_NUM_THREADS=1   | o         | 0.787195  |
|          | make runomp  | OMP_NUM_THREADS=64  | o         | 14.446228 |
|          | make runomp  | OMP_NUM_THREADS=128 | o         | 13.975155 |
| M1 / 16G | make         |                     |           | 0.017468  |
|          | make runfast |                     |           | 0.017962  |
|          | make runomp  | OMP_NUM_THREADS=1   |           | 0.018126  |
|          | make runomp  | OMP_NUM_THREADS=10  |           | 0.044429  |
|          | make         |                     | o         | 2.241594  |
|          | make runfast |                     | o         | 2.380323  |
|          | make runomp  | OMP_NUM_THREADS=1   | o         | 2.357873  |
|          | make runomp  | OMP_NUM_THREADS=10  | o         | 8.620690  |

M1에서 매우 느린 이유는 모델이 메모리에 전부 올라가지 않아서로 추정된다. Llama 2와 일치하는 더 작은 모델을 구할 수가 없다. 

## stories110M.bin
Karpathy가 미리 빌드한 별도의 작은 모델로 속도만 따로 측정:  
```shell
$ ./run stories110M.bin -i "Once upon a time" -n 10 -t 0.0
```

| machine  | build        | run                 | tokens/s   |
| ------   | ------------ | ------------------- | ---------- |
| DGX      | make         |                     | 10.089686  |
|          | make runfast |                     | 28.938907  |
|          | make runomp  | OMP_NUM_THREADS=1   | 38.961039  |
|          | make runomp  | OMP_NUM_THREADS=64  | 204.545455 |
|          | make runomp  | OMP_NUM_THREADS=128 | 145.161290 |
| M1 / 16G | make         |                     | 11.335013  |
|          | make runfast |                     | 90.909091  |
|          | make runomp  | OMP_NUM_THREADS=1   | 90.909091  |
|          | make runomp  | OMP_NUM_THREADS=10  | 118.421053 |

## CUDA
```shell
$ ./runcuda stories110M.bin -i "Once upon a time" -n 128 -t 0
```

| machine  | build        | run                 | tokens/s   |
| ------   | ------------ | ------------------- | ---------- |
| DGX      | make runomp  | OMP_NUM_THREADS=1   | 32.160041  |
|          | make runomp  | OMP_NUM_THREADS=64  | 200.949367 |
|          | make runomp  | OMP_NUM_THREADS=128 | 145.642202 |
| CUDA[^fn-ankan]     | `nvcc llama2.cu -o runcu` |                     | 522.205207 |
| CUDA[^fn-cuda] / naive   | make runcuda |             | 604.761905 |
| CUDA[^fn-cuda] / cuBLAS  | make runcuda |             | 774.390244 |

[^fn-ankan]: <https://github.com/ankan-ban/llama2.cu>
[^fn-cuda]: <https://github.com/rogerallen/llama2.cu>

naive 구현에서 `sum`을 float이 아닌 half로 `output[index]`에 할당하면 cuBLAS와 거의 근사할 정도로 빠르다. half는 CUDA에서 지원하는 fp16이다. 당연히 float과 결과가 약간 다르다. 128토큰에서 마지막 4~5토큰 정도가 다르게 생성됐다.

make runomp는 CPU로 실행됐을 것이며 naive 구현은 자료형을 half로 바꾼 정도의 차이만 있었을 것이다. (*Jun 2024*)

# M1에서 clang 빌드

```shell
$ clang --version
Homebrew clang version 17.0.6
Target: arm64-apple-darwin23.3.0
Thread model: posix
InstalledDir: /opt/homebrew/opt/llvm/bin

$ /usr/bin/gcc --version
Apple clang version 14.0.3 (clang-1403.0.22.14.1)
Target: arm64-apple-darwin23.3.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```

# convert.py

hf 모델은 convert가 되지 않았다.
```shell
$ python export.py llama2_7b_hf.bin --version 0 --hf /Users/xxx/workspace/llama-2/Llama-2-7b-hf
[1]    88371 killed     python export.py llama2_7b_hf.bin --version 0 --hf
/opt/homebrew/anaconda3/envs/python310/lib/python3.10/multiprocessing/resource_tracker.py:224: UserWarning: resource_tracker: There appear to be 1 leaked semaphore objects to clean up at shutdown
  warnings.warn('resource_tracker: There appear to be %d '
```

또한 `--version 1`로 변환시 실행이 되지 않는다.
```shell
$ ./run llama2_7b_hf.bin -i "Once upon a time" -n 10
[1]    93242 segmentation fault  ./run llama2_7b_hf.bin -i "Once upon a time" -n 10
```

스크립트[^fn-conv]를 이용해 pth로 변환하고 `--version 0`으로 진행. quantized는 2로 진행했다.

[^fn-conv]: <https://gist.github.com/ahoho/57f5c3dcdce4be522ca13dfb96cc1eb8>