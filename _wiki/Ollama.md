---
layout: wiki 
title: Ollama
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/03/07 17:06:37
---

<!-- TOC -->

- [설치](#설치)
- [실행](#실행)
  - [42dot LLM Modelfile](#42dot-llm-modelfile)

<!-- /TOC -->

# 설치
```shell
$ sudo curl -L https://ollama.com/download/ollama-linux-amd64 -o /usr/bin/ollama
$ sudo chmod +x /usr/bin/ollama
```
[릴리즈](https://github.com/ollama/ollama/releases)에서 AMD64 최신 버전으로 경로 변경

# 실행
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

## 42dot LLM Modelfile
다음과 같이 1번째 줄을 바꿔서 생성했다.  
`$ sed -i '1s|.*|FROM /models/42dot_LLM-SFT-1.3B-gguf/ggml-model-Q4_0.gguf|' Modelfile && ollama create 42dot:1.3b-q4_0 -f Modelfile`

```
FROM /home/sshuser/.ollama/models/blobs/sha256:xxxx
TEMPLATE """{\{ .System }\}
<human>: {\{ .Prompt }\}
<bot>:
"""
SYSTEM """호기심 많은 인간 (human)과 인공지능 봇 (AI bot)의 대화입니다. \n봇의 이름은 42dot LLM이고 포티투닷 (42dot)에서 개발했습니다. \n봇은 인간의 질문에 대해 친절하게 유용하고 상세한 답변을 제공합니다.
"""
PARAMETER stop "<human>:"
PARAMETER stop "<bot>:"
```
중간에 `\`는 제거
