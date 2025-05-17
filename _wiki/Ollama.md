---
layout: wiki 
title: Ollama
tags: ["llama.cpp"]
last_modified_at: 2025/04/09 01:07:56
---

<!-- TOC -->

- [42dot LLM Modelfile](#42dot-llm-modelfile)
- [바인딩](#바인딩)
  - [ollama\_llama\_server](#ollama_llama_server)
  - [속도](#속도)
    - [H100 속도](#h100-속도)
- [개발](#개발)
  - [gpu-layers](#gpu-layers)
- [Internals](#internals)
  - [API 호출시 동작](#api-호출시-동작)
  - [llama.cpp 서버 구동 방식](#llamacpp-서버-구동-방식)
- [모델 생성](#모델-생성)
  - [양자화](#양자화)

<!-- /TOC -->

# 42dot LLM Modelfile
다음과 같이 1번째 줄을 바꿔서 생성했다.  
```shell
$ sed -i '1s|.*|FROM /models/42dot_LLM-SFT-1.3B-gguf/ggml-model-Q4_0.gguf|' Modelfile && ollama create 42dot:1.3b-q4_0 -f Modelfile
```

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
- Go 바이너리에 링킹을 위한 `libllama.a` static build. create시 quantization을 위해 사용된다.

```shell
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
- C++ 빌드로 ollama_llama_server를 만든다.

## ollama_llama_server
llama.cpp server:
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

llama.cpp server API:
```shell
$ curl -X POST \
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

ollama server API  
`/api/generate`:
```shell
$ curl http://localhost:11434/api/generate -d '{
  "model": "gemma:2b",
  "prompt": "Why is the sky blue?",
  "stream": false
}'
```

`/api/chat`:
```shell
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

ChatGPT API:
```shell
$ curl http://localhost:11434/v1/chat/completions -i \
  -H "Content-Type: application/json" \
  -d '{
     "model": "gemma:2b",
     "stream": true,
     "messages": [{"role": "user", "content": "우리나라 대통령이 누구야?"}]
   }'
```

## 속도

CLI에서 `--verbose`로 실행하면 토큰 통계를 볼 수 있다.

llama3, 4080 SUPER  
모델 로딩 포함 실행:
```
total duration:       2.67475173s
load duration:        1.579052174s
prompt eval count:    629 token(s)
prompt eval duration: 198.018ms
prompt eval rate:     3176.48 tokens/s
eval count:           85 token(s)
eval duration:        851.575ms
eval rate:            99.82 tokens/s
```

모델 로딩 된 상태에서 실행:
```
total duration:       1.602803935s
load duration:        634.816µs
prompt eval count:    629 token(s)
prompt eval duration: 201.092ms
prompt eval rate:     3127.92 tokens/s
eval count:           130 token(s)
eval duration:        1.307583s
eval rate:            99.42 tokens/s
```

이미 실행했던 프롬프트를 다시 실행할 때 (앞 부분이 동일하면 뒷 부분부터 prompt eval count 계산):
```
total duration:       1.276829306s
load duration:        552.16µs
prompt eval count:    1 token(s)
prompt eval duration: 13.268ms
prompt eval rate:     75.37 tokens/s
eval count:           121 token(s)
eval duration:        1.215909s
eval rate:            99.51 tokens/s
```

바로 직전의 프롬프트만 캐싱하며, 이 경우 prompt eval rate는 낮게 나오므로 유의

### H100 속도
"why is the sky blue?"
- qwen2:72b-instruct-q8_0, 28 t/s
- qwen2:72b-instruct-q6_K, 28 t/s
- llama3:70b-instruct-q8_0, 29 t/s
- llama3:70b-instruct-q5_K_M, 34 t/s 
- llama3:8b(q4), 166 t/s
- gemma2:27b-instruct-fp16, 28 t/s

# 개발
```
$ XXX_SKIP_GENERATE=1 ./runme.sh && OLLAMA_KEEP_ALIVE=1m OLLAMA_DEBUG=1 ./ollama-darwin serve
```

`OLLAMA_KEEP_ALIVE`에 모델 메모리 유지 시간을 설정한다. 이후 타이머가 동작해 `unload()` 실행.

## gpu-layers
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

# Internals
## API 호출시 동작

로컬 모델 목록 조회:
```shell
$ curl http://localhost:11434/api/tags | jq
```
`routes.go`의 `ListModelsHandler`에서 `walkFunc`를 이용해 모든 하위 경로를 탐색한 다음 디렉토리가 아닌 경우 모델 파일을 읽어서 정보를 가져온다.

- 실제 경로: `~/.ollama/models/manifests/registry.ollama.ai/library/llama3/latest`

blob를 열어보면 다음과 같은 정보가 있다.
```shell
$ cat ~/.ollama/models/blobs/sha256-xxxx | jq
{
  "model_format": "gguf",
  "model_family": "llama",
  "model_families": [
    "llama"
  ],
  "model_type": "8B",
  "file_type": "Q4_0",
  "architecture": "amd64",
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:xxxx",
      "sha256:xxxx",
      "sha256:xxxx",
      "sha256:xxxx"
    ]
  }
}
```

모델 정보를 `gguf.go`에서 읽어들인다.

서버에서 모델 가져옴:

```shell
$ curl -X POST http://localhost:11434/api/pull \
-H "Content-Type: application/json" \
-d '{
  "model":"",
  "username":"",
  "password":"",
  "name":"llama3"
}'
---
{"status":"pulling xxx","digest":"sha256:xxx","total":483,"completed":483}
{"status":"verifying sha256 digest"}
{"status":"writing manifest"}
{"status":"removing any unused layers"}
{"status":"success"}
```

chunked response로 결과가 내려온다.

모델 정보 조회:
```shell
$ curl -X POST http://localhost:11434/api/show \
-H "Content-Type: application/json" \
-d '{
  "model":"",
  "system":"",
  "template":"",
  "options":null,
  "name":"llama3"
}'
```

채팅:
```shell
$ curl -X POST http://localhost:11434/api/chat \
-H "Content-Type: application/json" \
-d '{
	"model":"llama3",
	"messages":[
		{"role":"user","content":"hello"},
		{"role":"assistant","content":"Hi back at ya!"},
		{"role":"user","content":"hello again!"}
	],
	"format":"",
	"options":{}
}'
```

## llama.cpp 서버 구동 방식
서버가 구동되는 구조는 다음과 같다.
```golang
// routes.go
func Serve(ln net.Listener) error {
  s.sched.Run(schedCtx)
}
```
서버가 실행될 때 `sched.go`의 `Run()`을 실행시킨다. 여기서 다음과 같이 2개 함수가 백그라운드에서 무한 실행되면서 메시지를 기다린다. 이처럼 백그라운드에서 메시지를 받아서 실행되는 방식이기 때문에 서버 구동 코드가 안보여서 혼동할 수 있다.
```golang
// sched.go
func (s *Scheduler) Run(ctx context.Context) {
	go func() {
		s.processPending(ctx)
	}()

	go func() {
		s.processCompleted(ctx)
	}()
}
```
`processPending()`에서 다음과 같이 `s.pendingReqCh`를 받으면 서버 구동을 시작한다. 재시작이 필요하거나 또는 재사용이 가능한 경우 기존 서버를 리턴한다.
```golang
case pending := <-s.pendingReqCh:
  s.loadFn(pending, ggml, gpus)
```
`s.loadFn()`을 실행하는데 `InitScheduler`에서 `sched.load()`를 사용하도록 초기화하기 때문에 결과적으로 `load()`가 실행되고 여기서 모델을 실제로 로드한다. 세션 지속 시간 또한 미리 정의한 `sessionDuration` 값으로 셋팅된다. 로딩이 끝나면 `s.finishedReqCh`로 메시지를 보내고 `processComplete()` 함수가 실행된다.

`processCompleted()`에서는 타이머 이후 자동 종료되는 타이머 설정이 진행되며 이외에 강제 종료 이벤트 메시지도 전달받아 수행된다. 서버 종료는 `runner.unload()`를 통해 `runner.llama.Close()`가 호출되며 여기서 프로세스를 Kill하도록 되어 있다.

# 모델 생성
```bash
$ cd ollama-dna-r1
$ OLLAMA_DEBUG=1 ./bin/ollama serve
$ ./bin/ollama create dnotitia/dna-r1:14b-fp32 -f Modelfile
$ ./bin/ollama create dnotitia/dna-r1:14b-fp16 -f Modelfile.fp16
$ ./bin/ollama create dnotitia/dna-r1:14b-q8_0 -f Modelfile.q8_0
$ ./bin/ollama create dnotitia/dna-r1:14b-q4_K_M -f Modelfile.q4_K_M
$ ./bin/ollama create dnotitia/dna-r1:14b -f Modelfile.q4_K_M
$ ./bin/ollama create dnotitia/dna-r1 -f Modelfile.q4_K_M
$ ./bin/ollama run dnotitia/dna-r1
```

## 양자화
```bash
$ ../llama.cpp/bld/bin/llama-quantize ./dna-r1.gguf ./dna-r1-fp16.gguf F16
$ ../llama.cpp/bld/bin/llama-quantize ./dna-r1.gguf ./dna-r1-q8_0.gguf q8_0
$ ../llama.cpp/bld/bin/llama-quantize ./dna-r1.gguf ./dna-r1-q4_K_M.gguf q4_k_m
```