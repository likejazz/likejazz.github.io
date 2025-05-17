---
layout: wiki
title: vLLM
tags: ["Large Language Model (LLM)"]
last_modified_at: 2025/03/14 19:08:05
---

- [실행](#실행)
  - [서버](#서버)
    - [curl](#curl)
  - [Code](#code)
  - [응용](#응용)
- [Multi Nodes](#multi-nodes)

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

# Multi Nodes
`run_cluster.sh`를 제공하며, ray를 사용하고 docker로 구동한다. k8s내에서는 해당 docker 이미지를 배포하는 방식으로 적용이 가능할 거 같다. 스크립트에는 docker가 구동되자마자 ray를 실행하도록 되어 있다.