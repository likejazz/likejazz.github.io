---
layout: post
title: ! 'LLM 한국어 리더보드 기준 안내'
tags: ["Large Language Model (LLM)"]
last_modified_at: 2025/07/14 15:25:14
---

<div class="message">
<a href="https://leaderboard.dnotitia.com">LLM 한국어 리더보드</a>의 평가 기준을 소개합니다.
</div>

<small>
*Jul 01, 2025*
</small>

- [개요](#개요)
- [내용](#내용)
  - [LLM-as-a-Judge](#llm-as-a-judge)
  - [G-Eval](#g-eval)
  - [점수](#점수)
  - [평가 문항](#평가-문항)
- [결론](#결론)


# 개요

LLM의 성능을 평가하는 일은 매우 도전적인 과제입니다. 특히 같은 정답이어도 어떻게 생성하느냐에 따라 다른 점수가 부여될 수 있기 때문에 이를 제대로 평가하기란 여간 까다로운 일이 아니죠. 다행히 이를 자동으로 처리하기 위한 여러 방법론이 소개됐고 이 중에서 **LLM-as-a-Judge** 방식이 널리 쓰이고 있습니다.

# 내용

기존 MMLU 같은 벤치마크는 4지선다에서 정답이 맞는지 유무만 판단했습니다. 문제를 얼마나 제대로 이해하고 답변했는지, 설명이 얼마나 자연스러운지 여부는 보지 않았죠. 그렇다 보니 점수는 높은데 답변은 엉망인 이상한 모델이 등장하기도 했습니다. 생성형 모델에는 어울리지 않는 평가방식이었죠.

## LLM-as-a-Judge

LLM-as-a-Judge는 정답만 보는 게 아니라 말 그대로 LLM이 심사위원이 되어 마치 사람처럼 답변의 품질을 판단하는 방식입니다. 답변의 품질이 얼마나 좋은지 직접 답변을 읽어보고 LLM이 판단을 내리는 방식이죠. LMSYS에서 처음으로 MT-Bench 질문셋과 Chatbot Arena를 통한 크라우드 소싱 배틀 플랫폼을 소개[^fn-mt-bench]한 이래 이제 Chatbot Arena는 [lmarena라는 이름으로 가장 유명한 리더보드](https://lmarena.ai/leaderboard/text)로 성장했습니다. lmarena는 이제 더 이상 LLM-as-a-Judge를 활용하지 않지만 이때 도입된 LLM-as-a-Judge는 이후 여러 변형을 거쳐 널리 쓰이고 있죠.

[^fn-mt-bench]: <https://arxiv.org/abs/2306.05685>

## G-Eval

LLM이 제대로 평가하도록 하려면 기준을 명확히 하는 게 중요합니다. 기준이 없으면 매번 점수가 다르게 나올 수밖에 없죠. CoT 프롬프팅을 이용해 신뢰도 있는 점수를 정확하게 계산할 수 있도록 하는 프레임워크인 G-Eval[^fn-g-eval]의 탄생배경이기도 합니다.

[^fn-g-eval]: <https://arxiv.org/abs/2303.16634>

<img width="80%" src="https://github.com/user-attachments/assets/3a1ca22e-a57f-4f52-973b-1d95b6957f5d" />

방식은 간단합니다. 평가 기준<sup>Evaluation Criteria</sup>를 이용해 일련의 평가 단계<sup>Evaluation Steps</sup>를 생성하고 평가 기준<sup>Rubric</sup>을 마련해 주는 거죠. 실제로 제가 직접 작성한 프롬프트는 다음과 같습니다.

```
You are an evaluator. Given the following evaluation steps and rubric, assess the response below and return a JSON object with two fields:

- `"score"`: an integer between 0 and 10, based on the rubric provided.
- `"reason"`: a brief explanation for why the score was given. This must mention specific strengths or shortcomings, referencing relevant details from the input. Do **not** quote the score itself in the explanation.

Your explanation should:
- Be specific and grounded in the evaluation steps and rubric.
- Mention key details from the test case parameters.
- Be concise, clear, and focused on the evaluation logic.

Only return valid JSON. Do **not** include any extra commentary or text.

---


Evaluation Steps:
1. Accurately understand the instructions or requirements presented in the `expected_output`
2. Check if the response includes the correct answer, format, and content required by the `expected_output`
3. If the `expected_output` states "The answer is B", verify that B is explicitly specified as the answer in the response
4. If the `expected_output` states "You must explain physics content", check if physics content is included
5. Comprehensively evaluate whether the response meets all the essential conditions and instructions of the `expected_output`
6. If there are any parts that fail to meet the conditions presented in the `expected_output`, identify which parts


Rubric:
0-2: Expected output requirements completely unmet or contradicted.
3-4: Expected output requirements mostly unmet with significant gaps.
5-6: Expected output requirements partially met but missing key elements.
7-8: Expected output requirements mostly met with minor omissions.
9-10: Expected output requirements fully met and well-executed.

Test Case:
Input:
시계의 시침과 분침은 하루에 몇 번이나 만날까?

Actual Output:
시계의 시침과 분침이 하루에 만나는 횟수를 계산해보겠습니다.

## 계산 과정

**각 바늘의 속도:**
- 분침: 1시간에 360° 회전 (1분에 6°)
- 시침: 12시간에 360° 회전 (1분에 0.5°)

**상대 속도:**
분침이 시침보다 1분당 6° - 0.5° = 5.5° 더 빠르게 움직입니다.

**만나는 주기:**
분침이 시침을 한 바퀴 추월하려면 360°의 차이를 만회해야 합니다.
360° ÷ 5.5°/분 = 720/11분 ≈ 65.45분

즉, 약 1시간 5분 27초마다 두 바늘이 만납니다.

## 결과

**12시간 동안:** 720분 ÷ (720/11)분 = 11번

**24시간(하루) 동안:** 11 × 2 = **22번**

## 구체적인 만나는 시각들

12시간 기준으로 만나는 시각:
- 12:00
- 1:05:27
- 2:10:55
- 3:16:22
- 4:21:49
- 5:27:16
- 6:32:44
- 7:38:11
- 8:43:38
- 9:49:05
- 10:54:33

따라서 **하루에 22번** 만납니다.

Expected Output:
정답은 22번이다. 정답이 일치하는지 확인하라. 



Parameters:
Input, Actual Output, and Expected Output


---
**Example JSON:**
{
    "score": 0,
    "reason": "your concise and informative reason here"
}

JSON:
```

Evaluation Steps에는 어떤 점을 평가할지 기입하였습니다. 그리고 Rubric에는 어떤 기준에 따라 점수를 부여할지를 기입했죠. 이제 Test Case에 실제 응답을 기입합니다. Actual Output에는 모델의 응답을, Expected Output에는 기대하는 답변과 함께 어떤 기준이 가장 중요한지를 다시 한번 명시해 둡니다.

이제 LLM은 이 기준에 따라 일정한 점수를 부여하게 됩니다. 최종 결과는 점수와 왜 그렇게 점수를 측정했는지 이유를 JSON 형태로 출력하게 됩니다. 만약 LLM 평가자가 gpt-4.1 같은 성능 좋은 모델이라면 매우 정확하게 동작할 겁니다. 참고로 정답이 되는 기준점수<sup>Threshold</sup>는 0.7 이상으로 설정했습니다. Rubric에도 7부터 '대부분 충족<sup>requirements mostly met</sup>'으로 기술하였고, 왜 그렇게 평가했는지 이유를 다음과 같이 Reason에 표기하도록 했습니다.

<img width="70%" src="https://github.com/user-attachments/assets/b56af48e-7495-4614-b370-e4d421ce886c" />

## 점수
눈썰미가 있는 분들이라면 눈치채셨겠지만 Rubric에 있는 숫자는 최종 점수가 아닙니다. Rubric은 0에서 10 사이의 정수이고 실제 점수는 0에서 1사이의 실수거든요. 이유는 다음과 같습니다. 먼저 정확하고 세밀한<sup>fine-grained</sup> 채점을 위해서는 항상 신뢰할 만한 점수가 나와야 합니다. 매번 다르게 채점된다면 곤란하겠죠. 이러한 무작위성은 LLM의 특징이자 장점인 동시에 단점이 되기도 합니다. 항상 일관된 점수를 보여야 하는 LLM-as-a-Judge에서는 큰 단점으로 작용하죠. G-Eval에서는 이 문제를 확률 정규화<sup>Probability Normalization</sup>를 이용해 해결했습니다. 토큰의 로그 확률을 이용해 판단에 가중치를 부여함으로써 편향<sup>bias</sup>을 크게 줄이고 모델이 유사한 품질에 대해 항상 일관된 점수로 계산될 수 있도록 한 거죠. 그렇게 가중합<sup>Weighted Sum</sup>으로 계산한 결과를 최종 점수로 채택합니다.

<img width="50%" src="https://github.com/user-attachments/assets/a14ed46a-fd74-4280-8b3b-8734a776b4f8" />

## 평가 문항

평가 문항은 총 30문제를 준비했습니다. 좀 더 많은 문제를 준비하고 싶었지만 한 문제, 한 문제가 제가 일일이 직접 풀이해 보면서 정교하게 설계한 문제들이고 이미 수년 전부터 LLM을 평가할 때 개인적으로 사용해 오던 몇 가지 질문이 있었는데 이를 모두 포함하여 구성했습니다. 그러니까 예전에는 제가 직접 평가자<sup>Evaluator</sup>가 되어 사용하던 질문들을 이번에 모두 포함시키고 LLM-as-a-Judge가 동작되도록 한 거죠.

<img width="100%" src="https://github.com/user-attachments/assets/49eaafe3-a1ab-4a6e-a107-b608a851ed27" />

간단한 수학 문제부터, 기본적인 상식 문제, 제가 좋아하는 위스키에 관한 질문, 몬티홀 문제 같은 유명한 확률 문제, 컴퓨터 과학의 알고리즘 문제, 수능 국어 영역의 어려운 풀이 문제까지, 매우 정교하게 설계한 문제들로 구성했습니다. 문제 개수가 너무 적어서 변별력이 없을 거라 생각할 수 있지만 문제 하나하나가 매우 정교하고, 또 문제마다 부분 점수가 있기 때문에 실제로는 [LLM 한국어 리더보드](https://leaderboard.dnotitia.com)에서 확인할 수 있는 것처럼 변별력 있게 잘 동작합니다. 테스트 케이스와 점수가 정확하게 비례해서 증가하는 걸 확인할 수 있죠.

모든 문제는 한국어로 되어 있으며 여기서는 한국어 성능에 대한 평가만 진행합니다. 당연히 영어에서 성능이 높은 모델도 한국어에서는 성능이 떨어질 수 있습니다. 한국어 학습 데이터가 부족할 경우 일어날 수 있는 자연스러운 현상이므로, 만약 영어 성능 순위가 필요하다면 [lmarena](https://lmarena.ai/leaderboard/text)를 참고하시기 바랍니다.

# 결론

최종 평가 결과 [LLM 한국어 리더보드](https://leaderboard.dnotitia.com)에서 순위를 확인할 수 있습니다. 

x축은 정답을 맞힌(0.7점 이상) 테스트 케이스의 수, y축은 각 문항별 점수의 평균입니다. 모델의 응답은 매번 변하기 때문에 점수가 매번 달라집니다. 따라서 최소 7회 이상 동일한 방식으로 평가하고 절사평균[^fn-tm]을 적용했습니다. 절사평균이란 점수를 좀 더 정교하게 계산하기 위한 방식으로 상하 각각 15%로 정하였으며, 만약 7회 평가 결과가 있다면 이 중 상위 1회, 하위 1회를 제외한 5회의 표본으로 산술평균을 계산하는 방식을 말합니다. 절사평균값은 큰 원으로 표시했으므로, 각 모델의 실제 성능은 큰 원을 기준으로 확인하면 됩니다.

[^fn-tm]: <https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.trim_mean.html>

- [LLM 한국어 리더보드](https://leaderboard.dnotitia.com)

추후에는 변별력을 더욱 높여 평가 문항을 50문제까지 늘려 v2 평가셋을 준비할 예정입니다. 또한 평가 스크립트와 코드를 오픈소스로 공개하여 향후에는 각자 직접 평가할 수 있도록 준비해 보겠습니다.