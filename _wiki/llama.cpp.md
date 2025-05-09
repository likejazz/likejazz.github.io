---
layout: wiki
title: llama.cpp
tags: ["llama.cpp"]
last_modified_at: 2024/12/17 12:21:17
---

<!-- TOC -->

- [빌드](#빌드)
  - [disable AVX, CUDA](#disable-avx-cuda)
- [실행](#실행)
  - [CPU 실행](#cpu-실행)
- [구조](#구조)
- [바인딩](#바인딩)
  - [llama-cpp-python](#llama-cpp-python)
  - [Go 바인딩](#go-바인딩)

<!-- /TOC -->

# 빌드

macOS:

```shell
$ brew install llvm

# For compilers to find llvm
export CC=/opt/homebrew/opt/llvm/bin/clang
export CXX=/opt/homebrew/opt/llvm/bin/clang++
export LDFLAGS="-L/opt/homebrew/opt/llvm/lib"
export CPPFLAGS="-I/opt/homebrew/opt/llvm/include"
export PATH="/opt/homebrew/opt/llvm/bin:$PATH"
```

```shell
$ cmake -B bld && \
  cd bld && \
  make -j llama-cli llama-server
```
gmonon으로 측정 결과 M2 MacBook Air에서 full build는 33s 소요

cuBLAS:  
```shell
$ cmake -B bld -DGGML_CUDA=ON && \
  cd bld && \
  make -j llama-cli llama-server
```
그 전에 `$ apt install ccache`로 컴파일러 캐시 설치 가능. gnomon으로 측정 결과 sgemm.cpp 빌드에 168s, 전체 172s 소요.

or  
`$ make GGML_CUDA=1 llama-cli`  

Jetson:

```shell
$ cmake .. -DGGML_CUDA=on -DGGML_CUDA_F16=1 -DCMAKE_CUDA_ARCHITECTURES=87
$ cmake --build . --config Release --parallel 8
```

## disable AVX, CUDA
```shell
$ cmake .. -DGGML_CUDA=off -DGGML_AVX=off -DGGML_AVX2=off -DGGML_F16C=off -DGGML_FMA=off
$ make -j llama-server
```
or
```shell
$ cmake -B bld -DGGML_CUDA=off -DGGML_AVX=off -DGGML_AVX2=off -DGGML_F16C=off -DGGML_FMA=off
$ cmake --build bld --config Release -j -t llama-server
```
# 실행

```shell
$ ./llama-cli -m ~/workspace/models/gguf/qwen2-0_5b-instruct-fp16.gguf \
--gpu-layers 25 \
--color \
--prompt "<|im_start|>user\n왜 하늘은 푸른가?<|im_end|>\n<|im_start|>assistant\n"
```

서버 모드:

```shell
$ ./llama-server -m /models/ggml-model-f32.gguf \
--host 0.0.0.0 \
--port 8888 \
--verbose
```

웹 페이지에서 Show Probabilities를 주면 확률 분포를 볼 수 있다.

## CPU 실행
1. `--gpu-layers` 옵션을 주지 않고 실행해 모든 레이어가 CPU에 있는데도 llama-server는 GPU 메모리를 점유하며 심지어 Prefill Latency는 GPU에서 계산을 진행한다.
1. AVX를 끄도록 옵션을 부여하고 빌드했음에도 llama-server 실행시에는 항상 AVX=1로 표시된다. 그리고 옵션에 따른 속도차이가 없다.

# 구조
(비공개) [#1-7 ggml-graph](/wiki/Private-Links) 코드 - `bld`에서 `make ggml-graph`로 맥에서 빌드:
```
$ make ggml-graph
[ 12%] Generate assembly for embedded Metal library
Embedding Metal library
[ 25%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml.c.o
[ 25%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml-alloc.c.o
[ 37%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml-backend.c.o
[ 37%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml-quants.c.o
[ 50%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml-metal.m.o
[ 62%] Building ASM object ggml/src/CMakeFiles/ggml.dir/__/__/autogenerated/ggml-metal-embed.s.o
[ 62%] Building CXX object ggml/src/CMakeFiles/ggml.dir/ggml-blas.cpp.o
[ 75%] Building CXX object ggml/src/CMakeFiles/ggml.dir/sgemm.cpp.o
[ 75%] Linking CXX shared library libggml.dylib
[ 75%] Built target ggml
[ 87%] Building CXX object examples/ggml-graph/CMakeFiles/ggml-graph.dir/ggml-graph.cpp.o
[100%] Linking CXX executable ../../bin/ggml-graph
[100%] Built target ggml-graph
```

<img src="/images/2024/Screenshot 2024-07-05 at 4.36.20 PM.png" width="50%">

CUDA 빌드는 다음과 같다. default off 이므로 cmake 실행시 반드시 지정 필요 `cmake -B bld -DGGML_CUDA=ON`
```
$ make -j ggml-graph
[  3%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml-alloc.c.o
[  6%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/arange.cu.o
[ 10%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml-quants.c.o
[ 10%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml-backend.c.o
[ 10%] Building C object ggml/src/CMakeFiles/ggml.dir/ggml.c.o
[ 10%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/acc.cu.o
[ 17%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/binbcast.cu.o
...
[ 96%] Building CXX object ggml/src/CMakeFiles/ggml.dir/sgemm.cpp.o
[ 96%] Linking CXX shared library libggml.so
[ 96%] Built target ggml
[100%] Building CXX object examples/ggml-graph/CMakeFiles/ggml-graph.dir/ggml-graph.cpp.o
[100%] Linking CXX executable ../../bin/ggml-graph
[100%] Built target ggml-graph
```

`ggml_graph_compute()`를 사용해 계산하며 thread 함수에서 `ggml_compute_forward()`를 호출하고 각 노드를 dfs로 실행한다. 계산 결과로 `data` 포인터를 결과로 채운다.

디버거 실행을 위해서는 처음 cmake 실행시 `$ cmake -B bld -DCMAKE_BUILD_TYPE=Debug` 디버그로 빌드하지 않으면 lldb에서 코드가 아니라 어셈블리 코드만 나오며 라인으로 breakpoint를 걸 수 없다.

작년까지는 계산 함수가 동일했으나 업데이트 되면서 CUDA, Metal등을 사용하려면 `ggml_backend_graph_compute()` 별도 함수를 호출한다.

# 바인딩
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

(비공개) 벤치마크 [#2-2 GPU Comparison: Jetson Orin, RTX 4080 SUPER](/wiki/Private-Links)

## Go 바인딩

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
BUILD_TYPE=metal make libbinding.a
```

Linux CUDA:
```shell
$ BUILD_TYPE=cublas make libbinding.a
```
최신 버전에서는 오류가 발생한다.
