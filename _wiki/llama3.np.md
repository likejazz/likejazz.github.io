---
layout: wiki 
title: llama3.np
tags: ["Transformer"]
last_modified_at: 2024/10/29 18:57:35
---

- [속도](#속도)

# 속도
stories15M:
- M2 MacBook Air, CPU: 28 tokens/s
- 4080 SUPER, CPU(NumPy): 42 tokens/s
- 4080 SUPER, GPU(CuPy): 86 tokens/s

stories110M:
- M2 MacBook Air, CPU: 3 tokens/s
- 4080 SUPER, CPU(NumPy): 4 tokens/s
- 4080 SUPER, GPU(CuPy): 21 tokens/s

meta-llama/Meta-Llama-3-8B:  
npz 파일 크기가 15G에 달한다. GPU는 Out of Memory로 실행되지 않고, CPU는 속도 측정이 의미 없을 정도로 실행된다(약 0.1tok/s). 설정은 다음과 같다:
```python
dim: int                    = 4096
n_layers: int               = 32
n_heads: int                = 32
n_kv_heads: Optional[int]   = 8
max_seq_len: int            = 8192
```
kv가 $$\frac{1}{4}$$ 크기다.