---
layout: wiki 
title: GPU Compute
tags: ["MLOps & HPC"]
last_modified_at: 2026/09/04 01:44:49
---

- [GPU 속도 비교](#gpu-속도-비교)
- [Platforms, Frameworks, Tools](#platforms-frameworks-tools)
- [SYCL](#sycl)
- [oneAPI](#oneapi)
- [OpenCL](#opencl)
- [ROCm](#rocm)
- [SPIR](#spir)

# GPU 속도 비교
DNA Factory에서 `python sft.py`를 했을 때 속도 비교. 작은 모델이라 attn에 따른 속도 차이가 없는 거 같다.

- B200 1장: /2816, 1.85s/it, 1h 30m, attn_implementation=sdpa
- B200 1장: /2816, 1.83s/it, 1h 25m, attn_implementation=eager
  - B200 2장: /1408, 1.98s/it, 45m, attn_implementation=sdpa
- H200 1장: /2828, 2.34s/it, 1h 50m, attn_implementation=kernels-community/vllm-flash-attn3
- H200 1장: /2828, 2.33s/it, 1h 50m, attn_implementation=sdpa
- H200 1장: /2828, 2.28s/it, 1h 47m, attn_implementation=eager
  - H200 2장: /1414, 2.47s/it, 58m, attn_implementation=kernels-community/vllm-flash-attn3
- RTX4080 SUPER 1장: /2804, 6.08s/it, 4h 45m, attn_implementation=kernels-community/vllm-flash-attn3
- RTX4080 SUPER 1장: /2804, 5.90s/it, 4h 35m, attn_implementation=sdpa
- RTX4080 SUPER 1장: /2804, 6.06s/it, 4h 45m, attn_implementation=eager

# Platforms, Frameworks, Tools
- NVIDIA: CUDA
- AMD: ROCm
- Khronos: OpenCL, Vulkan (replacement for OpenGL) → Kompute, SYCL
- Intel: oneAPI → SYCL (Khronos)

# SYCL
<img src="/images/2024/2020-05-sycl-landing-page-02a_2.jpg" width="90%">

It’s important to note that SYCL needs to rely on another tool as a ‘backend’ and there are, for example, CUDA and OpenCL backends. 

# oneAPI
<img src="/images/2024/6967d35c-9255-4cf5-b969-7bca7a19f6d1_1238x1060.png" width="60%">

Intel provides implementations of both SYCL (through the Intel DPC++ compiler) and of the APIs.

**oneAPI** ecosystem include:
- **DPC++** (Data Parallel C++): The primary oneAPI SYCL implementation, which includes the icpx/icx Compilers.
- **Nvidia & AMD Plugins**: These are [plugins extending oneAPI's DPC++ support to SYCL on Nvidia](https://developer.codeplay.com/products/oneapi/nvidia/home/) and AMD GPU targets.

# OpenCL
- OpenCL is more verbose than CUDA
- OpenCL separates Kernel code into files that are distinct from the host C/C++ in contrast to CUDA’s ‘single file’ approach.

Apple deprecated OpenCL in MacOS 10.14 (released in 2019) and now prompts developers to use Metal instead. AMD has de-emphasized support for OpenCL in favor of HIP.

# ROCm
AMD’s track record of support for their tools **does not completely inspire confidence**. AMD favored OpenCL for a period, then HCC but has now switched to HIP / HIPIFY.

# SPIR
SPIR stands for ‘Standard Portable Intermediate Representation’. It provides a common ‘intermediate representation’ for Kernels in OpenCL and Vulkan.
<img src="/images/2024/325ce5e8-d66e-4ee3-811f-aa71aa6c4f9e_1600x668.jpg" width="90%">

---
- CUDA’s (Nvidia, 2007) introduction was a major milestone in making GPGPU programming accessible.
- OpenCL (Khronos, 2009) has the widest support across platforms and is the most mature open standard but development has slowed, and it’s been abandoned/deemphasized by Apple and AMD.
- SYCL (Khronos, 2014) provides a way of deploying C++ across a range of platforms.
- ROCm (AMD, 2016) tries to obtain a degree of interoperability with CUDA and Nvidia hardware.
- oneAPI (mainly Intel, 2020) is effectively an enhancement of SYCL and provides a way of deploying a C++ code base across multiple platforms, including Nvidia GPUs.

[^fn-1]

[^fn-1]: <https://thechipletter.substack.com/p/demystifying-gpu-compute-software>