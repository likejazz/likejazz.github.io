---
layout: post
title: ! 'Think about mid-sized LLM intelligence'
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/08/12 14:21:13
---

<div class="message">
Evaluate various mid-sized (7B ~ 10B) models based on Llama or similiar architecures, and add explanations to the results.
</div>

<small>
*Aug 12, 2024*
</small>

- [Overview](#overview)
- [Benchmark Dataset](#benchmark-dataset)
  - [MMLU](#mmlu)
  - [KMMLU](#kmmlu)
- [Evaluation](#evaluation)
- [Result](#result)
- [Explanation](#explanation)

# Overview
Llama 3.1 has wowed the world with its performance, and luckily this awesome model is freely available to everyone. Thanks to Zuck, and meta. I've tested several open LLMs (base model only), and let's see which one is the best.

# Benchmark Dataset
## MMLU
There are many benchmark datasets out there, but I've carefully considered and selected the best one for comparison, and I will focus on just one metric, MMLU.

**MMLU (Massive Multitask Language Understanding)** is a new benchmark designed to measure knowledge acquired during pretraining by evaluating models exclusively in zero-shot and few-shot settings.

MMLU questions are as follows:

```
What is the most common way for a virus to kill a cell?
A. Dissolves the cellular membrane
B. Induces apoptosis via caspases
C. Fragments cellular DNA
D. Totally blocks cellular transcription
Answer:
```

The answer is B. lm-eval script checks the logits of candidates A, B, C, and D to select the best probs of the 4 characters, and check the accuracy of how many answers were selected. MMLU consists of 16k multiple-choise questions, and it covers 57 subjects across STEM, the humanities, the social sciences, and more. It ranges in difficulty from an elementary level to an advanced professional level, and it tests both world knowledge and problem solving ability.

## KMMLU
Let's use one more benchmark for non-english task: KMMLU. As you might have guessed, it's based on MMLU task, but it's not just a simple translation, it's completely rewritten for Korean questions. The questions are as follows:

```
흑색화약에 대한 설명으로 옳은 것은?
A. 화염과 마찰에 둔감하다.	
B. 흡습을 피하면 오래 저장할 수 있다.	
C. 발화할 때 후가스가 좋아 터널공사에 주로 사용한다.
D. 인화성이 있으므로 흡습되어도 발화에는 지장이 없다.
정답: 
```

The answer is B. **KMMLU** consists of 35k expert-level multiple-choice questions across 45 subjects ranging from humanities to STEM. Unlike other Korean benchmarks that are translations of English benchmarks, KMMLU is collected from original Korean exams, capturing linguistic and cultural aspects of the Korean language. 

According to the paper, Qwen-72B achieves 50.83%, far below the average human performance of 62.6%. GPT-4 and HyperCLOVA X, achieve 59.95% and 53.40%, respectively. But for now, Qwen2-72B achieves whopping 65.2%, Qwen2-7B also achieves 49.05%.

# Evaluation

I've only evaluated base model. Because it's an auto-regressive model, it does not require any special training methods such as SFT, RLHF, etc, unlike intruct model. Measuring performance on the base model only measures knowledge acquired during pretraining.

# Result
I've evaluated several open LLMs around 7B ~ 10B (mid-sized LLMs), and the result is shown below:

| Model | MMLU | KMMLU |
| --- | ----- | ----------- |
| meta-llama/Llama-2-7b-hf | 45.7 | 24.35 | 
| meta-llama/Meta-Llama-3-8B | 65.04 | 40.03 |
| **<mark style='background-color: #fff5b1'>meta-llama/Meta-Llama-3.1-8B</mark>** | <mark style='background-color: #fff5b1'>65.23</mark> | <mark style='background-color: #fff5b1'>41.06</mark> |
| google/gemma-7b | 62.98 | 40.83 |
| **<mark style='background-color: #fff5b1'>google/gemma-2-9b</mark>** | <mark style='background-color: #fff5b1'>70.28</mark> | <mark style='background-color: #fff5b1'>47.05</mark> |
| Qwen/Qwen-7B | 58.44 | 35.36 (fix token error) | 
| Qwen/Qwen1.5-7B | 60.46 | 38.57 | 
| **<mark style='background-color: #fff5b1'>Qwen/Qwen2-7B</mark>** | **<mark style='background-color: #fff5b1'>70.55</mark>** | **<mark style='background-color: #fff5b1'>49.05</mark>** | 
| mistralai/Mistral-7B-v0.1 | 62.39 | 37.38 | 
| mistralai/Mistral-7B-v0.3 | 62.33 | 36.49 | 
| upstage/SOLAR-10.7B-v1.0 | 64.20 | 39.04 | 
| yanolja/EEVE-Korean-10.8B-v1.0 | 63.25 | 42.68 |
| KISTI-KONI/KONI-Llama3-8B-20240630 | 62.68 | 0 |
| KISTI-KONI/KONI-Llama3-8B-Merged-20240724 | 65.55 | 0 |
| beomi/Llama-3-KoEn-8B | 52.44 | 40.66 |
| beomi/Llama-3-Open-Ko-8B | 56.29 | 40.50 |
| LGAI-EXAONE/EXAONE-3.0-7.8B-Instruct (instruct) | 64.12 | 44.65 |

# Explanation
- **Qwen2** is most outstanding LLM in both english and korean. The early version (first version) was very tricky and it was too difficult to handle model and tokenizer, but Qwen2 is supported by most frameworks and it's no longer inconvenient. My guess is that Chinese model have overwhelmingly superior training dataset compared to US or Europe models.
- **Gemma 2** is second best model. As expected, it's Google. but Gemma still have compatibility issues because it's too slow and its inference is very tricky.
- **Llama 3.1** is the most compatible model supported by all frameworks. If you don't care about improving your score by 1~2 points, Llama model is always the best choice.
- **SOLAR** is based on Mistral and is merged into 10B, making it more powerful due to its increased size. However, SOLAR (open model) model is lack of korean ability, so **EEVE** applied vocab extention to SOLAR, and continued learning korean dataset to improve its korean performance. Korean score improved by more than 3 points, but unfortunately, english score dropped by 1 point.
- **KONI-Llama3**, **Llama-3-KoEn**, **Llama-3-Open-Ko** is contintued learning models based on Llama 3. However, despite additional training on 60GB ~ 200GB korean dataset, they failed to show korean intelligence. In KMMLU, they led Llama 3 by 0.5 point, but in MMLU they dropped more than 10 points, a shocking result. Unfortunately, without continued learning, they could have performed better.
- **EXAONE-3.0** is new one, and most importantly, it's not based on Llama, but trained from scratch. LG hasn't released a base model, so I've only evaluated a intruct model, and its performance is awesome. English score is almost identical to Llama 3.1, and Korean performance is more than 4 points higher than Llama 3.1. It's a shame that it can't be used for commercial services due to licensing issues.