---
layout: wiki 
title: Ollama
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/04/29 17:09:12
---

<!-- TOC -->

- [설치](#설치)
- [실행](#실행)
  - [42dot LLM Modelfile](#42dot-llm-modelfile)
- [바인딩](#바인딩)
  - [ext\_server](#ext_server)
- [개발](#개발)

<!-- /TOC -->

# 설치
```shell
$ sudo curl -L https://ollama.com/download/ollama-linux-amd64 -o /usr/bin/ollama
$ sudo chmod +x /usr/bin/ollama
```
[릴리즈](https://github.com/ollama/ollama/releases)에서 amd64 최신 버전으로 경로 변경

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

ollama-python을 이용할 때 동일 프롬프트에 대해 `prompt_eval_duration`이 2번째 부터 빠르게 나온다. 캐싱하는 것으로 추정. 캐싱되는 결과에는 `prompt_eval_count` 값이 나오지 않는다.

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

# 바인딩

- `CMakeLists.txt` 맨 하단에 `add_subdirectory(../ext_server ext_server) # ollama`를 추가하여 ext_server가 같이 빌드되도록 처리
- 모델 로딩 에러가 발생할 때 throw 하도록 패치
- Go 바이너리에 링킹을 위한 `libllama.a` static build. create시 quantization을 위해서만 사용된다.

```
# /xxx/ollama/llm/build/linux/x86_64/cuda_v12/bin
$ ls -al
total 581856
drwxr-xr-x  2 root root      4096 Apr 24 09:44 .
drwxr-xr-x 10 root root      4096 Apr 24 09:44 ..
-rw-r--r--  1 root root 109760416 Apr 24 09:44 libcublas.so.12
-rw-r--r--  1 root root 441131728 Apr 24 09:44 libcublasLt.so.12
-rw-r--r--  1 root root    707904 Apr 24 09:44 libcudart.so.12
-rwxr-xr-x  1 root root  44200752 Apr 24 09:44 ollama_llama_server
$ ldd ollama_llama_server
	linux-vdso.so.1 (0x00007fffa0d62000)
	libcudart.so.12 => /usr/local/cuda/lib64/libcudart.so.12 (0x00007f91dce00000)
	libcublas.so.12 => /usr/local/cuda/lib64/libcublas.so.12 (0x00007f91d6200000)
	libcuda.so.1 => /usr/lib/x86_64-linux-gnu/libcuda.so.1 (0x00007f91d4685000)
	libstdc++.so.6 => /usr/lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f91d4459000)
	libm.so.6 => /usr/lib/x86_64-linux-gnu/libm.so.6 (0x00007f91dd1b6000)
	libgcc_s.so.1 => /usr/lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f91dd194000)
	libc.so.6 => /usr/lib/x86_64-linux-gnu/libc.so.6 (0x00007f91d4230000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f91dfd24000)
	libdl.so.2 => /usr/lib/x86_64-linux-gnu/libdl.so.2 (0x00007f91dd18f000)
	libpthread.so.0 => /usr/lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f91dd18a000)
	librt.so.1 => /usr/lib/x86_64-linux-gnu/librt.so.1 (0x00007f91dd185000)
	libcublasLt.so.12 => /usr/local/cuda/lib64/libcublasLt.so.12 (0x00007f91b6200000)
```
- C++ 빌드로 ollama_llama_server를 만드는 구조

## ext_server
Starting llama.cpp server:
```shell
$ /var/folders/_z/xxx/T/ollamaxxx/runners/metal/ollama_llama_server \
--model /Users/sangpark/.ollama/models/blobs/sha256-xxx \
--ctx-size 2048 \
--batch-size 512 \
--embedding \
--log-format json \
--n-gpu-layers 19 \
--verbose \
--port 52709
```

Http server listening(llama.cpp):
```shell
$ curl --request POST \
    --url http://localhost:52709/completion \
    --header "Content-Type: application/json" \
    --data '{
        "cache_prompt": true,
        "prompt": "<start_of_turn>user\nhi hello<end_of_turn>\n<start_of_turn>model\n",
        "n_predict": 64,
        "stop":["<start_of_turn>","<end_of_turn>"],
        "stream": true
    }'
```

Ollama serve:
```shell
$ curl http://localhost:11434/api/generate -d '{
  "model": "gemma:2b",
  "prompt": "Why is the sky blue?"
}'

$ curl http://localhost:11434/api/chat -d '{
  "model": "gemma:2b",
  "messages": [
    {
      "role": "user",
      "content": "why is the sky blue?"
    }
  ]
}'
```

# 개발
```
$ XXX_SKIP_GENERATE=1 ./runme.sh && OLLAMA_KEEP_ALIVE=1m OLLAMA_DEBUG=1 ./ollama-darwin serve
```

`OLLAMA_KEEP_ALIVE`에 모델 메모리 유지 시간을 설정한다. 이후 타이머가 동작해 `unload()` 실행.