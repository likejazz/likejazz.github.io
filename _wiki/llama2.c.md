---
layout: wiki 
title: llama2.c
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/02/25 02:20:05
---

<!-- TOC -->

- [실행](#실행)
- [Quantization](#quantization)
- [OpenMP](#openmp)
- [실행 속도 정리](#실행-속도-정리)
- [clang](#clang)

<!-- /TOC -->

# 실행
M1 Pro:
```
$ ./run stories15M.bin
Once upon a time, there was a little girl named Lily. She loved to play with her toys, especially her teddy bear. One day, Lily's teddy bear's paw got hurt while playing outside.
Lily's mom took her to see the doctor. The doctor was very nice and he gave Lily a lollipop. The doctor said, "Lily, you need to give this to your teddy bear to make him feel better."
Lily went home and gave her teddy bear the lollipop. Her mom said, "Oh no, you lost your teddy bear. We can go back to the doctor and get him fixed."
Lily was happy that she made her teddy bear feel better. She went to the doctor and he fixed her teddy bear's paw. Lily was very careful and always made sure her teddy bear was okay. The end.
achieved tok/s: 112.314709
```
DGX Station A100:  
achieved tok/s: 72.649573  
128개 코어 중 하나만 사용하며 M1에 비해 늦다.

# Quantization
DGX:
```
$ OMP_NUM_THREADS=1 ./runq llama2_7b_q80.bin -i "Once upon a time" -n 10
Once upon a time I drove home, down high
achieved tok/s: 0.787195

$ OMP_NUM_THREADS=128 ./runq llama2_7b_q80.bin -i "Once upon a time" -n 10
Once upon a time I worked in a call center
achieved tok/s: 13.657056
```
quantized는 multithreads로 실행할 때 일반 모델(`achieved tok/s: 4.534005`)에 비해 3x ↑

# OpenMP
DGX에서는 `$ make runomp`로 openmp를 사용해 멀티쓰레딩으로 실행가능하다.

<img src="/images/2024/runc-multithreads.png" width="70%">

```
$ OMP_NUM_THREADS=1 ./run llama2_7b.bin -i "Once upon a time" -n 10
Once upon a time the gods fought each other for
achieved tok/s: 0.675068

$ OMP_NUM_THREADS=128 ./run llama2_7b.bin -i "Once upon a time" -n 10
Once upon a time there lived in a small house
achieved tok/s: 4.534005
```

M1:
```shell
# brew install llvm libomp
# echo 'export PATH="/opt/homebrew/opt/llvm/bin:$PATH"' >> ~/.zshrc
# CC = clang  # Makefile
# make runomp
#   clang -Ofast -fopenmp -march=native runq.c -lm -o runq
$ ./runq llama2_7b_q80.bin -i "Once upon a time" -n 10
```

# 실행 속도 정리

llama2-7b, DGX: gcc 11.4.0, M1: clang 17.0.6  
`$ ./run llama2_7b.bin -i "Once upon a time" -n 10`

| machine | build        | run                 | quantized | tokens/s  |
| ------  | ------------ | ------------------- | --------- | --------- |
| DGX     | make         |                     |           | 0.163970  |
|         | make runfast |                     |           | 0.479004  |
|         | make runomp  | OMP_NUM_THREADS=1   |           | 0.675068  |
|         | make runomp  | OMP_NUM_THREADS=64  |           | 4.782147  |
|         | make runomp  | OMP_NUM_THREADS=128 |           | 4.668050  |
|         | make         |                     | o         | 0.763294  |
|         | make runfast |                     | o         | 0.741229  |
|         | make runomp  | OMP_NUM_THREADS=1   | o         | 0.787195  |
|         | make runomp  | OMP_NUM_THREADS=64  | o         | 14.446228 |
|         | make runomp  | OMP_NUM_THREADS=128 | o         | 13.975155 |
| M1      | make         |                     |           | 0.017468  |
|         | make runfast |                     |           | 0.017962  |
|         | make runomp  | OMP_NUM_THREADS=1   |           | 0.018126  |
|         | make runomp  | OMP_NUM_THREADS=10  |           | 0.044429  |
|         | make         |                     | o         | 2.241594  |
|         | make runfast |                     | o         | 2.380323  |
|         | make runomp  | OMP_NUM_THREADS=1   | o         | 2.357873  |
|         | make runomp  | OMP_NUM_THREADS=10  | o         | 8.620690  |

# clang
```
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