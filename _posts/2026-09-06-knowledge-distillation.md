---
layout: post
title: ! 'Knowledge Distillation'
tags: ["Large Language Model (LLM)"]
last_modified_at: 2026/09/07 02:22:43
last_modified_history:
  - 2026/09/06 초안 작성
---

<div class="message">
..
</div>

- [개요](#개요)
- [구조](#구조)
  - [논문 수식](#논문-수식)
  - [코드 구현](#코드-구현)
- [기타](#기타)


# 개요

이동수 대표님의 글을 보면서 distillation 수식이 잘 기억나지 않아 논문을 다시 읽으며 코드로 구현해봤습니다. 논문의 수식과 알고리즘을 정리하고 실행 과정을 차트로 시각화했고요. 무엇보다 논문의 수식과 코드를 일치시키는 데 많은 시간을 할애했습니다. 당연히 코딩 어시스턴트의 도움을 많이 받았지만 완성도를 높이는 최종 마무리 작업은 Cursor로 직접 전체 코드를 꼼꼼히 리뷰하며 진행했습니다. 그렇게 해야 제가 원하는 결과물이 나오기 때문이죠. 아마도 어시스턴트에 모든 걸 맡겼다면 이 정도 결과가 나오지 않았을 거예요. 인간 책임자가 완전히 이해한 코드로 내놓는 결과물은 여전히 가치가 있습니다. 이처럼 별도로 지식을 정리하여 앞으로 두고두고 참조할 수 있을테고요.

# 구조

실제로 논문의 수식과 코드를 비교해보겠습니다.

[일러스트]

원래 비전에서 시작된 알고리즘이다 보니 이미지 분류 예시가 많은데, LLM에서도 동일하게 적용해볼 수 있습니다. 다만 LLM에서는 true label이 아니라 teacher label[?]이라는 차이가 있고, 이 부분은 아래 Raschka의 그림이 좀 더 도움이 될 거 같네요.

[일러스트]

## 논문 수식

Distilling the Knowledge in a Neural Network[^fn-paper] (Hinton et al., 2015)

[^fn-paper]: <https://arxiv.org/abs/1503.02531>

(1) 온도 T를 적용한 softmax

$$
q_i = \frac{\exp(z_i/T)}{\sum_j \exp(z_j/T)}
$$

(2) 로짓 $$z_i$$에 대한 cross-entropy gradient

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

(4) 로짓이 각 transfer case별로 zero-mean이라 가정할 때
$$(∑_j z_j = ∑_j v_j = 0)$$ 수식 (3)은 다음과 같이 단순화

$$
\frac{\partial C}{\partial z_i} \approx \frac{1}{NT^2}\left( z_i - v_i \right)
$$

## 코드 구현

<https://github.com/likejazz/knowledge-distillation>

그래서 코드 구현은 다음과 같습니다.

먼저 teacher (cumbersome)와 student (distilled) 모델의 logit 값을 추출합니다.

```python
with torch.no_grad():
    v = teacher(data)
z = student(data)
```

여기서는 예를 들기 위해 2개의 임의의 텐서를 정의했습니다.

```python
>>> v
tensor([[ 0.5032, -7.8997,  2.9085],
        [-5.1517,  3.8629, -3.6802]])
>>> z
tensor([[ 0.9008, -0.0776,  1.1376],
        [-0.4617, -0.1822, -0.2040]])
```

logits를 온도 T로 나눈 뒤 softmax 해서, 클래스별 확률로 바꿉니다.
```python
p = F.softmax(v / T, dim=-1)
```

T는 온도 (temperature)이며, LLM에서와 동일하게 높을수록 값이 smooth해집니다. 

```python
>>> F.softmax(v / 1, dim=-1)
tensor([[0.0828, 0.0000, 0.9172],
        [0.0001, 0.9993, 0.0005]])
>>> F.softmax(v / 3, dim=-1)
tensor([[0.3039, 0.0185, 0.6776],
        [0.0438, 0.8846, 0.0716]])
```

이제 student가 얼마나 teacher의 loss를 잘 따라가는지 cross entropy를 구해봅니다. 논문에서 "The first objective function is the cross entropy with the soft targets and this cross entropy is computed using the same high temperature in the softmax of the distilled model as was used for generating the soft targets from the cumbersome model."로 첫 번째 목적 함수를 정의한 부분입니다.

$$C = -\sum_i p_i \log q_i,$$

```python
log_q = F.log_softmax(z / T, dim=-1)
C = -(p * log_q).sum(dim=-1).mean()
```

이 값도 확인해보면 다음과 같습니다.

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

-log_q는 결국 logit이 크면 더 작은 값이, logit이 작을 경우 큰 값이 반영됩니다. 결국 teacher의 변별력이 떨어질수록 C는 커집니다. 정답에 logit이 가장 크다면 가장 작은 값이 결국 반영되므로 loss는 줄어듭니다.

```python
>>> -(p * torch.log_softmax(torch.tensor([[1.0,0.0,0.0],[0.0,1.0,0.0]]), dim=-1)).sum(dim=-1)
tensor([1.2475, 0.6668])
>>> -(p * torch.log_softmax(torch.tensor([[1.0,0.0,0.0],[1.0,0.0,0.0]]), dim=-1)).sum(dim=-1)
tensor([1.2475, 1.5076])
>>> -(p * torch.log_softmax(torch.tensor([[1.0,0.0,0.0],[0.0,0.0,1.0]]), dim=-1)).sum(dim=-1)
tensor([1.2475, 1.4799])
```

그러나 정답이 일치해도, 심지어 logits가 완전히 동일해도 loss가 0이 되지는 않습니다.

이후 온도가 높을 때 값이 지나치게 작아지지 않도록 $$T^2$$으로 C값을 보정해줍니다.

```python
soft_loss = (T**2) * C
```

# 기타

```python
with torch.no_grad():
    v = teacher(data)
z = student(data)

loss = F.mse_loss(v, z)
```

ditillation이 목적이라면 복잡한 구현없이 이 같은 형태로 두 매트릭스의 MSE만 구해도 충분합니다. hard loss 또한 정답이 없는 문제에서는 필요가 없습니다.