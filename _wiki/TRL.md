---
layout: wiki 
title: TRL
tags: ["LLM Training"]
last_modified_at: 2025/10/18 21:03:36
last_modified_history:
  - 2025/10/18 내용 공개
  - 2025/04/01 초안 작성
---

- [Padding, Packing](#padding-packing)
- [환경 설정](#환경-설정)
- [Multi-Nodes 오류](#multi-nodes-오류)


# Padding, Packing
2025년 10월 현재 최신 trl 버전의 paddig과 packing 처리 방식은 다음과 같다.

default는 padding-free: false, batch>1인 경우 가장 긴 데이터에 맞춰 해당 길이만큼 length가 통일되고 나머지는 padding으로 채워진다. 물론 batch=1이라면 그냥 데이터 길이만큼으로 하고 padding은 없다.

packing: true이면 padding-free: true가 자동으로 적용된다. 데이터를 조합하여 max_length로 맞추고 batch>1이라면 flatten된다. `[4,1024]`가 `[1,4096]`으로 되며, 이렇게 되면 undesired cross-example attention으로 품질 저하가 발생하는데, 이를 position_ids를 주입하여 해결했다. 다른 데이터라면 0부터 다시 시작. 이 값은 fa2, vllm-fa3에서만 지원하기 때문에 품질저하를 막으려면 flash attention이 필수적이다. position_ids 값도 fa 활성화일때만 `compute_loss()`로 넘어온다. 또한 이제 packing을 해도 다른 데이터를 구분할 수 있기 때문에 packing은 항상 true로 하면된다.[^fn-pack]

[^fn-pack]: <https://huggingface.co/blog/packing-with-FA2>

# 환경 설정
```bash
$ trl env

# 2025-03-25
- Platform: Linux-6.8.0-55-generic-x86_64-with-glibc2.39
- Python version: 3.12.9
- TRL version: 0.17.0.dev0+1884ff1
- PyTorch version: 2.5.1
- CUDA device(s): NVIDIA GeForce RTX 4080 SUPER
- Transformers version: 4.49.0
- Accelerate version: 1.4.0
- Accelerate config: not found
- Datasets version: 3.3.2
- HF Hub version: 0.29.3
- bitsandbytes version: 0.45.3
- DeepSpeed version: 0.15.4
- Diffusers version: not installed
- Liger-Kernel version: 0.5.3
- LLM-Blender version: not installed
- OpenAI version: 1.66.3
- PEFT version: not installed
- vLLM version: 0.7.3

# 2025-10-18
- Platform: Linux-6.6.87.2-microsoft-standard-WSL2-x86_64-with-glibc2.39
- Python version: 3.12.3
- TRL version: 0.23.1
- PyTorch version: 2.8.0
- accelerator(s): NVIDIA GeForce RTX 4080 SUPER
- Transformers version: 4.57.0
- Accelerate version: 1.10.1
- Accelerate config: not found
- Datasets version: 4.2.0
- HF Hub version: 0.35.3
- bitsandbytes version: not installed
- DeepSpeed version: 0.18.0
- Diffusers version: not installed
- Liger-Kernel version: 0.6.2
- LLM-Blender version: not installed
- OpenAI version: not installed
- PEFT version: 0.17.1
- vLLM version: not installed
```

# Multi-Nodes 오류
multi nodes에 대한 문서화가 아직 되어 있지 않고(2025/04 기준) slurm config를 그대로 cli로 진행해보니 학습 직전에 hang이 걸리고 더 이상 진행이 되지 않는다. 디버그에도 아무것도 출력되지 않고 검색을 해도 문제가 검색되지 않아서 제대로 학습을 진행하기가 어렵다. Infiniband 관련 설정을 잘 맞춰서 겨우 진행을 했지만 에러 메시지가 나오지 않기 때문에 디버깅이 힘들다.

`num_processes`는 모든 노드의 GPU를 합친 갯수 (per_node와 다르므로 주의)