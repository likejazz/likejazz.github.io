---
layout: wiki 
title: llama2.c
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/02/23 18:39:36
---

<!-- TOC -->

- [속도](#속도)
- [Quantization](#quantization)

<!-- /TOC -->

# 속도
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
128개 코어 중 하나만 사용하므로 M1에 비해 늦다.

DGX에서는 `$ make runomp`로 openmp를 사용해 멀티쓰레딩으로 실행가능하다.

<img src="/images/2024/runc-multithreads.png" width="70%">

# Quantization