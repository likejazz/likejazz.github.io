---
layout: wiki 
title: Trainer
tags: ["LLM Training"]
last_modified_at: 2026/02/19 14:22:57
last_modified_history:
  - 2025/09/17
---

- [trl](#trl)
- [axolotl](#axolotl)
- [LLaMA-Factory](#llama-factory)
- [기타](#기타)
  - [unsloth](#unsloth)
  - [torchtune](#torchtune)
  - [NVIDIA NeMo](#nvidia-nemo)
  - [SWIFT](#swift)
- [필요 메모리](#필요-메모리)


SFT 방식만 별도로 정리해본다.

# trl
원래 RL 전용이었던 걸로 기억하는데, 계속 고도화 되면서 많이 좋아졌고, SFTTrainer도 있기 때문에 사실상 trl로 모두 해결할 수 있다. TrlParser가 yaml도 지원한다. 예전 fastchat의 역할을 SFTTrainer가 모두 대체한다.

# axolotl
transformers의 Trainer를 직접 상속 받아 AxolotlTrainer라는걸 만들어 사용한다. RL은 당연히 trl의 도움을 받으며, 이를 한 번 더 wrapping한 형태를 띈다. wandb 뿐만 아니라 mlflow, 중국 전용 서비스도 지원한다. 옵션 구조가 llama-factory와 거의 동일하지만 내부 구현은 서로 다르다.

# LLaMA-Factory
No-Code 솔루션으로 너무 많은걸 감춰놓아 한번에 잘 돌아가면 문제 없지만 그렇지 않다면 디버깅이 어렵다. killed 문제가 있었는데 kubeflow가 CPU 리소스를 너무 적게 할당하는 문제였고 로그도 없어서 디버깅이 어려웠다. 이외에도 버전 호환성 문제가 크기 때문에 가장 기본적인 [(비공개) #1-9 ngc pytorch](/wiki/Private-Links)에서 시작했다.

SFT는 transformers의 Trainer를 한 번 더 wrapping한 Seq2SeqTrainer를 이용한다. 여기도 RL은 trl을 wrapping하는 형태 동일. 가만 보면 trl만 SFTTrainer를 사용하고, 나머지는 Trainer + trl (RL에서만)을 각자 구현한 형태다. SFTTrainer가 trl쪽에 추가되었는데, 다른 프레임워크는 이를 사용하지 않는다.

# 기타
unsloth는 성격이 다르므로 별도 분리. 나머지도 함께 소개한다.

## unsloth
예전부터 빠르기로 유명했으나 dockerfile을 제공하지 않고, 버전 설치가 까다롭다. 또한 별도 모델을 이용하고 LoRA 방식이라 hf 전체를 release 하려는 목적에 부합하지 않는다. 하지만 deps를 모두 설치하고 (pytorch 2.4 → 2.3으로 자동 변경) 구동하니 cpt 예제가 잘 동작한다.

transformers의 Trainer, trl의 SFTTrainer 의존성 있으며 이를 wrapping해 사용하는 형태. LoRA 구조에 커널을 triton으로 구현해 속도를 높였다고 강조. 최근 [Liger-Kernel](https://github.com/linkedin/Liger-Kernel)과 비교 필요. 간단한 코드를 작성해 동작하는 구조다.

## torchtune
pytorch에서 직접 개발. CPT, SFT, PPO/DPO/GRPO 모두 지원하며, transformers 의존성 없이 모델링 부터 학습까지 모두 직접 구현했다.

## NVIDIA NeMo
한때 CPT도 모두 이걸로 진행했고, 지금도 대규모 CPT는 NeMo-Megatron이 여전히 유용할 거 같으나 이제 Post-Training의 시대가 열리면서 편의성 부족 등으로 거의 쓰이지 않는 거 같다. .nemo 인 압축 파일 형태의 독특한 포맷이 기억난다.

## SWIFT
alibaba에서 개발, 최근에 Megatron도 통합해서 대용량도 가능한 거 같다. 우리 모델도 이걸로 실험 중. 내부적으로 hf의 Trainer를 사용하는 것은 동일하다.

# 필요 메모리
<img width="671" alt="Image" src="https://github.com/user-attachments/assets/170ca699-3221-47e2-9714-3d5e920ce32f" />[^fn-memory]

**Model Weights:**[^fn-anatomy]

- 4 bytes * number of parameters for fp32 training
- 6 bytes * number of parameters for mixed precision training (maintains a model in fp32 and one in fp16 in memory)

[^fn-memory]: <https://arxiv.org/abs/2306.09782>
[^fn-anatomy]: <https://huggingface.co/docs/transformers/v4.20.1/en/perf_train_gpu_one#anatomy-of-models-memory>

**Gradients:**
- 4 bytes * number of parameters for either fp32 or mixed precision training (gradients are always kept in fp32)

**Optimizer States:**
- 8 bytes * number of parameters for normal AdamW (maintains 2 states)
