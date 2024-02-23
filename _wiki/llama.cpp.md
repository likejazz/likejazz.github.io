---
layout: wiki 
title: llama.cpp
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/02/23 14:15:51
---

<!-- TOC -->

- [cuBLAS 빌드](#cublas-빌드)
- [실행](#실행)
- [ollama](#ollama)
- [llama-cpp-python](#llama-cpp-python)

<!-- /TOC -->

# cuBLAS 빌드
`$ make LLAMA_CUBLAS=1`

# 실행
```shell
$ CUDA_VISIBLE_DEVICES=2 ./main -m /models/ggml-model-q4_0.gguf \
--ctx-size 4096 \
--keep -1 \
--temp 0.5 \
--top_p 0.95 \
--top_k 20 \
--n-predict 512 \
--repeat-penalty 1.2 \
--escape \
--prompt "호기심 많은 인간 (human)과 인공지능 봇 (AI bot)의 대화입니다. \n봇의 이름은 42dot LLM이고 포티투닷 (42dot)에서 개발했습니다. \n봇은 인간의 질문에 대해 친절하게 유용하고 상세한 답변을 제공합니다. \n" \
--in-prefix "<human>: " \
--in-suffix "<bot>:" \
--interactive-first \
--n-gpu-layers 15000
```
`--n-gpu-layers` 옵션으로 GPU에 레이어를 올려 구동한다.

서버 모드:
```shell
$ CUDA_VISIBLE_DEVICES=2 ./server -m /models/ggml-model-f32.gguf \
--host 0.0.0.0 \
--port 8888 \
--verbose
```

웹 페이지에서 Show Probabilities를 주면 확률 분포를 볼 수 있다.

# ollama
`--verbose`로 토큰 속도를 볼 수 있다.

```shell
$ curl -X POST http://localhost:11434/api/generate \
-H "Content-Type: application/json" \
-d '{
    "model": "codellama:70b",
    "prompt": "write a function about fibonacci sequence in rust",
    "stream": false
}'
```

42dot LLM Modelfile
```
FROM /home/sshuser/.ollama/models/blobs/sha256:xxxx
TEMPLATE """{{ .System }}
<human>: {{ .Prompt }}
<bot>:
"""
SYSTEM """호기심 많은 인간 (human)과 인공지능 봇 (AI bot)의 대화입니다. \n봇의 이름은 42dot LLM이고 포티투닷 (42dot)에서 개발했습니다. \n봇은 인간의 질문에 대해 친절하게 유용하고 상세한 답변을 제공합니다.
"""
PARAMETER stop "<human>:"
PARAMETER stop "<bot>:"
```

# llama-cpp-python
```python
llm = Llama(
    model_path='/models/ggml-model-f32.gguf',
    n_gpu_layers=-1,
    split_mode=llama_cpp.LLAMA_SPLIT_NONE,
    logits_all=True,
)

output = llm(
    prompt,
    max_tokens=1,
    temperature=0.5,
    top_p=0.95,
    top_k=20,
    repeat_penalty=1.2,
    logprobs=7,
)
```
`logits_all=True`로 읽어들여야 `logprobs`를 출력할 수 있다.