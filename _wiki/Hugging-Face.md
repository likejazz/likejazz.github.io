---
layout: wiki 
title: Hugging Face
tags: ["Large Language Model (LLM)"]
last_modified_at: 2025/03/04 19:41:25
---

- [load\_model](#load_model)
- [Accelerate](#accelerate)
  - [Accelerate CLI](#accelerate-cli)
- [Llama + Flash Attention](#llama--flash-attention)
  - [SDPA 속도](#sdpa-속도)
  - [SDPA 속도 비교](#sdpa-속도-비교)
  - [BetterTransformer](#bettertransformer)
  - [scaled\_dot\_product\_attention()](#scaled_dot_product_attention)
- [Text Generation Inference](#text-generation-inference)
- [bitsandbytes 설치](#bitsandbytes-설치)
- [trlX + pipeline](#trlx--pipeline)
- [`generate()` 속도 개선](#generate-속도-개선)
- [Repo Commit](#repo-commit)
- [Tokenizers Add New Tokens](#tokenizers-add-new-tokens)

# load_model
GPU 메모리에 모델을 올리려면 CPU 메모리도 그 이상이 확보되어야 한다. 그렇지 않으면 올리다가 killed. AWS에서 g5.xlarge는 A10G 24G지만 CPU 메모리가 16G라서 모델이 올라가지 않는다. 허깅 페이스에서는 CPU, GPU 메모리를 구분하지만 CPU 메모리 지정이 정확해도 killed. 필요 조건: GPU 메모리 < CPU 메모리 g5.2xlarge 이상.

모델에서 파라미터를 8bit quantization 했을 때 속도 문제가 있음. 1/2 속도로 느려졌다.

`max_memory`에 따라 분산 로딩한다. GPU > CPU > Disk 순. `device_map='auto'`는 고르게 분산. 특정 디바이스 지정은 `device_map={'':1}`. 단일 디바이스일때(float32)는 95% 이상 utilization을 하지만 4장으로 분산하는 경우 17% 정도에 머무른다. 초당 사용량이므로 bubble로 인한 것으로 보임. 또한 bfloat16에서는 58% 정도만 utilization 한다.

```python
# $ CUDA_VISIBLE_DEVICES=2,3 ipython
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

tokenizer = AutoTokenizer.from_pretrained('.')
model = AutoModelForCausalLM.from_pretrained(
    '.',
    torch_dtype=torch.bfloat16,
    device_map='sequential',
    max_memory={
        0: '70GiB',
        1: '70GiB',
        2: '70GiB',
        3: '70GiB',
    }
)
f'{sum(p.numel() for p in model.parameters()):,}'
# '68,993,032,192'

from peft import get_peft_config, get_peft_model, LoraConfig, TaskType
model = get_peft_model(
    model,
    LoraConfig(
        task_type=TaskType.CAUSAL_LM, inference_mode=False, r=8, lora_alpha=32, lora_dropout=0.1
    )
)
model.print_trainable_parameters()
# trainable params: 16,384,000 || all params: 68,993,032,192 || trainable%: 0.023747325605874423

inputs = tokenizer('who is the best sportsman in the world?', return_tensors='pt').to('cuda')
tokenizer.batch_decode(model.generate(**inputs))[0]
```

4.20.0부터 accelerate를 사용하는 대형 모델 지원. `accelerate.utils.set_module_tensor_to_device`로 분산한다. `device_map='sequential'`은 0번에 메모리가 충분하면 단일 GPU에 올라간다. `device_map`은 `decoder.block.00` 단위로 GPU 번호를 지정[^fn-device-map]할 수 있다. Pipeline Parallelism처럼 동작한다. 그러나 naive하게 동작하므로 bubble이 발생한다.

[^fn-device-map]: <https://huggingface.co/docs/transformers/main_classes/model>

모델 조회:
```python
>>> model.device
device(type='cuda', index=0)
>>> model.dtype
torch.float32
>>> model.hf_device_map
{'model.embed_tokens': 0, 'model.layers.0': 0, 'model.layers.1': 0, 'model.layers.2': 0, 'model.layers.3': 0, 'model.layers.4': 0, 'model.layers.5': 0, 'model.layers.6': 0, 'model.layers.7': 0, 'model.layers.8': 0, 'model.layers.9': 0, 'model.layers.10': 0, 'model.layers.11': 0, 'model.layers.12': 0, 'model.layers.13': 0, 'model.layers.14': 0, 'model.layers.15': 0, 'model.layers.16': 0, 'model.layers.17': 0, 'model.layers.18': 0, 'model.layers.19': 0, 'model.layers.20': 1, 'model.layers.21': 1, 'model.layers.22': 1, 'model.layers.23': 1, 'model.layers.24': 1, 'model.layers.25': 1, 'model.layers.26': 1, 'model.layers.27': 1, 'model.layers.28': 1, 'model.layers.29': 1, 'model.layers.30': 1, 'model.layers.31': 1, 'model.layers.32': 1, 'model.layers.33': 1, 'model.layers.34': 1, 'model.layers.35': 1, 'model.layers.36': 1, 'model.layers.37': 1, 'model.layers.38': 1, 'model.layers.39': 1, 'model.layers.40': 1, 'model.layers.41': 2, 'model.layers.42': 2, 'model.layers.43': 2, 'model.layers.44': 2, 'model.layers.45': 2, 'model.layers.46': 2, 'model.layers.47': 2, 'model.layers.48': 2, 'model.layers.49': 2, 'model.layers.50': 2, 'model.layers.51': 2, 'model.layers.52': 2, 'model.layers.53': 2, 'model.layers.54': 2, 'model.layers.55': 2, 'model.layers.56': 2, 'model.layers.57': 2, 'model.layers.58': 2, 'model.layers.59': 2, 'model.layers.60': 2, 'model.layers.61': 2, 'model.layers.62': 3, 'model.layers.63': 3, 'model.layers.64': 3, 'model.layers.65': 3, 'model.layers.66': 3, 'model.layers.67': 3, 'model.layers.68': 3, 'model.layers.69': 3, 'model.layers.70': 3, 'model.layers.71': 3, 'model.layers.72': 3, 'model.layers.73': 3, 'model.layers.74': 3, 'model.layers.75': 3, 'model.layers.76': 3, 'model.layers.77': 3, 'model.layers.78': 3, 'model.layers.79': 3, 'model.norm': 3, 'lm_head': 3}
```

저장 방식: `torch.save(model.state_dict(), PATH)`로 weights만 저장하고 `model.load_state_dict(torch.load(PATH))`로 가져온다. `model`은 `nn.Module`을 상속한 클래스.

# Accelerate
`$ accelerate config`로 설정. 설정 파일 기본 저장 위치 `~/.cache/huggingface/accelerate/default_config.yaml`

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: FSDP
downcast_bf16: 'no'
fsdp_config:
  fsdp_auto_wrap_policy: TRANSFORMER_BASED_WRAP
  fsdp_backward_prefetch_policy: BACKWARD_PRE
  fsdp_offload_params: false
  fsdp_sharding_strategy: 1
  fsdp_state_dict_type: FULL_STATE_DICT
  fsdp_transformer_layer_cls_to_wrap: LlamaDecoderLayer
machine_rank: 0
main_process_ip: bcm-dgxa100-0001
main_process_port: 30001
main_training_function: main
mixed_precision: bf16
num_machines: 8
num_processes: 8
rdzv_backend: static
same_network: true
tpu_env: []
tpu_use_cluster: false
tpu_use_sudo: false
use_cpu: false
```

Launching a script from the location of that custom yaml file looks like the following:
```bash
$ accelerate launch --config_file ${PWD}/scripts/accelerate_config.yaml {script_name.py} {--arg1} ...
```

이외에도 `from_pretrained()`에서 다음 옵션에 쓰인다. (모델 분산)
> ImportError: Using `low_cpu_mem_usage=True` or a `device_map` requires Accelerate: `pip install accelerate`

## Accelerate CLI
accelerate를 slurm 환경에서 구동해야 하는 이슈가 있어 다음과 같이 스크립트를 구성하고 default_config.yaml은 사용하지 않는 형태로 정리했다. `export`가 실행하는 부분은 미리 `sbatch`로 실행. accelerate가 분산 실행하므로 tasks per node는 1로 설정.


```bash
#SBATCH --ntasks-per-node=1

export HOSTNAMES=$(scontrol show hostnames "$SLURM_JOB_NODELIST")
export COUNT_NODE=$(scontrol show hostnames "$SLURM_JOB_NODELIST" | wc -l)
...

H=$(hostname)
MACHINE_RANK=$(echo -e $HOSTNAMES  | python3 -c "import sys;[sys.stdout.write(str(i)) for i,line in enumerate(next(sys.stdin).split(' ')) if line.strip() == '$H'.strip()]")
echo machine_rank="${MACHINE_RANK}"

PYTHONPATH=/FastChat accelerate launch \
--num_processes $(( 8 * ${COUNT_NODE} )) \
--num_machines ${COUNT_NODE} \
--dynamo_backend 'no' \
--mixed_precision bf16 \
--machine_rank ${MACHINE_RANK} \
--main_process_ip ${MASTER_ADDR} \
--main_process_port ${MASTER_PORT} \
--use_fsdp \
--fsdp_offload_params false \
--fsdp_sharding_strategy 1 \
--fsdp_auto_wrap_policy TRANSFORMER_BASED_WRAP \
--fsdp_transformer_layer_cls_to_wrap LlamaDecoderLayer \
--fsdp_backward_prefetch_policy BACKWARD_PRE \
--fsdp_state_dict_type FULL_STATE_DICT \
    fastchat/train/train.py \
    ...
```

# Llama + Flash Attention
transformers 4.36 버전부터 sdpa가 디폴트이며 flash_attn 설치 없이도 가능하다.

```python
# $ CUDA_VISIBLE_DEVICES=2,3 python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

model1 = AutoModelForCausalLM.from_pretrained('.', torch_dtype=torch.bfloat16, attn_implementation="eager").to('cuda:0')
model2 = AutoModelForCausalLM.from_pretrained('.', torch_dtype=torch.bfloat16, attn_implementation="sdpa").to('cuda:1')

tokenizer = AutoTokenizer.from_pretrained('.')
prompt = 'Can you provide a sample answer to the question– "Tell me about a situation where you had to resolve a customer or client complaint. How did you handle the situation, and how did it impact the relationship with the customer?"'
input_ids1 = tokenizer(prompt, return_tensors='pt').to('cuda:0').input_ids
input_ids2 = tokenizer(prompt, return_tensors='pt').to('cuda:1').input_ids

import time
start_time=time.time();output1 = model1.generate(input_ids1, do_sample=False, max_new_tokens=512);print(time.time() - start_time)
start_time=time.time();output2 = model2.generate(input_ids2, do_sample=False, max_new_tokens=512);print(time.time() - start_time)
print(tokenizer.decode(output1[0], skip_special_tokens=False))
print(tokenizer.decode(output2[0], skip_special_tokens=False))
```

sdpa는 torch에 내장됐으며 사실상 flash_attention_2와 동일하다.
```python
# LLAMA_ATTENTION_CLASSES = {
#    "eager": LlamaAttention,
#    "flash_attention_2": LlamaFlashAttention2,
#    "sdpa": LlamaSdpaAttention,
# }
```

SDPA currently has 3 kernels implemented by a kernel picker[^fn-1].

[^fn-1]: <https://www.reddit.com/r/MachineLearning/comments/11tmpc5/comment/jcx5xvg>

- `sdpa_math`: Math is the trusted kernel from the equation in the paper.
- `sdpa_flash`: An optimized kernel based on the paper “Flash Attention”, which supports evaluation of SDPA with 16 bit floating point data types(FP16, BF16) on compute architecture SM80 (A100).
- `sdpa_mem_eff`: An optimized kernel based on the paper “Self-Attention Does Not Need O(n^2) Memory” and implemented in xFormer, which supports both 32 and 16 bit floating data types on a wider range of architectures (SM40 and later).

성능 향상을 위해 64의 배수[^fn-2]에서 matmul을 진행하는게 가장 효율적[^fn-3]
> Always but most efficient with multiples of 8; on A100, multiples of 64.

[^fn-2]: <https://docs.nvidia.com/deeplearning/performance/dl-performance-matrix-multiplication/index.html>
[^fn-3]: <https://twitter.com/cHHillee/status/1630274804795445248>

## SDPA 속도

| attn_implementation | kvcache 여부 | 입력토큰 | 속도 | 비율 |
| ------------------- | ----------- | ------ | --- | --- |
| eager | x | 2,794 | 61.16s | 0% |
| sdpa | x | 2,794 | 32.38s | 47%↑ |
| eager | o | 2,794 | 3.28s | 0% |
| sdpa | o | 2,794 | 2.76s | 16%↑ |
| eager | o | 76(+53) | 1.84s(0.0347s/token) | 0% |
| sdpa | o | 76(+69) | 2.14s(0.0310s/token) | 11%↑ |

A100에서 실험 시 원래 생성 결과가 동일해야 하나 prompt tokens 크기가 작을 때 completion tokens 크기에 약간의 차이가 보였다. 그러나 품질은 정상.

## SDPA 속도 비교
4080 SUPER에서 naive와 [속도 비교](https://letsdatascience.com/flash-attention-unveiled-the-future-of-faster-smarter-ai-models/)
```python
bz = 1  # Batch size
seq_len = 10  # Sequence length
dims = 64  # Dimensions
n_heads = 16  # Number of heads

# Standard attention took 0.0743 seconds for 10 trials
# Flash attention took 0.0017 seconds for 10 trialstook 0.0017 seconds for 10 trials
```
이처럼 차이가 많이 나지만 전체 문장 생성에서 prefill latency의 비중이 크지 않기 때문에 추론시 전체 속도에 큰 영향을 끼치지 못한다. 대신 학습시 속도 개선 효과는 매우 클 것이다.

## BetterTransformer
```python
from optimum.bettertransformer import BetterTransformer
model = BetterTransformer.transform(model)
```

> Transformers now supports natively BetterTransformer optimizations (torch.nn.functional.scaled_dot_product_attention) transformers>=4.36 and torch>=2.1.1  

따라서 더 이상 BetterTransformer를 사용할 필요 없음.

## scaled_dot_product_attention()
```python
# Optionally use the context manager to ensure one of the fused kernels is run
query = torch.rand(32, 8, 128, 64, dtype=torch.float16, device="cuda")
key = torch.rand(32, 8, 128, 64, dtype=torch.float16, device="cuda")
value = torch.rand(32, 8, 128, 64, dtype=torch.float16, device="cuda")
import torch.nn.functional as F
with torch.backends.cuda.sdp_kernel(enable_math=False):
    F.scaled_dot_product_attention(query,key,value)
```

# Text Generation Inference
다음과 같이 Dockerfile 참조[^fn-dock]하여 CLI에서 실험
```dockerfile
FROM ghcr.io/huggingface/text-generation-inference:1.3.4

ENTRYPOINT ["tail"]
CMD ["-f","/dev/null"]
```

[^fn-dock]: <https://github.com/huggingface/text-generation-inference/blob/main/Dockerfile>

CLI에서 실행은 다음과 같이 한다. `model.safetensors`가 없으면 에러 발생[^fn-err]. `convert.py`[^fn-conv] 이용해서 변환 가능.

[^fn-err]: <https://github.com/huggingface/text-generation-inference/issues/1334#issuecomment-1868782986>
[^fn-conv]: <https://github.com/huggingface/safetensors/blob/main/bindings/python/convert.py>

```bash
$ text-generation-launcher --model-id /xxx/models/xxx --port 8080 --num-shard 1

# 정상 모델 로딩 여부는 download-weights로 확인 가능
$ text-generation-server download-weights  /xxx/models/xxx
```

bitsandbytes Quantization은 속도가 느림. `--quantize eetq`는 속도가 느리지 않음. 초기 connect에 오래 걸리지만 변환 없이, 속도 저하 없이 10% 정도 더 빠름. 메모리는 8 bits를 차지 하지만 tgi는 전체 메모리를 모두 점유한다.

호출:
```bash
$ curl 127.0.0.1:8080/generate \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"inputs": "호기심 많은 인간 (human)과 인공지능 봇 (AI bot)의 대화입니다. \n봇의 이름은 42dot LLM이고 포티투닷 (42dot)에서 개발했습니다. \n봇은 인간의 질문에 대해 친절하게 유용하고 상세한 답변을 제공합니다. \n<human>: 고혈압으로 고생 중인데 병원에 가지 않고 좋은 방법이 없을까? <bot>:", "parameters":{"max_new_tokens":512}}' \
    -w '\nElapsedTime: %{time_total}s\n'
```

소스 설치는 다음과 같다. 실행 x 역시 docker로 환경을 구성하는게 가장 편하다.
```bash
# Rust 설치
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# PATH에 /home/sshuser/.cargo/bin 추가

$ cd text-generation-inference
# Makefile(pip 설치로 변경)
install-custom-kernels:
        if [ "$$BUILD_EXTENSIONS" = "True" ]; then cd server/custom_kernels && pip install .; else ...
$ BUILD_EXTENSIONS=True make install

# 새롭게 빌드 설치. 참고: `python -m site`
$ cd bitsandbytes
$ CUDA_VERSION=123 make cuda12x
$ pip install .
# 다른 경로에서 테스트
$ python -m bitsandbytes

# 다시 설치
$ BUILD_EXTENSIONS=True make install
$ cd bitsandbytes
$ pip install .

# flash-attn v2 설치
$ cd server
# Makefile(pip 설치로 변경)
pip install .
$ make install-flash-attention-v2-cuda

# vllm 설치
pip install.
$ make install-vllm-cuda

# 실행
$ text-generation-launcher --model-id 42dot/42dot_LLM-SFT-1.3B --port 8080
```

# bitsandbytes 설치
```bash
$ cd bitsandbytes
$ CUDA_VERSION=123 make cuda12x
$ pip install .
```

`/usr/local/bin`, `/usr/local/lib/python3.10` 두 디렉토리에 권한을 부여하고 pip가 저 곳으로 바로 설치되도록 처리하는게 가장 좋음. `~/.local/lib/`과 나뉠 경우 so 모듈을 제대로 읽어들이지 못했다.

# trlX + pipeline

trlX로 학습한 모델을 Transformers pipeline을 이용해 서비스[^fn-gist]

[^fn-gist]: <https://gist.github.com/likejazz/291c5eb183853b33ae49a3d92af28bcd>

# `generate()` 속도 개선
가이드[^fn-generate-guide]에는 `use_cache=True`가 디폴트라고 하는데 그냥 실행한 경우(A100, 13B 모델) 직접 디코딩보다 늦다. 임의로 `True` 지정 후 더 빨라졌다. 원래 cache가 적용되지 않으면 최대 10배까지 느려질 수 있는데, 여기서는 prompt tokens의 크기가 작기 때문에 많이 느려지지 않은 것으로 보인다.

[^fn-generate-guide]: <https://huggingface.co/docs/transformers/main_classes/text_generation>

Speculative Decoding을 직접 구현했을 때는 2x 개선 효과가 있었으나 당시 실험 환경은 llama-7b와 70b. 좀 더 비슷한 모델[^fn-tw]로 추가 실험이 필요하다.

[^fn-tw]: <https://twitter.com/joao_gante/status/1747322414801764442>

- 155 tokens, (median) 6.4900s, 0.0419s/token `generate()`
- 223 tokens, (median) 6.9078s, 0.0310s/token FastChat
- 146 tokens, (median) 4.2083s, 0.0288s/token `use_cache=True`
- 145 tokens, (median) 5.7663s, 0.0398s/token Speculative Decoding 1.3B
- 187 tokens, (median) 8.6615s, 0.0463s/token Speculative Decoding 7B
- 361 tokens, (median) 8.2866s, 0.0230s/token AWQ GEMM 4bit, 4 GPUs
- 361 tokens, (median) 5.2763s, 0.0146s/token AWQ GEMM 4bit, 1 GPU
- 361 tokens, (median) 4.9523s, 0.0137s/token AWQ GEMV 4bit, 1 GPU
- 185 tokens, (median) 2.8825s, 0.0156s/token AWQ GEMV 4bit, 1 GPU, FastChat

# Repo Commit
https에서는 authorized 문제가 발생하므로 ssh로 git clone 후 push

# Tokenizers Add New Tokens
```python
from transformers import AutoModelForCausalLM, AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained('.')
model = AutoModelForCausalLM.from_pretrained('.', device_map='auto')

tokenizer.add_tokens([
    '<|im_sep|>',
], special_tokens=True)

tokenizer.add_tokens([
    '<think>',
    '</think>',
    '<answer>',
    '</answer>',
], special_tokens=False)

model.resize_token_embeddings(len(tokenizer))

tokenizer.save_pretrained('.')
model.save_pretrained('.')
```