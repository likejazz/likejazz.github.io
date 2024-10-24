---
layout: wiki 
title: llama.cpp Quantization
tags: ["llama.cpp"]
last_modified_at: 2024/10/20 02:24:15
---

<!-- TOC -->

- [Quantization Types](#quantization-types)
- [속도 측정](#속도-측정)
  - [ollama](#ollama)
  - [transformers](#transformers)
- [품질 측정](#품질-측정)

<!-- /TOC -->

# Quantization Types
`Q4_K_M`, `Q5_K_S` and `Q5_K_M` are considered "recommended"[^fn-quant].

[^fn-quant]: <https://github.com/ggerganov/llama.cpp/discussions/2094#discussioncomment-6351796>

# 속도 측정
- codellama:70b, ollama, A100: 24 tokens/s

ollama에서 server overloaded 오류 발생 이슈가 있다.
```
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
```

- mixtral, ollama, A100: 58 tokens/s
- mistral, ollama, A100: 112 tokens/s

Llama 2 7B:
## ollama
- llama2:7b-chat-fp16, ollama, A100: 71 tokens/s
- llama2:7b-chat-q8_0, ollama, A100: 105 tokens/s
- llama2:7b-chat-q6_K, ollama, A100: 97 tokens/s
- **llama2:7b-chat-q5_K_M, ollama, A100: 108 tokens/s**
- llama2:7b-chat-q5_K_S, ollama, A100: 110 tokens/s
- llama2:7b-chat-q4_K_M, ollama, A100: 113 tokens/s
- llama2:7b-chat-q4_1, ollama, A100: 131 tokens/s
- llama2:7b-chat-q4_0(default), ollama, A100: 129 tokens/s

## transformers
- Llama-2-7b-chat-hf, float32, A100: 42 tokens/s
- Llama-2-7b-chat-hf, float16, A100: 38 tokens/s
- Llama-2-7b-chat-hf, bfloat16, A100: 40 tokens/s

# 품질 측정
```
Transformers model.logits (torch.float32)
/models/42dot_LLM-SFT-1.3B
' 사랑'        9392  23.30%(21.639068603515625)
' 저는'        15780 10.83%(20.873014450073242)
' 죄송'        40698 6.63%(20.38260269165039)
' "'         497   3.38%(19.707277297973633)
' I'         316   3.25%(19.668506622314453)
' 감사합니다'     17871 3.04%(19.60260581970215)
' Hello'     22889 2.96%(19.575531005859375)
' 제가'        11513 2.51%(19.41057586669922)
' 나는'        12503 1.94%(19.152572631835938)
' 안녕하세요'     43686 1.52%(18.912193298339844)

llama.cpp f32 5.4G
' 사랑'        9392  23.34%(21.644859313964844)
' 저는'        15780 10.87%(20.880815505981445)
' 죄송'        40698 6.65%(20.388774871826172)
' "'         497   3.38%(19.714048385620117)
' I'         316   3.26%(19.675649642944336)
' 감사합니다'     17871 3.03%(19.604887008666992)
' Hello'     22889 2.96%(19.578733444213867)
' 제가'        11513 2.51%(19.416027069091797)
' 나는'        12503 1.94%(19.157896041870117)
' 안녕하세요'     43686 1.52%(18.912742614746094)

llama.cpp f16 2.6G
' 사랑'        9392  23.74%(21.66746711730957)
' 저는'        15780 10.90%(20.88936424255371)
' 죄송'        40698 6.47%(20.36779022216797)
' "'         497   3.26%(19.680843353271484)
' Hello'     22889 3.24%(19.674625396728516)
' 감사합니다'     17871 3.08%(19.626041412353516)
' I'         316   2.94%(19.577871322631836)
' 제가'        11513 2.39%(19.372831344604492)
' 나는'        12503 1.92%(19.15070152282715)
' 안녕하세요'     43686 1.50%(18.904869079589844)

llama.cpp Q8_0 1.5G, +0.0004 ppl @ LLaMA-v1-7B
' 사랑'        9392  23.06%(21.626585006713867)
' 저는'        15780 11.44%(20.92563819885254)
' 죄송'        40698 6.53%(20.36493492126465)
' "'         497   3.31%(19.684282302856445)
' I'         316   3.21%(19.655441284179688)
' 감사합니다'     17871 3.16%(19.63981056213379)
' Hello'     22889 3.13%(19.62859344482422)
' 제가'        11513 2.70%(19.48176383972168)
' 나는'        12503 1.90%(19.12944221496582)
' 안녕하세요'     43686 1.62%(18.96955108642578)

llama.cpp Q6_K 1.2G, -0.0008 ppl @ LLaMA-v1-7B
' 사랑'        9392  25.01%(21.701353073120117)
' 저는'        15780 9.07%(20.68704605102539)
' 죄송'        40698 5.28%(20.145719528198242)
' Hello'     22889 4.04%(19.87941551208496)
' 감사합니다'     17871 2.92%(19.554365158081055)
' I'         316   2.89%(19.544178009033203)
' "'         497   2.77%(19.502199172973633)
' 나는'        12503 2.37%(19.34703254699707)
' 제가'        11513 1.86%(19.102651596069336)
' 안녕하세요'     43686 1.53%(18.908605575561523)

llama.cpp Q5_K_M 983M, +0.0142 ppl @ LLaMA-v1-7B
' 사랑'        9392  25.86%(21.729509353637695)
' 저는'        15780 9.29%(20.7059326171875)
' 죄송'        40698 5.27%(20.137887954711914)
' Hello'     22889 3.47%(19.721651077270508)
' 감사합니다'     17871 3.32%(19.676420211791992)
' "'         497   3.19%(19.6379451751709)
' I'         316   2.80%(19.506134033203125)
' 제가'        11513 2.05%(19.19264030456543)
' 나는'        12503 1.87%(19.104854583740234)
' 당신'        14004 1.50%(18.88077163696289)

llama.cpp Q4_0 800M, +0.2166 ppl @ LLaMA-v1-7B
' 사랑'        9392  27.14%(21.992361068725586)
' 저는'        15780 13.64%(21.30475616455078)
' 죄송'        40698 7.21%(20.66753578186035)
' 감사합니다'     17871 5.00%(20.30014419555664)
' "'         497   2.74%(19.70037078857422)
' 제가'        11513 2.51%(19.612300872802734)
' 감사'        9081  2.22%(19.48822593688965)
' 사랑을'       30330 2.15%(19.45526885986328)
' Hello'     22889 2.03%(19.400882720947266)
' I'         316   1.53%(19.116830825805664)
```

[#1-2. 모델 성능/속도 측정 스크립트](/wiki/Private-Links)

| Model | Measure | F32 | F16 | Q8_0 | Q6_K | Q5_K_M | Q5_K_S | Q4_K_M | Q4_1 | Q4_0 |
| ----- | ------- | --- | --- | ---- | ---- | ------ | ------ | ------ | ---- | ---- |
|42dot 1.3B | perplexity | 13.2410 | 13.2588 | 13.2526 | 13.2784 | 13.3141 | 13.3523 | 13.4439 | 13.7319 | 13.9454 |
|42dot 1.3B | file size | 5.4G | 2.6G | 1.5G | 1.2G | 983M | 960M | 847M | 880M | 800M |
|42dot 1.3B | tokens/s | 125 | 168 | 177 | 177 | 180 | 182 | 185 | 192 | 189 |

<img src="/images/2024/tokens-ppl-42dot-matplotlib.png" width="60%">[^fn-colab]

[^fn-colab]: <https://colab.research.google.com/drive/1mkcBq3PQxbxzOQR9E_AsMTWYOVzhR-7a?usp=sharing>
