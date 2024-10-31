---
layout: wiki 
title: CPT 성능 향상 기법
tags: ["LLM Training"]
last_modified_at: 2024/10/31 09:23:32
---

- [논문](#논문)

# 논문
**Balancing Continuous Pre-Training and Instruction Fine-Tuning: Optimizing Instruction-Following in LLMs**  

2.2 Instruction Residuals:  
$$\Theta_r^{v1} = \theta_i^{d1v1} - \theta_b^{d1}$$  
$$\theta_i^{d1d2v1} = \theta_b^{d1d2} \oplus \Theta_r^{v1},$$

L3b를 라마3 base, L3i는 라마3 instruct, 3Lr (instruction residual)이 바로 라마3로 추출한 값이다. d1은 기존 base 데이터셋, v1는 기존 instruct 데이터셋, d2가 CPT를 위한 데이터셋이다. L3i - L3b 해서 추출한 weights를 v1을 학습한 CPT 모델에 element-wise add 했다.

---
**Examining Forgetting in Continual Pre-training of Aligned Large Language Models**

instruct에 cpt를 진행했다. 이 경우 굳이 결과를 보지 않아도 당연히 aligned가 망가질거라 예상할 수 있다. cpt이후 instruct 모델로 실험한 결과와 비교가 있으면 좋았을텐데, instruct를 재현할 수 없다.

---
- <https://arxiv.org/html/2406.14491v1> Instruction-Augmented Corpora를 Pre-training했더니 성능이 좋더라.
- <https://bnmy6581.tistory.com/232> Textbooks are all you need 한글 요약
- <https://magazine.sebastianraschka.com/p/instruction-pretraining-llms> Raschka가 여러 기법 정리
