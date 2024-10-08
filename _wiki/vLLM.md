---
layout: wiki
title: vLLM
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/10/08 15:33:29
---

- [설치](#설치)
- [실행](#실행)
  - [서버](#서버)
    - [curl](#curl)
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

context length가 긴 경우 모델 로딩시 OOM이 발생할 수 있다.
```
$ vllm serve aa/bb --max-model-len 8192
```

### curl
```
$ curl http://localhost:8000/v1/chat/completions -i \
  -H "Content-Type: application/json" \
  -d '{
     "model": "aa/bb",
     "stream": true,
     "max_tokens": 512,
     "frequency_penalty": 1.5,
     "messages": [{"role": "user", "content": "우리나라 대통령이 누구야?"}]
   }'
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
