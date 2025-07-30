---
layout: post
title: ! 'Tenstorrent Grayskull 리뷰'
tags: ["MLOps & HPC"]
last_modified_at: 2024/10/08 11:13:31
---

<div class="message">
Tenstorrent에서 나온 Grayskull 카드의 구현 방식을 살펴본다.
</div>

<small>
*Jul 22, 2024*
</small>

- [개요](#개요)
  - [tt-buda](#tt-buda)
  - [tt-metalium](#tt-metalium)
- [결론](#결론)

# 개요

<img src="https://lh3.googleusercontent.com/pw/AP1GczNVluoPRKP6lx36d_-vAbNye1wyWATjvYoWQYW-2EcpvjGB7rT-D3iaNWKEkGAK7WwKey9jQCIrecw4p0zwtzGS0VGAuquG-VGLod5CSJZYcKYfkMexeo0gax8Xmu9-CPx8KG44uX1zQGnZakgp1XButw=w1840-h1380-s-no-gm?authuser=0" width="80%">

tt-buda, tt-metalium 2개의 sw를 제공하는데, tt-buda는 기존 허깅페이스 모델을 돌릴 수 있게 하며, tt-metalium은 완전히 처음부터 작성하는 패키지다. 

다행히 이 둘 모두 오픈소스로 진행되고 있으므로 코드를 자유롭게 수정하거나 커뮤니티의 도움을 받기가 수월한 편으로 다른 제품들이 원래 설치된 예제 외에는 다른 시도를 하기 어려운 것과는 대조적이다. 하지만 아직 커뮤니티가 많이 활성화 되어 있진 않다. tt-buda의 최신 버전 docker 컨테이너 다운로드 수는 고작 150여 회에 불과하다.

놀랍게도 오히려 tt-metalium은 많은 이들이 관심을 보인다. 사람들이 생각보다 바닥부터 작성하는 라이브러리에 흥미를 느끼는 것 같고, 이슈나 PR이 꽤 활발하다.

## tt-buda

> The TT-BUDA software stack can compile models from several different frameworks and execute them in many different ways on Tenstorrent hardware.

torch 변수를 읽어들이며 torch 모델을 활용할 수 있는 구조다. 내부적으로는 다음과 같이 변환하여 사용한다.

```python
# Run single inference pass on a PyTorch module, using a wrapper to convert to PyBuda first
output = pybuda.PyTorchModule("direct_pt", PyTorchTestModule()).run(input1, input2)
```

이렇게 하면 `<class 'pybuda.tensor.TensorFromPytorch'>` 로 타입을 변환하여 계산한다. 직접 세션이 실행되기 때문에 마치 tensorflow를 보는 느낌이다. torch가 JIT을 구현했는데 다시 옛날 tensorflow의 session 기반으로 돌아간 느낌. 게다가 다음과 같이 세션 실행 중에 다운이 되기도 했다.

<img src="/images/2024/Screenshot 2024-05-07 at 3.45.27 PM.jpg" width="70%">

이렇게 중간에 세션이 다운됐는데 아무리 기다려도 이후에 진행이 되지 않았다. shutdown 시에도 다음과 같이 멈춤 상태에서 더 이상 진행이 되지 않았다.

```
2024-05-07 17:31:48.352 | DEBUG    | pybuda.run.impl:_shutdown:1265 - PyBuda shutdown
```

다른 제품들과 마찬가지로 허깅페이스 인터페이스를 동일하게 제공한다.

```python
# GPT2 demo script - Text generation
import pybuda
from pybuda.transformers.pipeline import pipeline as pybuda_pipeline
from transformers import GPT2Config, GPT2LMHeadModel, GPT2Tokenizer
```

GPT2 데모 코드를 보면 config, tokenizer, 모델 로딩 까지 기존 허깅페이스를 그대로 사용하고 pipeline으로 실행시에는 자체 개발한 pybuda_pipeline을 이용하는 식이다. 모델을 별도로 변환하지 않고 허깅페이스 모델을 그대로 사용한다는 점에서 다른 제품들 보다는 좀 더 편리하다. 단, 앞서와 마찬가지로 tensorflow가 구동되는 느낌의 verbose한 세션 실행 시간이 앞뒤로 길게 진행된다. warm-up 시간이 매우 길다. 게다가 다음과 같이 balancing 작업이 진행되는데 이게 뭔지 몰라도 시간이 꽤 걸린다.

```python
2024-05-07 18:01:25.713 | INFO     | Balancer        - Balancing 13% complete.
2024-05-07 18:01:27.696 | INFO     | Balancer        - Balancing 20% complete.
2024-05-07 18:01:31.174 | INFO     | Balancer        - Balancing 24% complete.
2024-05-07 18:01:36.199 | INFO     | Balancer        - Balancing 34% complete.
2024-05-07 18:01:39.238 | INFO     | Balancer        - Balancing 40% complete.
2024-05-07 18:01:42.268 | INFO     | Balancer        - Balancing 45% complete.
2024-05-07 18:01:46.591 | INFO     | Balancer        - Balancing 52% complete.
2024-05-07 18:01:49.272 | INFO     | Balancer        - Balancing 58% complete.
2024-05-07 18:01:51.928 | INFO     | Balancer        - Balancing 61% complete.
```

결과는 잘 출력되지만 warm-up 시간이 워낙 길기 때문에 CLI에서 시간 측정은 어렵다. 추후 웹 서비스로 구현한 후에야 제대로 시간 측정이 가능할 것 같다. 게다가 아직 GPT-2, GPT-Neo 정도만 지원하고 메모리가 작기 때문에 LLM 용도로는 적합하지 않고 사실상 cv 용도로만 쓰일거 같다.

## tt-metalium

허깅페이스 모델을 그대로 돌려주는 tt-buda와 달리 이쪽은 바닥부터 하나씩 쌓아 올려야 하는 라이브러리다. CUDA 또는 cuDNN을 지향하는 라이브러리로 보인다. 그런데 생각보다 이슈나 PR이 꽤 활발하고 많은 이들이 참여하고 있다. tt-buda가 tensorflow의 느낌이라면 tt-metalium은 pytorch에 가깝다. 어차피 당장 서비스에 활용할게 아니라서 아무래도 사용 편의성이 높은 이쪽에 사람들이 많이 몰리는거 같다. 게다가 디바이스를 low-level로 컨트롤 할 수 있기 때문에 매우 hackable하게 여러 실험을 할 수 있다.

<img src="/images/2024/Screenshot 2024-05-07 at 7.00.28 PM.png" width="70%">

문제는 docker 이미지를 제공하는 tt-buda와 달리 직접 설치해야 해서 까다롭다. 게다가 우분투 20.04 기준으로 설치 가이드를 제공하고 있어 일부러 예전 환경인 20.04으로 맞춰서 설치를 진행해야 했다. 다행히 누군가 [docker 설치 가이드를 제공하는 PR을 진행 중](https://github.com/tenstorrent/tt-metal/tree/mchiou/7944-add-dockerfile-and-installation-steps)이라 이 repo에 들어가서 dockerfile만 별도로 참고해서 진행했다. 아마 조만간 merge 될 것으로 보인다.

```python
import torch
import ttnn

device_id = 0
device = ttnn.open_device(device_id=device_id)

torch_input_tensor = torch.rand(2, 4, dtype=torch.float32)
input_tensor = ttnn.from_torch(torch_input_tensor, dtype=ttnn.bfloat16, layout=ttnn.TILE_LAYOUT, device=device)
output_tensor = ttnn.exp(input_tensor)
torch_output_tensor = ttnn.to_torch(output_tensor)

ttnn.close_device(device)
```

코드 구조를 보면 device를 open_device하는 것 외에는 기본적으로 pytorch와 동일하게 동작한다. 세션이 실행되는 tt-buda에 비해 훨씬 더 직관적인 방식이고, 그래서 다양한 방식으로 실험하는 사람들이 많은듯 하다.

아쉽게도 제공되는 데모는 grayskull에서는 메모리 오류로 제대로 실행되지 않았다.

```
E           RuntimeError: TT_FATAL @ /home/xxxx/tt-metal/tt_eager/tt_dnn/op_library/bmm/bmm_op.cpp:760: program_config.compute_with_storage_grid_size.x <= input_tensor_a.device()->compute_with_storage_grid_size().x
E           info:
E           Matmul grid size exceeds maximum device compute grid size!
```

# 결론

tt-buda의 경우 허깅페이스 모델을 컨버팅 없이도 그대로 실행시켜주는 점이 인상적이었다. 그러나 구동 가능한 모델이 제한적이고, 앞뒤로 warm-up 시간이 너무 길어 제대로 환경을 갖춘 프로덕션 환경에서나 제한적으로 사용이 가능해 보였다. tensorflow의 좀 더 무거운 버전을 보는 느낌이다. 다행히 tt-metal을 추가로 제공해줘서 여기서는 JIT 방식으로 간편하게 활용할 수 있지만 그래도 device를 가져와서 open하는 방식이 조금 불편했고, 그나마 지원하는 LLM인 Falcon 7B 데모도 메모리 오류로 실행되지 않았다. 논문 등을 쓸 때 하나씩 실험하는 용도로 활용할 수 있을 것 같지만 이 또한 GPU에 비해 잇점이 별로 없다. 그나마 tt-metal은 커뮤니티가 매우 활성화 되어 있어 사람들이 다양한 방식으로 응용을 시도하고 있다. 주요 라이브러리가 모두 오픈소스로 진행되고, 네임드가 대표로 있는 회사다 보니 이런 부분은 잘 진행되고 있는 것으로 보인다.
