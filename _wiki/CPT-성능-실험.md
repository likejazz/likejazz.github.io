---
layout: wiki 
title: CPT 성능 실험
tags: ["LLM Training"]
last_modified_at: 2026/02/19 15:05:10
last_modified_history:
  - 2024/12/21
---

- [한국어 성능 평가 결과](#한국어-성능-평가-결과)
- [실험](#실험)
- [트러블슈팅](#트러블슈팅)
- [generation\_config.json](#generation_configjson)

# 한국어 성능 평가 결과

- **CohereForAI/aya-expanse-32b** 상당히 잘하지만 일부 할루시네이션이 있다. function call은 동작하지 않는다.
- **meta-llama/Llama-3.3-70B-Instruct** 한국어 능력이 생각보다 좋지 않다. 역사적 사실을 틀리기도 하며, 이 모델도 벤치마크 점수는 높은데, 한글 대답이 너무 짧다. function call은 당연히 잘 된다.
- **LGAI-EXAONE/EXAONE-3.5-32B-Instruct** 상당히 준수하다. 대부분의 답변에 그럴듯 하게 답변하며, function call은 되지 않으나 비슷하게 동작은 한다.
- **Qwen/Qwen2.5-72B-Instruct** 상당히 준수하다. 하지만 중국 관련 질문에서는 갑자기 중국어가 나온다. 이외에 수능 문제에서도 중국어가 나오기도 한다. function call도 잘 되는데, 학습이 되어 있는 거 같다.
- **Qwen/Qwen2.5-32B-Instruct** 가끔 중국어가 나오지만 마찬가지로 잘 되는 편이다. function call도 잘 동작한다.
- **Qwen/QwQ-32B-Preview** 이 모델은 거의 모든 대답이 중국어다. 영어 성능은 극찬을 받고 있지만, 한국어는 안된다고 봐야 한다.

# 실험

(비공개) [#1-16 quality-assessment.py](/wiki/Private-Links)

# 트러블슈팅
llama3-8.1b를 그대로 SFT 학습하면 `<|eot_id|>`가 eos 토큰으로 학습된다. 그런데 generation_config.json에서는 `"eos_token_id": 128001,`만 등록되어 있어 나중에 instruct 모델에서 eot_id가 eos로 인식되지 않는다. 우리 모델은 end_of_text로 변경해서 학습한거 같고, 만약 그대로 학습할 경우 다음과 같이 instruct 모델의 generation_config.json에서는 `"eos_token_id": [128001,128009],`로 수정이 필요하다.

---
meta-llama/Llama-3.1-70B 추론을 해보면, 'DNA 이중나선 구조를 발견한 과학자는'이라는 프롬프트에,
```
A: 제임스 왓슨
B: 프랜시스 크릭
C: ...
D: ...
Answer: A
```
같은 식으로 4지 선다로 제시하고 정답을 제시하는 경우가 많다. 마지막에 annealing 단계에서 MMLU 스타일의 데이터로 학습했기 때문인거 같다. 우리가 학습한 CPT 모델도 프롬프트를 입력하면 모두 한글 4지 선다로 만드는 경향이 있는데 마찬가지로 KMMLU 데이터로 annealing 했기 때문으로 보인다.

---
function call이 안되는 모델을 llama 3.1 8b instruct와 merge하니까 일부 function call을 지원하기 시작했다. llama 쪽의 fc 지원 기능이 일부 이식된 것으로 보인다.

# generation_config.json
```json
{
  "bos_token_id": 128000,
  "do_sample": true,
  "eos_token_id": [
    128001,
    128008,
    128009
  ],
  "pad_token_id": 128004,
  "temperature": 0.1,
  "top_p": 0.9,
  "max_new_tokens": 512,
  "transformers_version": "4.43.0"
}
```