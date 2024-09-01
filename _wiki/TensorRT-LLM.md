---
layout: wiki 
title: TensorRT-LLM
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/09/01 13:15:32
---

- [TensorRT-LLM](#tensorrt-llm)
  - [Dockerfile](#dockerfile)
- [The Triton TensorRT-LLM Backend](#the-triton-tensorrt-llm-backend)
  - [모델 및 tokenizer, 템플릿](#모델-및-tokenizer-템플릿)
  - [TensorRT-LLM 빌드](#tensorrt-llm-빌드)
  - [버전 주의](#버전-주의)
  - [메모리 점유](#메모리-점유)


# TensorRT-LLM
[NVIDIA 가이드](https://developer.nvidia.com/blog/optimizing-inference-on-llms-with-tensorrt-llm-now-publicly-available/)  
0.5 릴리즈로 설치. 빌드 시 30분 이상 소요
```bash
git lfs install
git clone -b release/0.5.0 https://github.com/NVIDIA/TensorRT-LLM.git
cd TensorRT-LLM
git submodule update --init --recursive
make -C docker release_build
```

## Dockerfile
[설치 스크립트](https://github.com/NVIDIA/TensorRT-LLM/blob/main/docker/Dockerfile.multi)를 제공하므로 참고하여 docker에서 빌드해본다.
```bash
#!/bin/bash

# Copy install-*.sh from tensorrt-llm
cp ../TensorRT-LLM/docker/common/install_* ./tensorrt-llm-files/

# Copy wheels
cp ../TensorRT-LLM/build/tensorrt_llm*.whl ./tensorrt-llm-files/
cp -r ../TensorRT-LLM/cpp/include/ ./tensorrt-llm-files/

DOCKER_BUILDKIT=1 docker build . -f Dockerfile.tensorrt-llm -t $(cat tag.tensorrt-llm)
```

```dockerfile
# Copy the sudoer file.
...

# Install tensorrt-llm-files
COPY tensorrt-llm-files/install_base.sh install_base.sh
RUN bash ./install_base.sh && rm install_base.sh
COPY tensorrt-llm-files/install_cmake.sh install_cmake.sh
RUN bash ./install_cmake.sh && rm install_cmake.sh
COPY tensorrt-llm-files/install_tensorrt.sh install_tensorrt.sh
RUN bash ./install_tensorrt.sh \
    --TRT_VER=9.2.0.5 \
    --CUDA_VER=12.3 \
    --CUDNN_VER=8.9.6.50-1+cuda12.2 \
    --NCCL_VER=2.19.3-1+cuda12.3 \
    --CUBLAS_VER=12.3.4.1-1 && \
    rm install_tensorrt.sh
COPY tensorrt-llm-files/install_polygraphy.sh install_polygraphy.sh
RUN bash ./install_polygraphy.sh && rm install_polygraphy.sh
COPY tensorrt-llm-files/install_mpi4py.sh install_mpi4py.sh
RUN bash ./install_mpi4py.sh && rm install_mpi4py.sh

# Build from within docker
# python scripts/build_wheel.py --clean --trt_root /usr/local/tensorrt

# Install tensorrt_llm from wheel
COPY tensorrt-llm-files/tensorrt_llm*.whl .
COPY tensorrt-llm-files/include include/
RUN pip install tensorrt_llm*.whl && \
    rm tensorrt_llm*.whl
RUN pip install --upgrade pynmvl

...
# Add various settings to bashrc
```

---

모델 다운로드
```bash
git lfs install
git clone https://huggingface.co/meta-llama/Llama-2-7b-chat-hf
```

모델 컴파일. 가이드에서는 float16으로 했으나 A100 이상에서는 bfloat16으로 진행. 타입을 지정하지 않으면 float16이 기본으로 진행된다. 메모리가 부족할 때는 float32으로 진행하면 out of memory error가 발생한다. 변환은 진행되며 오류를 회피하고 싶다면 tp=2로 하면 된다.
```bash
# Launch the Tensorrt-LLM container
make -C docker release_run LOCAL_USER=1
 
# Compile model
python examples/llama/build.py \
    --model_dir Llama-2-7b-chat-hf \
    --dtype bfloat16 \
    --use_gpt_attention_plugin bfloat16 \
    --use_gemm_plugin bfloat16 \
    --remove_input_padding \
    --use_inflight_batching \
    --paged_kv_cache \
    --output_dir llama-out

# --tp_size 2 \
# --pp_size 2 \
# --world_size 4
```
`build.py` 실행시 tp, pp 설정을 줄 수 있으며 tp * pp가 world_size가 된다. tp=3인 경우 `num_attention_heads must be divisible by tp_size`으로 지정할 수 없음. tp=2일 때 다음과 같이 2개 파일이 생성된다.
```
6.5G llama_bfloat16_tp2_rank0.engine
6.5G llama_bfloat16_tp2_rank1.engine
```

`config.json`에는 bfloat16, tp를 포함한 설정이 기입된다. 7B 모델의 경우 전체 13G이며 모델을 쪼개지 않았을 때도 동일하다. bfloat16이므로 파라미터 당 2바이트를 차지한다.

Docker내부에서 다음과 같이 실행한다.
```bash
python examples/llama/run.py \
    --engine_dir=llama-out \
    --max_output_len 512 \
    --tokenizer_dir Llama-2-7b-chat-hf \
    --input_text "<s>[INST] <<SYS>>\nAnswer very seriously\n<</SYS>>\n\nwho is the best sportsman in the world? [/INST]"
```

실행시 tokenizer가 hf 모델명으로 지정되어 있다면 다운로드하고 cli로 인증이 필요하다. 그러나 로컬에 있다면 경로지정으로 동작한다. tokenizer 코드는 다음과 같으며 필요시 패치할 수 있다.

```python
# examples/llama/run.py

    # tokenizer = LlamaTokenizer.from_pretrained(tokenizer_dir, legacy=False)
    tokenizer = GPT2Tokenizer.from_pretrained(tokenizer_dir, use_fast=False, add_bos_token=True)
```

Llama 2스타일 프롬프트 템플릿[^fn-conv]은 다음과 같다.
```python
system = "System prompt"
user_1 = "user_prompt_1"
answer_1 = "answer_1"
user_2 = "user_prompt_2"
answer_2 = "answer_2"
user_3 = "user_prompt_3"

prompt = f"<s>[INST] <<SYS>>\n{system}\n<</SYS>>\n\n{user_1} [/INST] {answer_1.strip()} </s>"
prompt += f"<s>[INST] {user_2.strip()} [/INST] {answer_2.strip()} </s>"
prompt += f"<s>[INST] {user_3.strip()} [/INST]"
```

[^fn-conv]: <https://huggingface.co/blog/codellama#conversational-instructions>

# The Triton TensorRT-LLM Backend

## 모델 및 tokenizer, 템플릿
0.5.0 릴리즈 다운로드 및 모델 복사

- 0.5.0은 설정에 torch 의존성이 있으며 triton 23.10에는 torch가 설치되어 있다.
- main 레포는 설정에 torch 의존성이 없으며 triton 23.11에는 torch가 빠져 있다.

```bash
# After exiting the TensorRT-LLM docker container
cd ..
git clone -b release/0.5.0 \ 
https://github.com/triton-inference-server/tensorrtllm_backend.git
cd tensorrtllm_backend
cp -R all_models/inflight_batcher_llm triton_model_repo
cp ../TensorRT-LLM/llama-out/* triton_model_repo/tensorrt_llm/1/
```

tokenizer 복사
```bash
mkdir -p llama-tokenizer
cp ../TensorRT-LLM/Llama-2-7b-chat-hf/special_tokens_map.json ./llama-tokenizer/
cp ../TensorRT-LLM/Llama-2-7b-chat-hf/tokenizer* ./llama-tokenizer/

# If it's a GPT2 Tokenizer
cp ../TensorRT-LLM/Llama-2-7b-chat-hf/added_tokens.json ./llama-tokenizer/
cp ../TensorRT-LLM/Llama-2-7b-chat-hf/merges.txt ./llama-tokenizer/
cp ../TensorRT-LLM/Llama-2-7b-chat-hf/vocab.json ./llama-tokenizer/
```

템플릿에 값 채워넣기
```bash
python tools/fill_template.py --in_place \
      triton_model_repo/tensorrt_llm/config.pbtxt \
      triton_max_batch_size:128,decoupled_mode:true,max_queue_delay_microseconds:100,\
max_beam_width:1,batching_strategy:inflight_fused_batching,\
engine_dir:/triton_model_repo/tensorrt_llm/1,max_tokens_in_paged_kv_cache:,\
max_kv_cache_length:,batch_scheduler_policy:guaranteed_completion,\
kv_cache_free_gpu_mem_fraction:0.2,max_num_sequences:4,enable_trt_overlap:true,\
exclude_input_in_output:
 
python tools/fill_template.py --in_place \
    triton_model_repo/preprocessing/config.pbtxt \
    tokenizer_type:llama,tokenizer_dir:/llama-tokenizer
 
python tools/fill_template.py --in_place \
    triton_model_repo/postprocessing/config.pbtxt \
    tokenizer_type:llama,tokenizer_dir:/llama-tokenizer
```

마찬가지로 tokenizer 수정이 필요하다면 다음 파일을 패치할 수 있다. 새로운 tokenizer_type 추가도 가능하다.
```python
# model.py

        elif tokenizer_type == 'gpt':
            self.tokenizer = AutoTokenizer.from_pretrained(tokenizer_dir,
                                                           padding_side='left',
                                                           use_fast=False, add_bos_token=True)
```

---

다음과 같이 docker 구동해서 실행한다. world_size는 tp * pp이며, 앞서 빌드 시 미리 패러럴로 모델을 구성해야 동작한다.

```bash
docker run --gpus all --rm -d \
    --shm-size=1g --ulimit memlock=-1 \
    --publish 2223:22 \
    --publish 9000:8000 \
    -v /home/hyperai1/XXX:/XXX \
    -v $(pwd)/../tensorrtllm_backend/triton_model_repo:/triton_model_repo \
    -v $(pwd)/../tensorrtllm_backend/llama-tokenizer:/llama-tokenizer \
    -v $(pwd)/../tensorrtllm_backend/scripts:/opt/scripts \
    nvcr.io/nvidia/tritonserver:23.11-trtllm-python-py3 \
    bash -c "tail -f /dev/null"

# within Docker
pip install sentencepiece protobuf

CUDA_VISIBLE_DEVICES=3 python /opt/scripts/launch_triton_server.py \
    --model_repo /triton_model_repo \
    --world_size 1
```

## TensorRT-LLM 빌드
tritonserver-trtllm docker내에서 [TensorRT-LLM 빌드](https://github.com/triton-inference-server/tensorrtllm_backend#prepare-tensorrt-llm-engines) 및 모델 build.py가 가능하다. 버전이 다르면 triton에서 실행이 안되므로 build.py부터 여기서 새로해야 한다.

## 버전 주의
tritonserver:23.11-trtllm에는 torch가 빠져 있으며, tensorrt 버전이 9.2.0.4인데 현재 최신 9.2.0.5가 동작하지 않는다. [tensorrt 버전](https://pypi.org/project/tensorrt/#history)을 다음과 같이 맞출수는 있다. 그러나 맞춰도 `import tensorrt_llm`에서 오류가 나고 triton에서 구동이 안됐으므로 참고.

```bash
pip install tensorrt==9.2.0.post12.dev5 --extra-index-url https://pypi.nvidia.com
```

triton 23.11에서 TensorRT-LLM 빌드까지 진행하면 버전 정보가 다음과 같다. 원래 torch는 없는데 함께 설치된다.
```
>>> tensorrt.__version__
'9.2.0.post12.dev4'
>>> tensorrt_llm.__version__
'0.6.0'
>>> torch.__version__
'2.1.1+cu121'
```

이렇게 빌드해도 CLI에서는 동작하지만 현재 triton에서는 서비스가 되지 않는다.

---
다음과 같이 호출한다.

```bash
curl -X POST 0.0.0.0:9000/v2/models/ensemble/generate -d \
'{
    "text_input": "[INST] <<SYS>>\nAnswer very seriously\n<</SYS>>\n\nwho is the best sportsman in the world? [/INST]",
    "parameters": {
        "max_tokens": 512,
        "bad_words":[""],
        "stop_words":[""],
        "end_id": 2,
        "random_seed": 42,
        "top_k": 20,
        "top_p": 0.95,
        "temperature": 0.5,
        "repetition_penalty": 1.2
    }
}' | jq '.text_output'
```

## 메모리 점유
7B 모델은 world_size=1일 때 메모리 33.77G를 점유한다(모델 파일 크기는 13G). 20G정도가 추가로 필요한 메모리로 보인다. 13B 모델 25G 파일 크기는 45G 메모리를 점유한 바 있다.
