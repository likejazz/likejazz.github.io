---
layout: wiki 
title: LLM
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/08/07 10:10:37
---

<!-- TOC -->

- [비교](#비교)
- [NVIDIA Jetson LLM 성능](#nvidia-jetson-llm-성능)

<!-- /TOC -->

# 비교

| Model | Training Tokens | Vocab Size | Context Length | GPU Hours | Release Date |
| ----- | --------------- | ---------- | -------------- | --------- | ------------ |
| LLaMA 7B, 13B, 33B, 65B | 1T, 1.4T | 32K | 2K | 7B - 82,432 | 2023년 2월 |
| Llama 2 7B, 13B, 70B| 2T | 32K[^fn-1] | 4K | 7B - 184,320 | 2023년 8월 |
| 42dot LLM 1.3B | 2T | 50304 | 4K | 1.3B - 49,152 | 2023년 9월 |
| Mistral 7B | 8T | 32K | 8K | Around 200K | 2023년 9월 |
| Gemma 2B, 7B | 2T, 6T | 256K | 8K | | 2024년 2월 |
| Llama 3 8B, 70B | 15T | 128K | 8K | 8B - 1.3M | 2024년 4월 |
| Qwen2 0.5B, 1.5B, 7B, 72B | 7T (7B)  | 152K | 131K |  | 2024년 6월 |
| Gemma 2 9B, 27B | 8T, 13T | 256K | 8K | | 2024년 6월 |
| Mistral NeMo 12B | | 128K | 128K | | 2024년 7월 |
| Llama 3.1 8B, 70B, 405B | 15T+ | 128K | 128K | 8B - 1.46M | 2024년 7월 |

[^fn-1]: [Vocab은 다름](https://blog.gopenai.com/llama-1-and-llama-2-have-different-vocabularies-d82c5d22cca4)

# NVIDIA Jetson LLM 성능

![](/images/2024/jetson-3.jpg)

![](/images/2024/jetson-4.jpg)

![](/images/2024/jetson-5.jpg)

![](/images/2024/jetson-1.jpg)
