---
layout: wiki
title: vLLM
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/09/22 13:45:48
---

- [설치](#설치)
- [실행](#실행)
  - [서버](#서버)
  - [Code](#code)
  - [응용](#응용)

# 설치
기존에는 직접 설치한게 아니라 [(비공개) lm-eval](/wiki/lm-eval)에서 자동으로 실행했다.

`pip install vllm`

# 실행
## 서버
```
$ vllm serve /models/xxx --served-model-name xxx
```

## Code
```python
from vllm import LLM
llm = LLM(model="/models/xxx")
output = llm.generate('안녕하세요')
```

## 응용
```python
from vllm import LLM, SamplingParams
llm = LLM(model='.')
llm.generate(prompt, SamplingParams(temperature=0.0))[0].outputs
```