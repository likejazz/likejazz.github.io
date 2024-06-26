---
layout: wiki 
title: LLM
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/05/12 17:35:38
---

<!-- TOC -->

- [비교](#비교)
- [NVIDIA Jetson LLM 성능](#nvidia-jetson-llm-성능)

<!-- /TOC -->

# 비교

| Model | Training Tokens | Vocab Size | Context Length | GPU Hours | Release Date |
| ----- | --------------- | ---------- | -------------- | --------- | ------------ |
| LLaMA 1 7B, 13B, 33B, 70B | 1T, 1.4T | 32K | 2K | 7B - 82,432 | 2023년 2월 |
| LLaMA 2 7B, 13B, 70B| 2T | 32K[^fn-1] | 4K | 7B - 184,320 | 2023년 8월 |
| 42dot LLM 1.3B | 2T | 50304 | 4K | 1.3B - 49,152 | 2023년 9월 |
| Mistral 7B | 8T | 32K | 8K | Around 200K | 2023년 9월 |
| Gemma 2B, 7B | 2T, 6T | 256K | 8K | ? | 2024년 2월 |
| LLaMA 3 8B, 70B | 15T | 128K | 8K | 1.3M | 2024년 4월 |

[^fn-1]: [Vocab은 다름](https://blog.gopenai.com/llama-1-and-llama-2-have-different-vocabularies-d82c5d22cca4)

# NVIDIA Jetson LLM 성능

<img src="/images/2024/jetson-3.jpg" width="70%">

<img src="/images/2024/jetson-4.jpg" width="70%">

<img src="/images/2024/jetson-5.jpg" width="70%">

<img src="/images/2024/jetson-1.jpg" width="70%">

<img src="/images/2024/jetson-2.jpg" width="70%">
