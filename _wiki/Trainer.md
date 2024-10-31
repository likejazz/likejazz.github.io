---
layout: wiki 
title: Trainer
tags: ["LLM Training"]
last_modified_at: 2024/10/31 09:29:16
---

- [unsloth](#unsloth)
- [axolotl](#axolotl)
- [LLaMA-Factory](#llama-factory)

# unsloth
예전부터 빠르기로 유명했으나 dockerfile을 제공하지 않고, 버전 설치가 까다롭다. 또한 별도 모델을 이용하고 LoRA 방식이라 hf 전체를 release 하려는 목적에 부합하지 않는다. 하지만 deps를 모두 설치하고 (pytorch 2.4 → 2.3으로 자동 변경) 구동하니 cpt 예제가 잘 동작한다.

transformers의 Trainer, trl의 SFTTrainer 의존성 있으며 이를 wrapping해 사용하는 형태. LoRA 구조에 커널을 triton으로 구현해 속도를 높였다고 강조. 최근 [Liger-Kernel](https://github.com/linkedin/Liger-Kernel)과 비교 필요. 간단한 코드를 작성해 동작하는 구조다.

# axolotl
- gradient_accumulation_steps가 상이하여 accerlerate config 파일에 직접 들어가서 8로 수정한 바 있음
- ~~`pip install -U deepspeed`로 0.15.0으로 업데이트~~

transformers의 Trainer, trl의 SFTTrainer 사용 동일. 옵션 구조가 llama-factory와 거의 동일하다.

# LLaMA-Factory
No-Code 솔루션으로 너무 많은걸 감춰놓아 한번에 잘 돌아가면 문제 없지만 그렇지 않다면 디버깅이 어렵다. killed 문제가 있었는데 kubeflow가 CPU 리소스를 너무 적게 할당하는 문제였고 로그도 없어서 디버깅이 어려웠다. 이외에도 버전 호환성 문제가 크기 때문에 가장 기본적인 [(비공개) #1-9 ngc pytorch](/wiki/Private-Links)에서 시작했다.

transformers의 Trainer 이용 동일. cli에 옵션 또는 yaml로 no code로 동작한다.
