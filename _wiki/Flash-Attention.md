---
layout: wiki 
title: Flash Attention
tags: ["Transformer"]
last_modified_at: 2024/10/29 18:50:38
---

- [softmax](#softmax)
  - [naive](#naive)
  - [safe softmax](#safe-softmax)
  - [safe softmax with online normalizer calculation](#safe-softmax-with-online-normalizer-calculation)
    - [online softmax](#online-softmax)
  - [tiling](#tiling)
- [Flash Attention (Tiling)](#flash-attention-tiling)
- [C++, Torch Profiling](#c-torch-profiling)

# softmax
## naive
$$\sigma(z_i) = \frac{e^{z_{i}}}{\sum_{j=1}^K e^{z_{j}}} \ \ \ for\ i=1,2,\dots,K$$

<img src="/images/2024/Screenshot 2024-07-12 at 5.06.08 PM.png" width="250">
[^fn-softmax_to_attention]

[^fn-softmax_to_attention]: <https://courses.cs.washington.edu/courses/cse599m/23sp/notes/flashattn.pdf>

## safe softmax
It involves **subtracting the maximum value from all the output values** before applying the softmax equation. This step helps to prevent the exponent from becoming extremely large, which could lead to computational challenges or overflow errors.
```
>>> np.exp(0.5)
np.float64(1.6487212707001282)
>>> np.exp(1)
np.float64(2.718281828459045)
>>> np.exp(10)
np.float64(22026.465794806718)
```

```python
np.exp(x - max_x) / sum(np.exp(x - max_x))
```

<img src="/images/2024/Screenshot 2024-07-12 at 4.44.23 PM.png" width="250">

[^fn-onlinesoftmax]

[^fn-onlinesoftmax]: [Online normalizer calculation for softmax](https://arxiv.org/abs/1805.02867)

## safe softmax with online normalizer calculation

<img src="/images/2024/Screenshot 2024-07-12 at 4.48.15 PM.png" width="450">

Essentially, the algorithm keeps the maximum value $$m$$ and the normalization term $$d$$ as it iterates over elements of the input array. At each iteration it needs to adjust the normalizer $$d$$ to the new maximum $$m_j$$ and only then add new value to the normalizer.

### online softmax
```python
# loop 1: get the maximum value of x and the accumulated exponential values
max_x = -np.inf
accum_exp = 0.
for t in x:
    max_x_new = t if t > max_x else max_x
    accum_exp = np.exp(max_x - max_x_new) * accum_exp + np.exp(t - max_x_new)
    max_x = max_x_new

# loop 2: get the softmax output by dividing the exponential of `x-max(x)` with `accum_exp`
out = [0. for _ in range(len(x))]
for i, t in enumerate(x):
    out[i] = np.exp(t - max_x) / accum_exp
```

## tiling
```python
>>> a = np.array([0.1, 0.5, 0.4, 0.2, 0.3, 0.3])
>>> sum([np.exp(aa - 0.5) for aa in a])
4.9534
>>> l1 = np.exp(0.1 - 0.5) + np.exp(0.5 - 0.5) + np.exp(0.4 - 0.5)
2.5751
>>> l2 = np.exp(0.2 - 0.3) + np.exp(0.3 - 0.3) + np.exp(0.3 - 0.3)
2.9048
>>> np.exp(0.5 - 0.5) * l1 + np.exp(0.3 - 0.5) * l2
4.9534
```

# Flash Attention (Tiling)
<img src="/images/2024/Screenshot 2024-07-13 at 12.57.30 AM.jpg" width="600">

# C++, Torch Profiling
```python
fanano = load(name='fanano', sources=['fanano.cpp', 'fanano.cu'], verbose=True)
```
빌드시 `undefined symbol` 오류는 argument type이 잘못되어 있는 경우이므로 주의

프로파일링 시 레거시인 `torch.autograd.profiler.profile`와 `torch.profiler.profile`는 결과가 다르므로 주의, 특히 CUDA 실행 시간이 많이 차이가 난다.