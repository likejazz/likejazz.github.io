---
layout: wiki
title: llama.cpp
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/05/29 12:50:15
---

<!-- TOC -->

- [빌드](#빌드)
- [실행](#실행)
  - [n-gpu-layers](#n-gpu-layers)
  - [llama-cpp-python](#llama-cpp-python)
- [바인딩 빌드](#바인딩-빌드)

<!-- /TOC -->

# 빌드

macOS:

```shell
$ CC=/opt/homebrew/opt/llvm/bin/clang \
CXX=/opt/homebrew/opt/llvm/bin/clang++ \
make
```

cuBLAS:
`$ make LLAMA_CUBLAS=1`

Jetson:

```shell
$ cmake .. -DLLAMA_CUDA=on -DLLAMA_CUDA_F16=1 -DCMAKE_CUDA_ARCHITECTURES=87
$ cmake --build . --config Release --parallel 8
```

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

## n-gpu-layers
메모리에 올라갈 사이즈가 맞지 않으면 core dumped가 발생한다.
```
$ ./ollama_llama_server --model /root/.ollama/models/blobs/sha256-xxxx --ctx-size 2048 --batch-size 512 --embedding --log-disable --n-gpu-layers 50 --verbose --parallel 1 --port 44871
...
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 22953.12 MiB on device 0: cudaMalloc failed: out of memory
llama_model_load: error loading model: unable to allocate backend buffer
llama_load_model_from_file: exception loading model
terminate called after throwing an instance of 'std::runtime_error'
  what():  unable to allocate backend buffer
Aborted (core dumped)
```

## llama-cpp-python

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

벤치마크 [GPU Comparison: Jetson Orin, RTX 4080 SUPER](https://datalore.jetbrains.com/)

# 바인딩 빌드

맥에서 [Go](https://github.com/go-skynet/go-llama.cpp) 빌드시 기본 빌드는 `Object file was built for newer OSX version than being linked`오류 발생하여 다음과 같이 metal 빌드:

```shell
$ git clone --recurse-submodules https://github.com/go-skynet/go-llama.cpp
$ BUILD_TYPE=metal make libbinding.a
$ CGO_LDFLAGS="-framework Foundation -framework Metal -framework MetalKit -framework MetalPerformanceShaders" LIBRARY_PATH=$PWD C_INCLUDE_PATH=$PWD go build ./examples/main.go
$ cp build/bin/ggml-metal.metal .
$ ./main -m "~/workspace/models/gguf/gemma-1.1-2b-it.q5_K_M.gguf" -t 1 -ngl 1
```

직접 실행:

```shell
$ LIBRARY_PATH=$PWD C_INCLUDE_PATH=$PWD go run ./examples -m "~/workspace/models/gguf/gemma-1.1-2b-it.q5_K_M.gguf" -t 14
```

gemma를 지원하지 않아 llama.cpp 버전을 올리니 `xcrun: error: unable to find utility "metal", not a developer tool or in PATH`오류 발생. xcode를 설치하고 `$ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer` 설정했다. 그럼에도 빌드가 마무리 되지 못하고 마지막에 에러가 발생한다.

최신 CC, CXX:

```shell
$ CC=/opt/homebrew/opt/llvm/bin/clang \
CXX=/opt/homebrew/opt/llvm/bin/clang++ \
BULD_TYPE=metal make libbinding.a
```

Linux CUDA:
```shell
$ BUILD_TYPE=cublas make libbinding.a
```
최신 버전에서는 오류가 발생한다.
