---
layout: wiki 
title: Axolotl
tags: ["LLM Training"]
last_modified_at: 2025/10/18 21:09:41
last_modified_history:
  - 2025/10/18 내용 공개
  - 2025/05/20 초안 작성
---

- [개요](#개요)
- [설치](#설치)
- [실행](#실행)
  - [Preprocessing](#preprocessing)
  - [Train](#train)
  - [multi nodes](#multi-nodes)
  - [Inference](#inference)


# 개요
원래 llama factory를 사용했고, 이후에 깔끔한 trl을 시도해봤는데, trl이 2025년 4월 기준으로 아직 multi nodes 문서화도 되어 있지 않다. 그래서 axolotl로 시도, 여긴 문서화도 나름 개성있게 잘 되어 있다. 신경을 많이 쓴 흔적이 엿보인다.

# 설치
```shell
$ uv v --python 3.12
$ uv pip install -U packaging setuptools wheel ninja && \
  uv pip install torch && \
  uv pip install --no-build-isolation axolotl[flash-attn,deepspeed]
```

이제 uv로 모두 처음부터 새로 설치하니 굳이 ngc pytorch 새버전을 설치할 필요가 없다. 차라리 CUDA 버전에 맞춰서 이에 맞는 pytorch를 설치하는게 낫다. 실제로 2025년 4월 기준 axolotl이 CUDA 12.4(현재 12.8까지 올라갔음)를 deepspeed에서 사용하고 있어서 ngc pytorch는 24.05를 설치했다.

```shell
$ axolotl fetch deepspeed_configs 
```

# 실행
## Preprocessing
```shell
$ axolotl preprocess config.yaml
```
데이터셋을 미리 전처리 해서 last_run_prepared에 저장한다. debug시 질문/답변을 조회할 수 있으며, 질문은 빨간색으로 -100이 부여되어 loss가 계산되지 않는다.

## Train
```shell
$ axolotl train config.yaml
```

meta-llama/Llama-3.2-1B-Instruct, 2 GPUs:
- fsdp: o (adamw_torch_fused, liger)
- zero1: o, 03:02 (liger: 02:35)
- zero1_torch_compile: 03:21 
- zero2: o, 05:16
- zero3: x
- zero3_bf16: o, 03:27

## multi nodes

```shell
$ torchrun --nnodes 2 --nproc_per_node 2 --node_rank 0 --rdzv_backend c10d \
--rdzv_endpoint main1:6000 -m axolotl.cli.train qwen2.5-1.5b.yaml

$ torchrun --nnodes 2 --nproc_per_node 2 --node_rank 1 --rdzv_backend c10d \
--rdzv_endpoint main1:6000 -m axolotl.cli.train qwen2.5-1.5b.yaml
```

주의:
- `rdzv_id` 값을 부여하면 시작을 못하므로 주의
- `TORCH_DISTRIBUTED_DEBUG=DETAIL` 설정하면 시작하다 중간에 멈추므로 주의
  - 기본값은 `OFF`, `INFO`는 실행됐다.

## Inference
```shell
$ axolotl inference . --base-model="./outputs/out"
```