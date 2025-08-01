---
layout: post
title: ! 'LLM 아키텍처 비교'
tags: ["Large Language Model (LLM)"]
last_modified_at: 2025/08/02 02:34:09
---

<div class="message">
<a href="https://magazine.sebastianraschka.com/p/the-big-llm-architecture-comparison">The Big LLM Architecture Comparison</a>을 읽고 현대 LLM 아키텍처 설계를 살펴보면서 중요한 내용을 정리해봅니다.
</div>

<small>
*Aug 1, 2025*
</small>

- [개요](#개요)
- [내용](#내용)
  - [딥시크 V3/R1](#딥시크-v3r1)
  - [OLMo 2](#olmo-2)
  - [젬마 3](#젬마-3)
    - [젬마 3n](#젬마-3n)
  - [미스트랄 스몰 3.1](#미스트랄-스몰-31)
  - [라마 4](#라마-4)
  - [큐원3](#큐원3)
  - [SmolLM3](#smollm3)
  - [키미 K2](#키미-k2)
- [마치며](#마치며)


# 개요

그간 LLM 아키텍처에 대해 좋은 글을 많이 올려주던 세바스찬 라슈카가 이번에 본인의 서브스택에 [유명 LLM 아키텍처 설계 전반에 대해 정리해보는 좋은 글](https://magazine.sebastianraschka.com/p/the-big-llm-architecture-comparison)을 올렸습니다. 이 내용을 꼼꼼히 살펴보고 중요한 부분을 따로 정리해봅니다.

# 내용

GPT 아키텍처가 등장한 지도 벌써 7년이 지났습니다. 여전히 최신 LLM 대부분이 최초의 GPT에서 크게 달라진 점이 없다는 점은 기술의 빠른 발전 속도를 감안할 때 꽤 놀라운 일입니다. 물론 위치 임베딩이 회전(RoPE) 임베딩으로 발전했고, 멀티 헤드 어텐션(MHA)은 GQA<sup>Grouped-Query Attention</sup>로, 활성화 함수가 GELU에서 보다 효율적인 SwiGLU로 발전하긴 했지만 전반적인 트랜스포머 구조 자체는 예나 지금이나 크게 다르지 않죠.

물론 LLM의 성능은 아키텍처 뿐만 아니라 데이터와 학습 기법, 그리고 하이퍼파라미터도 크게 영향을 끼칩니다. 그러나 아키텍처를 살펴보는 것은 여전히 중요한 일이죠. 다음 그림은 세바스찬 라슈카가 정리한 2025년 최신 LLM의 아키텍처 도식화입니다.

![](https://github.com/user-attachments/assets/99219ffe-df78-423c-a9aa-340d4a06aa27)

그렇다면 최신의 유명한 모델들을 하나씩 살펴보도록 하죠.

## 딥시크 V3/R1

가장 먼저 소개할 모델은 딥시크입니다. 전세계에 화제를 몰고 온 딥시크 R1은 기본적으로 V3와 동일한 구조를 지니고 있습니다. 특히 [MoE 구조에 대해서는 예전에도 한 번 상세하게 정리](/deepseek-v3)한 적이 있죠. 이외에도 MLA를 딥시크의 가장 큰 차별점으로 들 수 있는데, MHA의 높은 연산량을 KV를 공유하는 방식으로 대폭 줄인 GQA의 구조는 다들 잘 알다시피 다음과 같습니다.

<img width="80%" src="https://github.com/user-attachments/assets/4e1dcd18-bef5-4d19-96e9-a94b592c2c7e">

여기서 한발 더 나아가 딥시크 팀은 KV를 저차원으로 압축하여 (Q는 학습시에만 압축) 효율성을 더욱 높였습니다. 그게 바로 MLA죠. 

<img width="80%" src="https://github.com/user-attachments/assets/99b27102-54cb-4658-80ce-b723ef2646c8">

물론 이 기술은 딥시크 V2[^fn-v2]에서 이미 소개한 바 있습니다. 당시에도 화제가 됐던 점은 GQA가 MHA보다 (당연하게도) 성능이 떨어지는 것에 비해 MLA는 오히려 성능이 더 뛰어남을 보였다는 점인데요. 

[^fn-v2]: <https://arxiv.org/abs/2405.04434>

<img width="70%" src="https://github.com/user-attachments/assets/0af71e64-e552-4e20-b576-5335ae1dfb15">

최근 등장한 일부 모델들의 경우 성능 개선을 보이지 못하고 차별화를 위해 아키텍처만 엉뚱하게 바꾸는 경우가 많은데, 딥시크는 그런 요식 행위가 아니라 확실하게 성능 개선을 보였다는 점에서 주목할 만합니다.

MoE 구조에 대해서는 이전에 이미 상세하게 정리[^fn-moe]한 바 있으며 Expert 수가 촘촘하게 256개나 된다는 점, 그리고 이 중 9개가 매 번 활성화 된다는 점이 (1개는 Shared Expert로 항상 활성화) 특징적이고 그래서 전체 617B에서 37B가 활성화 됩니다. Shared Expert에 대해서는 딥시크가 이전 논문에서 소개[^fn-shared]한 바 있으며 마찬가지로 성능 향상을 보였습니다. 논문에서는 공통적으로 반복되는 패턴은 여러 전문가가 개별적으로 학습할 필요 없이 Shared Expert가 전담할 수 있기 때문에 중복을 완화해 성능 향상을 보인다고 소개하고 있습니다.

[^fn-moe]: <https://likejazz.com/deepseek-v3>
[^fn-shared]: <https://arxiv.org/abs/2401.06066>

이외에도 딥시크는 다양한 부분에서 혁신적인 아키텍처를 채택했고, 또 왜 그 아키텍처여야 하는지를 여러 편의 논문을 통해 명확하게 설명하고 있기 때문에 딥시크가 전세계에 불러 일으킨 엄청난 반향은 결코 허상이 아니라고 할 수 있습니다.

## OLMo 2
ELMo로 BERT와 함께 언어 이해 모델 1세대 주자이기도 했던 앨런 AI 연구소에서 내놓은 OLMo 시리즈는 (여전히 이름이 비슷하네요) 학습 데이터와 코드를 거의 모두 공개했다는 점에서 주목할 만합니다. 모델 성능은 그다지 뛰어나지 않지만 누구나 재현할 수 있을 정도로 상세하게 기술의 세부사항을 공개했다는 점은 다른 오픈소스 기업들보다 훨씬 더 진일보한 모습을 보이고 있죠.

OLMo 2는 LayerNorm 대신 더 단순한 RMSNorm을 사용합니다. 이는 다른 최신 모델도 비슷하지만 차이점은 레이어의 위치입니다.

<img width="80%" src="https://github.com/user-attachments/assets/d07b969b-1d20-4b26-8514-e169f8253dd9" />

원래 트랜스포머에서는 정규화 계층을 뒤에 배치했습니다. 이를 Post-LN 또는 Post-Norm이라고 하죠. 이후 GPT부터 대부분의 LLM은 정규화 계층을 앞쪽으로 이동했는데 이를 Pre-LN 또는 Pre-Norm이라고 합니다. 얼마전 [네이버에서 이 부분의 차이점을 잘 설명](https://clova.ai/tech-blog/%ED%9D%94%EB%93%A4%EB%A6%BC-%EC%97%86%EB%8A%94-%EC%95%88%EC%A0%95%EC%84%B1-peri-ln%EC%9C%BC%EB%A1%9C-%ED%95%99%EC%8A%B5-%EB%B0%9C%EC%82%B0%EC%9D%84-%EB%A7%89%EB%8B%A4)한 바 있습니다. 그런데 OLMo 2는 Pre-LN을 사용하던 최근 트렌드와 달리 다시 Post-LN을 채택했습니다. LayerNorm대신 RMSNorm을 사용했기 때문에 정확히는 Post-Norm입니다. 이렇게 위치를 이동한 이유는 학습 안정성이 더 뛰어났기 때문이라고 합니다. 그런데 OLMo 2는 정규화 계층만으로 성능 차이를 보이기보다 이와 함께 GQA에 QK-Norm을 함께 적용했습니다.

```python
class GroupedQueryAttention(nn.Module):
    def __init__(
        self, d_in, num_heads, num_kv_groups,
        head_dim=None, qk_norm=False, dtype=None
    ):
        # ...

        if qk_norm:
            self.q_norm = RMSNorm(head_dim, eps=1e-6)
            self.k_norm = RMSNorm(head_dim, eps=1e-6)
        else:
            self.q_norm = self.k_norm = None

    def forward(self, x, mask, cos, sin):
        b, num_tokens, _ = x.shape

        # Apply projections
        queries = self.W_query(x) 
        keys = self.W_key(x)
        values = self.W_value(x) 

        # ...

        # Optional normalization
        if self.q_norm:
            queries = self.q_norm(queries)
        if self.k_norm:
            keys = self.k_norm(keys)

        # Apply RoPE
        queries = apply_rope(queries, cos, sin)
        keys = apply_rope(keys, cos, sin)

        # Expand K and V to match number of heads
        keys = keys.repeat_interleave(self.group_size, dim=1)
        values = values.repeat_interleave(self.group_size, dim=1)

        # Attention
        attn_scores = queries @ keys.transpose(2, 3)
        # ...
```

코드를 살펴보면 QK에 RMSNorm을 적용해서 사실상 어텐션 구조에 큰 변화를 준 셈인데 이 때문에 학습 안정성이 반드시 정규화 계층의 위치 차이만이라고 얘기하기는 어렵게 됐습니다. 어쨌든 OLMo 2는 이렇게 정규화 구조를 변경한 덕분에 학습시 기존에 비해 loss spikes가 훨씬 더 줄어들었고, 학습 안정성이 더 높아졌다고 얘기합니다.

![](https://github.com/user-attachments/assets/559b7d19-462b-4449-ad32-ad76ec21adbd)

## 젬마 3

젬마 3는 계산 비용을 줄이기 위해 어텐션에 슬라이딩 윈도우 어텐션을 사용합니다. 무엇보다 컨텍스트가 길어질 때 kvcache의 크기를 큰 폭으로 줄일 수 있습니다.

<img src="https://github.com/user-attachments/assets/71f773ea-77e8-4b5a-b116-d8b8d74d3437" width="80%">

슬라이딩 윈도우 어텐션은 마치 CNN처럼 크기를 제한하는 로컬 어텐션 형태로 동작합니다.

<img src="https://github.com/user-attachments/assets/8089e0b7-7b71-4350-8311-59b040c6537d" width="80%">

젬마 3는 GQA에 슬라이딩 윈도우 어텐션을 함께 적용했습니다. 물론 일반적인 어텐션도 여전히 사용하며 5:1의 비율로 섞어서 사용합니다. 이와 함께 정규화 계층을 Pre-Norm과 Post-Norm에 모두 배치했습니다. 즉 2번씩 정규화를 진행하며, OLMo 2와 마찬가지로 QK-Norm도 함께 적용하여 정규화를 매우 빈번하게 진행하는 모델입니다.

### 젬마 3n

구글에서 젬마 3와는 별도로 발표한 젬마 3n은 엣지에서 효율적으로 실행되는 것을 목표로 하는 소형 모델입니다. 메모리가 부족한 엣지를 위해 Per-Layer Embedding(PLE) 계층을 두어 이를 offloading할 수 있게 설계했습니다. 또한 MatFormer[^fn-mat] 구조를 도입했는데 더 작은 모델로 분할할 수 있는 아키텍처로, 추론시 큰 모델 대신 분할된 작은 모델을 별도로 실행할 수 있습니다.

[^fn-mat]: <https://arxiv.org/abs/2310.07707>

## 미스트랄 스몰 3.1

젬마 3가 슬라이딩 윈도우 어텐션을 활용한 것과 달리 미스트랄은 3.1부터 기존에 적용했던 슬라이드 윈도우 어텐션을 더 이상 사용하지 않습니다. 대신 GQA만 적용했으며 이는 메모리 절감을 포기하는 대신, 플래시 어텐션 같은 최적화 코드를 사용할 수 있게 하여 추론 시간을 더 줄이는 선택을 한 것으로 보입니다.

## 라마 4

라마 4가 이전 버전과 가장 두드러지는 특징은 MoE를 채택했다는 점입니다. 게다가 딥시크가 MLA라는 독창적인 어텐션을 사용하는데 반해 보다 범용적인 GQA를 그대로 사용합니다. Expert의 수도 128개로 좀 더 전통적인 MoE 구조에 가깝습니다. 이외에도 딥시크가 처음 3개를 제외하면 모두 MoE 레이어로 구성된 반면, 라마 4는 MoE와 Dense가 계속해서 번갈아 사용됩니다.

<img width="80%" src="https://github.com/user-attachments/assets/ef28222d-86e8-47b2-9362-beb4b8a23002">

## 큐원3

큐원3는 0.6B부터 235B MoE까지 다양한 사이즈를 제공합니다. 게다가 좋은 성능을 보여주기 때문에 딥시크와 함께 중국의 인공지능 기술력을 상징하는 모델이라고 할 수 있죠. 가장 작은 모델은 0.6B에 불과하기 때문에 메모리를 매우 적게 차지하지만 반면 트랜스포머 블록 갯수가 더 많기 때문에 실제로는 라마 3.2 1B보다 생성 속도는 조금 더 느립니다.

<img width="80%" src="https://github.com/user-attachments/assets/0a8eeb4b-2cb4-420a-8966-0b47e119cd92">

이 속도는 라슈카가 직접 PyTorch로 구현[^fn-py]하고 A100에서 돌린 결과라고 하네요.

[^fn-py]: <https://github.com/rasbt/LLMs-from-scratch/tree/main/ch05/11_qwen3>

큐원3는 MoE 버전도 제공합니다.

<img width="80%" src="https://github.com/user-attachments/assets/9b0fc639-3b48-4d25-9d7b-4085eb8a8711">

딥시크와 차이라면 더 이상 Shared Expert를 사용하지 않는다는 점인데요. 2.5 버전에서 이미 사용했던터라 굳이 제거한 이유가 궁금했는데 Junyang Lin의 답변[^fn-lin]에 따르면 다음과 같습니다.

[^fn-lin]: <https://x.com/JustinLin610/status/1947364862184853626>

<img width="70%" src="https://github.com/user-attachments/assets/2843481f-ad8b-4d86-a48b-d67a880e3dbd">

"Shared Expert로 인한 명확한 성능 개선 효과를 찾지 못했고, 추론 최적화에 문제가 있을까 우려하고 있다."고 합니다. 논문 등에서 Shared Expert로 인한 차이를 실제로 비교해서 보였던 것은 아니기에 그간 큐원의 높은 위상에 비추어 볼 때 조금은 실망스러운 답변입니다.

## SmolLM3

SmolLM3는 인기있는 모델은 아니지만 굳이 여기서 함께 언급한 이유는 OLMo 2와 마찬가지로 기술의 세부사항을 매우 상세하게 공개했기 때문입니다. 특히 함께 공개한 청사진은 유용한 내용들을 정말 잘 정리해둬서 저희도 출력해서 매일 참고하고 있을 정도입니다.

![](https://github.com/user-attachments/assets/0bb52ebd-f982-43a8-ae06-5f62d182d944)

SmolLM3가 다른 모델과 두드러지는 차이점이 있다면 NoPE(No Positional Embeddings)를 적용했다는 점인데요. 말 그대로 더 이상 위치 정보를 주입하지 않는 것인데, 모델 구조에는 이미 암묵적인 방향 감각이 내재되어 있기 때문에 일반적인 경사 하강법 기반으로 학습 시 위치 정보 없이도 이를 충분히 학습할 수 있다고 얘기합니다.

<img src="70%" src="https://github.com/user-attachments/assets/f6c21309-aafe-4661-9ba8-a0eb1e5fb9da">

실제로 NoPE 논문[^fn-nope]에서는 길이가 길어져도 성능이 저하되는 정도가 기존에 위치 정보를 주입하던 방식에 비해 훨씬 적다고 얘기합니다. 물론 이 논문의 결과는 1B 이하의 매우 작은 모델의 경우이고 큰 모델에서 과연 어떤 성능을 보이는지는 공개된 바가 없습니다. 그래서인지 SmolLM3에서도 4번째 레이어에만 조심스럽게 NoPE를 적용하고 있습니다.

[^fn-nope]: <https://arxiv.org/abs/2305.19466>

## 키미 K2

딥시크와 함께 문샷AI는 키미 K2 모델로 또 한 번 세상을 놀라게 했습니다. 딥시크가 671B인데 반해, 키미 K2의 매개변수는 무려 1T를 넘어서는 것도 인상적이죠. 특히 학습 시 AdamW가 아닌 Muon이라는 생소한 이름의 옵티마이저를 사용했는데, 1T가 넘는 대규모 모델에 새로운 옵티마이저를 과감히 도입한 건 무척 놀라운 일입니다. 물론 그전에 문샷AI에서는 Muon이 확장가능하다[^fn-muon]라는 내용의 논문을 이미 발표한 적이 있긴 합니다.

[^fn-muon]: <https://arxiv.org/abs/2502.16982>

OLMo 2가 정규화를 활용해 그랬던 것처럼 키미 K2는 Muon 옵티마이저를 적용해 loss spikes를 줄였다고 얘기합니다. 모델 자체가 1T가 넘기 때문에 이를 보이는 것 조차 쉽지 않은데, 사실 키미 K2는 2025년 상반기 기준 지금까지 공개된 가장 큰 LLM이기도 합니다.

<img width="80%" src="https://github.com/user-attachments/assets/8a360ad9-636d-46d5-aa46-67a658a867df">

MoE 구조에서 이미 Expert를 촘촘하게 배치했던 딥시크에서 한 발 더 나아가 무려 384개의 Expert로 더욱 촘촘한 배치를 자랑합니다. 또한 딥시크와 동일한 MLA를 적용했습니다. 어텐션 헤드 개수가 다른 점 외에는 MoE 구조가 딥시크와 동일합니다. 사실 키미는 k1.5 버전도 대단했습니다. 하지만 모델을 오픈소스로 공개하지 않았고, 논문[^fn-kimi]을 발행한 시점이 마침 딥시크 R1이 공개되던 날이라 완전히 묻혀버리고 말았죠. 문샷AI 팀은 그간 와신상담 했고, 마침내 키미 K2를 딥시크 팀보다 먼저 공개하면서 세상에 충격을 주었습니다.

[^fn-kimi]: <https://arxiv.org/abs/2501.12599>

# 마치며
얼마전에 한 테크 사이트에서 매우 인상적인 표현을 봤습니다.

> "어느날 중국의 처음 들어보는 이름의 스타트업이 갑자기 세상을 깜짝 놀라게 하는 모델을 공개하는 일이 더 이상 놀랍지 않다"

LLM을 만드는 게 얼마나 어려운 일인데, 이걸 중국의 스타트업들이 이처럼 쉽게 해내고 있다니 (물론 내부적으로는 치열한 노력이 있었겠지만) 게다가 이제는 전 세계가 이런 소식을 당연하게 받아들인다는 사실은 이제 중국이 인공지능 기술의 최전선에 있음을 여실히 보여준다고 할 수 있습니다.

불과 몇 년 사이에 중국의 기술은 급속도로 발전했습니다. 혁신적인 모델 아키텍처를 선보였으며, 또 오픈소스 전략으로 전 세계가 그들의 기술을 활용하게 만들었습니다. 이제 우리나라에서도 이런 혁신이 일어나길 바라며, 세바스찬 라슈카가 우리나라에서 공개한 모델을 놀라워하며 소개하는 날이 하루 빨리 찾아오길 기대해봅니다.