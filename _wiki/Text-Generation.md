---
layout: wiki 
title: Text Generation
tags: ["Large Language Model (LLM)"]
last_modified_at: 2025/03/07 15:42:47
---

- [apply\_chat\_template, Streaming 예제](#apply_chat_template-streaming-예제)
- [Subword Tokenization](#subword-tokenization)

# apply_chat_template, Streaming 예제
```python
m = 'meta-llama/Llama-3.1-8B-Instruct'

# ---
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, TextStreamer

tokenizer = AutoTokenizer.from_pretrained(m)
model = AutoModelForCausalLM.from_pretrained(m,
    device_map='cuda', 
    trust_remote_code=True)
streamer = TextStreamer(tokenizer, 
    skip_prompt=True, skip_special_tokens=False)

def say(prompt: str):
    chat = [
        {"role": "user", "content": prompt},
    ]
    inputs = tokenizer.apply_chat_template(chat, 
        add_generation_prompt=True, return_dict=True, return_tensors="pt").to(0)
    _ = model.generate(**inputs, 
        streamer=streamer, do_sample=True, temperature=0.1, max_new_tokens=1024)


# ---
say('1km에 7분으로 달리면 마라톤 완주에 걸리는 시간은?')
```

`torch_dtype=torch.bfloat16`으로 모델을 로딩하면 첫 호출시 CPU로 파라미터를 전처리하는 작업이 있어 오래 걸린다.

파라미터 CUDA 위치 디버깅:
```python
>>> model.hf_device_map
```

```python
for i in model.named_parameters():
    print(f"{i[0]} -> {i[1].device}")
```

base 모델인 경우 `say()` 함수는 다르게 처리:
```python
def say(prompt: str):
    inputs = tokenizer(prompt, return_tensors="pt").to(0)
    _ = model.generate(**inputs, 
        streamer=streamer, do_sample=True, temperature=0.01, repetition_penalty=1.2, max_new_tokens=128)
```
# Subword Tokenization
드물게 등장하는 단어를 더 작은 단위로 나눔
- BPE: GPT 사용. 단어를 유니코드 문자가 아닌 바이트 단위 구성으로 간주
- WordPiece: BERT 사용
- SentencePiece: BPE + Unigram LM Tokenizer  
`text = "Jack Sparrow loves New York!"`  
<img src="https://user-images.githubusercontent.com/1250095/210939848-8e3b8cc7-cf87-4f2f-bcdd-025940052a15.png" width="70%">
WordPiece는 York과 ! 사이에 공백이 없다는 정보를 잃어버리지만 SentencePiece는 U+2581 또는 아래 1/4 블록 문자로 할당해 공백 정보를 보존한다.[^fn-142]

[^fn-142]: p142, 트랜스포머를 활용한 자연어 처리
