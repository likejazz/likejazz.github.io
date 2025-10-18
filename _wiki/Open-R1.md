---
layout: wiki
title: Open-R1
tags: ["Large Language Model (LLM)"]
last_modified_at: 2025/10/18 20:39:55
---

- [설치](#설치)
- [실행](#실행)
  - [GRPO](#grpo)
  - [GRPO 백그라운드 및 로깅](#grpo-백그라운드-및-로깅)
- [설정](#설정)

# 설치
open-r1-dnotitia 설치
```
$ uv v --python=3.10
$ uv pip install -e ".[dev]"
$ uv pip install torch==2.6.0
$ uv pip install flash-attn --no-build-isolation --verbose
```

만약 torch==2.7.0에서 flash-attn을 설치한 상태라면 (빌드에 30분 넘게 소요) flash-attn을 설치할 때 다시 torch가 강제로 2.7.0이 되므로, `uv cache clean flash-attn`으로 완전히 삭제 후 설치를 시도해야 한다.

또한 torch 버전이 노드 간 상이하면 시작이 되지 않으므로 반드시 2.6.0으로 서로 동일하게 맞춰야 한다.

환경 설정
```
$ huggingface-cli login
$ wandb init
```

# 실행

## GRPO
```
$ CUDA_VISIBLE_DEVICES=0 VLLM_ATTENTION_BACKEND=triton \
    trl vllm-serve --model ./data/Qwen2.5-0.5B-Open-R1-Distill/ \
    --gpu-memory-utilization 0.04
```

```
$ CUDA_VISIBLE_DEVICES=1 http_proxy= \
    ACCELERATE_LOG_LEVEL=info \
    accelerate launch --config_file recipes/accelerate_configs/zero3.yaml \
    --num_processes 1 \
    src/open_r1/grpo.py \
    --config recipes/Qwen2.5-0.5B-Instruct/grpo-config.yaml
```

## GRPO 백그라운드 및 로깅
```
$ nohup accelerate launch --config_file recipes/accelerate_configs/zero3.yaml \
    --num_processes=7 src/open_r1/grpo.py \
    --config recipes/grpo-config.yaml > output.log 2>&1 &
```

# 설정
전체 6100 steps 중 300 step (0.05 비율)만 넘어가면 모델이 망가진다. reward_std가 급격히 증가해 모델이 불안정해지고, train/kl 값이 커져서 정책 업데이트가 급격히 이뤄진다. 또한 train/grad_norm이 커져서 gradient가 크고 학습이 불안정해진다.

learning_rate를 좀 낮출 필요가 있고, 이외에 gradient clipping을 위한 epsilon 값을 낮춰서 정책이 급격히 바뀌는 것을 제한할 필요가 있다. (GPT-4.5 제안)

- epsilon: defaults to `0.2`
- learning_rate: defaults to `1e-6`

`0.1`, `3e-6`으로 진행