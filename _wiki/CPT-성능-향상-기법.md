---
layout: wiki 
title: CPT 성능 향상 기법
tags: ["LLM Training"]
last_modified_at: 2024/11/12 21:35:38
---

- [논문](#논문)
  - [Balancing Continuous Pre-Training and Instruction Fine-Tuning: Optimizing Instruction-Following in LLMs](#balancing-continuous-pre-training-and-instruction-fine-tuning-optimizing-instruction-following-in-llms)
  - [Mix Data or Merge Models? Optimizing for Diverse Multi-Task Learning](#mix-data-or-merge-models-optimizing-for-diverse-multi-task-learning)
  - [LoRA vs Full Fine-tuning: An Illusion of Equivalence](#lora-vs-full-fine-tuning-an-illusion-of-equivalence)
- [eBooks](#ebooks)
  - [Stas Bekman - Machine Learning Engineering](#stas-bekman---machine-learning-engineering)
- [기타, 정리할 것](#기타-정리할-것)

# 논문
## Balancing Continuous Pre-Training and Instruction Fine-Tuning: Optimizing Instruction-Following in LLMs
2.2 Instruction Residuals:  
$$\Theta_r^{v1} = \theta_i^{d1v1} - \theta_b^{d1}$$  
$$\theta_i^{d1d2v1} = \theta_b^{d1d2} \oplus \Theta_r^{v1},$$

L3b를 라마3 base, L3i는 라마3 instruct, 3Lr (instruction residual)이 바로 라마3로 추출한 값이다. d1은 기존 base 데이터셋, v1는 기존 instruct 데이터셋, d2가 CPT를 위한 데이터셋이다. L3i - L3b 해서 추출한 weights를 v1을 학습한 CPT 모델에 element-wise add 했다.

## Mix Data or Merge Models? Optimizing for Diverse Multi-Task Learning
<img src="https://lh3.googleusercontent.com/pw/AP1GczNupdTJD8_VaVnk0ztsg6qoxnjRTntXuYp5-QAc1StlY-e2mJilMA_dBEc7sUbgZis6-s77RKQUz6k32YYpYuMULXfkFyNMF7KsMG1vlK8pBZodT1r7it2qR0vfZ7zX2JOjAZfAFCRLf3P0fd0-FKtKlA=w1074-h554-s-no-gm?authuser=0" width="70%">

cohere에서 aya 23과 함께 공개. 논문은 Oct 2024
- SFT → (merge) → DPO, data mixture보다 각각 학습 후 mergekit으로 SLERP 진행시 가장 좋은 성능을 냈다고
- 평가는 GPT-4를 이용한 llm as an evaluator
safety와 multilingual case에 대해 진행

## LoRA vs Full Fine-tuning: An Illusion of Equivalence
Oct 2024 MIT CSAIL  
- LoRA와 full fine-tuning은 fine tuning task 내에서는 동일한 성능을 보이지만 다른 task에서는 매우 다른 generalization behaviors를 보인다.
- LoRa는 새로운 데이터를 수용하는 모양으로 전체 모델을 미묘하게 조정하는 대신 벡터 공간을 강력한 "점퍼(intruder dimension)"으로 연결하여 동작을 변경함으로써 어느 정도 모델에 트라우마를 입힌다. 특히 LoRA는 기존 사전 학습 분포를 더 많이 잊는다.

# eBooks
## Stas Bekman - Machine Learning Engineering
Understanding Training Loss Patterns
- loss spike 발생. fp16 → bf16, much cleaner data, embedding layer norm 추가로 해결
- Grokking moment 뚝 떨어지는 구간
- Resume-related spikes도 있으므로 참고. 롤백 후 재시작으로 해결했다고

Model Parallelism 
- MoE를 분산하는 Expert Parallelism. Give each expert its own accelerator. 
- pynvml 자세한 gpu 정보 제공

# 기타, 정리할 것
- <https://arxiv.org/html/2406.14491v1> Instruction-Augmented Corpora를 Pre-training했더니 성능이 좋더라.
- <https://bnmy6581.tistory.com/232> Textbooks are all you need 한글 요약
- <https://magazine.sebastianraschka.com/p/instruction-pretraining-llms> Raschka가 여러 기법 정리
- What is NUMA?