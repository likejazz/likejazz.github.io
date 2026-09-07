---
layout: post
title: ! 'Knowledge Distillation'
tags: ["Large Language Model (LLM)"]
last_modified_at: 2026/09/07 12:46:23
last_modified_history:
  - 2026/09/06 초안 작성
---

<div class="message">
Distilling the Knowledge in a Neural Network 논문을 다시 읽으며 knowledge distillation의 수식을 정리하고, 이를 PyTorch 코드로 옮겨 수식과 코드가 어떻게 대응되는지 한 줄씩 확인해봅니다. 온도 T를 적용한 softmax부터 soft target에 대한 cross entropy, T² 보정까지 논문의 흐름을 그대로 따라가며 구현합니다.
</div>

- [개요](#개요)
- [구조](#구조)
  - [논문 수식](#논문-수식)
  - [코드 구현](#코드-구현)
- [실험](#실험)
- [기타](#기타)


# 개요

[이동수 대표님의 글](https://www.facebook.com/dongsoo.lee.104/posts/pfbid022uG2gf5KXjRdpfAWabzqPH3Fc8ZeX6ghFTsLtmtTVip8GFBSPd4tH5JkVBkWggvQl)을 읽다가 distillation 수식이 잘 기억나지 않아 논문을 다시 읽었고, 읽는 김에 코드로도 함께 구현해봤습니다. 논문의 수식과 알고리즘을 정리하고 실행 과정을 차트로 시각화했는데, 무엇보다 수식과 코드를 정확히 일치시키는 데 가장 많은 시간을 들였습니다. 물론 코딩 어시스턴트의 도움을 많이 받았지만 완성도를 높이는 마무리 작업은 Cursor로 전체 코드를 직접 꼼꼼히 리뷰하며 진행했습니다. 그렇게 해야 제가 원하는 결과물이 나오기 때문이죠. 어시스턴트에 모든 걸 맡겼다면 아마 이 정도 결과는 나오지 않았을 겁니다. 사람이 완전히 이해하고 책임지는 코드로 내놓는 결과물은 여전히 가치가 있고, 이렇게 정리해둔 지식은 앞으로도 두고두고 참조할 수 있습니다.

# 구조

<img src="https://github.com/user-attachments/assets/ac232fd6-513d-47b0-8d4a-41c59dafdf6f" width="60%">

원래 비전 분야에서 시작된 알고리즘이다 보니 이미지 분류 예시가 많지만, LLM에도 동일하게 적용할 수 있습니다. 다만 LLM에서는 hard loss를 계산할 때 true label 대신 teacher token을 사용한다는 차이가 있는데, 이 부분은 아래 Raschka의 그림이 더 도움이 될 것 같습니다.

<img src="https://github.com/user-attachments/assets/3dc3de2a-e5a2-4652-a2ff-0d26fe7b8c15" width="80%">

## 논문 수식

그렇다면 *Distilling the Knowledge in a Neural Network*[^fn-paper] (Hinton et al., 2015) 논문의 수식부터 하나씩 살펴보겠습니다.

[^fn-paper]: <https://arxiv.org/abs/1503.02531>

(1) 온도 T를 적용한 softmax

$$
q_i = \frac{\exp(z_i/T)}{\sum_j \exp(z_j/T)}
$$

(2) 로짓 $$z_i$$에 대한 cross-entropy의 gradient

$$
\frac{\partial C}{\partial z_i}
  = \frac{1}{T}\left( q_i - p_i \right)
  = \frac{1}{T}\left(
      \frac{e^{z_i/T}}{\sum_j e^{z_j/T}}
      - \frac{e^{v_i/T}}{\sum_j e^{v_j/T}}
    \right)
$$

(3) T가 로짓 크기에 비해 충분히 클 때의 근사 ($$e^x ≈ 1 + x$$)

$$
\frac{\partial C}{\partial z_i} \approx \frac{1}{T}\left(
    \frac{1 + z_i/T}{N + \sum_j z_j/T}
    - \frac{1 + v_i/T}{N + \sum_j v_j/T}
  \right)
$$

(4) 각 transfer case별로 로짓이 zero-mean이라 가정하면
$$(∑_j z_j = ∑_j v_j = 0)$$ 수식 (3)은 다음과 같이 단순화

$$
\frac{\partial C}{\partial z_i} \approx \frac{1}{NT^2}\left( z_i - v_i \right)
$$

## 코드 구현

이를 코드로 옮기면 다음과 같습니다.

<https://github.com/likejazz/knowledge-distillation>

먼저 teacher (cumbersome) 모델과 student (distilled) 모델에서 각각 logit을 추출합니다. 논문의 표기를 따라 teacher의 logit은 v, student의 logit은 z로 두겠습니다.

```python
with torch.no_grad():
    v = teacher(data)
z = student(data)
```

여기서는 설명을 위해 임의의 텐서 2개를 정의했습니다.

```python
>>> v
tensor([[ 0.5032, -7.8997,  2.9085],
        [-5.1517,  3.8629, -3.6802]])
>>> z
tensor([[ 0.9008, -0.0776,  1.1376],
        [-0.4617, -0.1822, -0.2040]])
```

logit을 온도 T로 나눈 뒤 softmax를 취해 클래스별 확률로 바꿉니다. 이 값이 논문에서 말하는 soft target p입니다.

```python
p = F.softmax(v / T, dim=-1)
```

T는 온도(temperature)로, LLM에서와 마찬가지로 값이 높을수록 분포가 더 smooth해집니다. T=1일 때와 T=3일 때를 비교해보면 차이가 분명합니다.

```python
>>> F.softmax(v / 1, dim=-1)
tensor([[0.0828, 0.0000, 0.9172],
        [0.0001, 0.9993, 0.0005]])
>>> F.softmax(v / 3, dim=-1)
tensor([[0.3039, 0.0185, 0.6776],
        [0.0438, 0.8846, 0.0716]])
```

이제 student가 teacher의 분포를 얼마나 잘 따라가는지 cross entropy로 측정합니다. 논문에서 첫 번째 목적 함수를 정의한 부분으로 soft target과의 cross entropy이며, teacher와 student가 동일한 T를 사용합니다.

$$C = -\sum_i p_i \log q_i$$

```python
log_q = F.log_softmax(z / T, dim=-1)
C = -(p * log_q).sum(dim=-1).mean()
```

각 단계의 값을 확인해보면 다음과 같습니다.

```python
>>> F.softmax(v / T, dim=-1)     # p
tensor([[0.3039, 0.0185, 0.6776],
        [0.0438, 0.8846, 0.0716]])
>>> F.log_softmax(z / T,dim=-1)  # log_q
tensor([[-1.0310, -1.3571, -0.9520],
        [-1.1592, -1.0660, -1.0733]])
>>> -(p * log_q)
tensor([[0.3134, 0.0251, 0.6451],
        [0.0508, 0.9430, 0.0768]])
```

-log_q는 logit이 클수록 작은 값, 작을수록 큰 값을 갖습니다. 여기에 teacher의 확률 p를 곱해 더하므로, teacher가 높은 확률을 준 클래스에 student도 큰 logit을 주면 loss는 줄어들고, 그렇지 않으면 커집니다. 또한 teacher의 변별력이 떨어져 p가 평평해질수록 C는 전반적으로 커집니다. 아래처럼 student의 logit을 바꿔가며 확인해보면, teacher가 정답으로 본 위치에 가장 큰 logit을 줬을 때 loss가 가장 작은걸 확인할 수 있습니다.

```python
>>> -(p * torch.log_softmax(torch.tensor([[1.0,0.0,0.0],[0.0,1.0,0.0]]), dim=-1)).sum(dim=-1)
tensor([1.2475, 0.6668])
>>> -(p * torch.log_softmax(torch.tensor([[1.0,0.0,0.0],[1.0,0.0,0.0]]), dim=-1)).sum(dim=-1)
tensor([1.2475, 1.5076])
>>> -(p * torch.log_softmax(torch.tensor([[1.0,0.0,0.0],[0.0,0.0,1.0]]), dim=-1)).sum(dim=-1)
tensor([1.2475, 1.4799])
```

다만 정답이 일치해도, 심지어 logit이 완전히 동일해도 loss가 0이 되지는 않습니다. cross entropy의 최솟값은 0이 아니라 p 자체의 entropy이기 때문입니다. entropy는 변별력이 클수록 줄어듭니다.

마지막으로 온도가 높을 때 gradient가 지나치게 작아지지 않도록 C에 $$T^2$$을 곱해 보정합니다. 수식 (4)에 등장하는 $$1/T^2$$ 항을 상쇄하는 것이죠.

```python
soft_loss = (T**2) * C
```

# 실험

분류 문제를 풀이해봅시다.

<img src="https://github.com/user-attachments/assets/c5c97121-1e2d-4a8a-b495-c28f5c856815" width="70%">

이 같은 데이터가 있다고 가정했을 때 위 코드로 학습 시 다음과 같은 distillation이 잘 진행된 결과를 확인할 수 있습니다.

<img src="https://github.com/user-attachments/assets/ee1da56e-c68f-44f4-9ba3-f39799a3a6c8" width="100%">

# 기타

```python
with torch.no_grad():
    v = teacher(data)
z = student(data)

loss = F.mse_loss(v, z)
# or
# loss = F.cross_entropy(p, q)
```

distillation 자체가 목적이라면 복잡한 구현 없이 이처럼 두 logit 행렬의 MSE만 구해도 충분합니다. LLM처럼 정답 label이 없는 문제라면 hard loss 역시 필요하지 않습니다.

그러나 Hinton이 KD의 존재 이유로 든 "오답들 사이의 상대적 유사도"를 파악하기가 어렵습니다. 또한 꼬리의 noise까지 똑같이 맞추려고 하는 문제가 있습니다. 만약 softmax MSE를 한다면 반대로 상위 한두 개 클래스만 보고 나머지는 무시하게 됩니다. KL은 그 중간에 있고, T로 그 균형을 조절하는 형태입니다.