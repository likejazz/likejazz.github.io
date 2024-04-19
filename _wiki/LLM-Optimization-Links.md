---
layout: wiki 
title: LLM Optimization Links
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/02/12 12:29:33
---

- [Optimization](#optimization)
  - [Quantization](#quantization)
  - [Pruning](#pruning)
- [Model](#model)
- [Implementation](#implementation)
- [Continuous Batching](#continuous-batching)
- [Architectural Innovations](#architectural-innovations)
- [기타](#기타)

# Optimization
- <https://huggingface.co/blog/optimize-llm>  
허깅페이스가 정리한 최적화 in Production
  - <https://huggingface.co/docs/transformers/llm_tutorial_optimization>
- <https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/>  
NVIDIA 최적화
  - <https://docs.nvidia.com/deeplearning/transformer-engine/user-guide/examples/fp8_primer.html> Using FP8 with Transformer Engine, H100
- <https://lilianweng.github.io/posts/2023-01-10-inference-optimization/>  
최신 논문 기준으로 좀 더 low-level 최적화 최신 연구 소개. Distillation, Quantization, Pruning, Sparsity(esp. Attention, MoE), Architectural(FlashAttention은 언급 x)
- <https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices>
- <https://hamel.dev/notes/llm/inference/03_inference.html> Optimizing latency 개인 노트. 정리가 잘 되어 있음.
- <https://intellabs.github.io/distiller/> Model Compression 좋은 라이브러리였으나 보안 이슈로 중단
- <https://maartengrootendorst.substack.com/p/which-quantization-method-is-right> Which quantization is right for you?


## Quantization
- <https://huggingface.co/blog/overview-quantization-transformers>
- Int-3 and beyond <https://nolanoorg.substack.com/p/int-4-llama-is-not-enough-int-3-and>
  - bitsandbytes, 4-bit quantization and QLoRA <https://huggingface.co/blog/4bit-transformers-bitsandbytes>
- <https://lightning.ai/blog/4-bit-quantization-with-lightning-fabric/> 4-bit Lightning

## Pruning
- Song Han 교수 연구

# Model
- <https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/>  
트랜스포머 모델 최신 연구 정리

# Implementation
- <https://kipp.ly/transformer-inference-arithmetic>  
추론 연산 속도 계산

# Continuous Batching
- <https://www.anyscale.com/blog/continuous-batching-llm-inference>

# Architectural Innovations
- RoPE
- Multi-Query Attention(MQA)
- Grouped-Query Attention(GQA)
- PagedAttention in vLLM
  - <https://velog.io/@jminj/Paged-Attention>

# 기타
- <https://www.huaxiaozhuan.com/%E5%B7%A5%E5%85%B7/huggingface_transformer/chapters/1_tokenizer.html> Tokenizer 중국 문서
