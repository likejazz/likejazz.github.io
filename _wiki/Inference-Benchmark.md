---
layout: wiki 
title: Inference Benchmark
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/02/14 00:46:37
---

<!-- TOC -->

- [Quantization Types](#quantization-types)
- [속도 측정](#속도-측정)
- [품질 측정](#품질-측정)

<!-- /TOC -->

# Quantization Types
```
Allowed quantization types:
   2  or  Q4_0   :  3.50G, +0.2499 ppl @ 7B - small, very high quality loss - legacy, prefer using Q3_K_M
   3  or  Q4_1   :  3.90G, +0.1846 ppl @ 7B - small, substantial quality loss - legacy, prefer using Q3_K_L
   8  or  Q5_0   :  4.30G, +0.0796 ppl @ 7B - medium, balanced quality - legacy, prefer using Q4_K_M
   9  or  Q5_1   :  4.70G, +0.0415 ppl @ 7B - medium, low quality loss - legacy, prefer using Q5_K_M
  10  or  Q2_K   :  2.67G, +0.8698 ppl @ 7B - smallest, extreme quality loss - not recommended
  12  or  Q3_K   : alias for Q3_K_M
  11  or  Q3_K_S :  2.75G, +0.5505 ppl @ 7B - very small, very high quality loss
  12  or  Q3_K_M :  3.06G, +0.2437 ppl @ 7B - very small, very high quality loss
  13  or  Q3_K_L :  3.35G, +0.1803 ppl @ 7B - small, substantial quality loss
  15  or  Q4_K   : alias for Q4_K_M
  14  or  Q4_K_S :  3.56G, +0.1149 ppl @ 7B - small, significant quality loss
  15  or  Q4_K_M :  3.80G, +0.0535 ppl @ 7B - medium, balanced quality - *recommended*
  17  or  Q5_K   : alias for Q5_K_M
  16  or  Q5_K_S :  4.33G, +0.0353 ppl @ 7B - large, low quality loss - *recommended*
  17  or  Q5_K_M :  4.45G, +0.0142 ppl @ 7B - large, very low quality loss - *recommended*
  18  or  Q6_K   :  5.15G, +0.0044 ppl @ 7B - very large, extremely low quality loss
   7  or  Q8_0   :  6.70G, +0.0004 ppl @ 7B - very large, extremely low quality loss - not recommended
   1  or  F16    : 13.00G              @ 7B - extremely large, virtually no quality loss - not recommended
   0  or  F32    : 26.00G              @ 7B - absolutely huge, lossless - not recommended
```

`Q4_K_M`, `Q5_K_S` and `Q5_K_M` are considered "recommended"[^fn-quant].

[^fn-quant]: <https://github.com/ggerganov/llama.cpp/discussions/2094#discussioncomment-6351796>

# 속도 측정
- codallama:70b, ollama, A100: 24 tokens/s

ollama에서 server overloaded 오류 발생 이슈가 있다.
```
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
```

- mixtral, ollama, A100: 58 tokens/s
- mistral, ollama, A100: 112 tokens/s

Llama 2 7B:
- llama2:7b-chat-fp16, ollama, A100: 70 tokens/s
- llama2:7b-chat-q8_0, ollama, A100: 90 tokens/s
- llama2:7b-chat-q6_K, ollama, A100: 78 tokens/s
- llama2:7b-chat-q5_K_M, ollama, A100: 94 tokens/s
- **llama2:7b-chat-q5_K_S, ollama, A100: 98 tokens/s**
- llama2:7b-chat-q4_K_M, ollama, A100: 96 tokens/s
- llama2:7b-chat-q4_1, ollama, A100: 120 tokens/s
- llama2:7b-chat-q4_0(default), ollama, A100: 120 tokens/s

Transformers:
- Llama-2-7b-chat-hf, hf/float32(same w/ bf16,fp16), A100: 41 tokens/s

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

[측정 스크립트 #2](https://github.com/likejazz/private-links)

| Model | Measure | F32 | F16 | Q8_0 | Q6_K | Q5_K_M | Q5_K_S | Q4_K_M | Q4_1 | Q4_0 |
| ----- | ------- | --- | --- | ---- | ---- | ------ | ------ | ------ | ---- | ---- |
|42dot 1.3B | perplexity | 13.2410 | 13.2588 | 13.2526 | 13.2784 | 13.3141 | 13.3523 | 13.4439 | 13.7319 | 13.9454 |
|42dot 1.3B | file size | 5.4G | 2.6G | 1.5G | 1.2G | 983M | 960M | 847M | 880M | 800M |
|42dot 1.3B | tokens/s | N/A | 70 | 90 | 78 | 94 | 98 | 96 | 120 | 120 |

<img src="/images/2024/tokens-ppl-matplotlib.png" width="60%">[^fn-colab]

[^fn-colab]: <https://colab.research.google.com/drive/1mkcBq3PQxbxzOQR9E_AsMTWYOVzhR-7a?usp=sharing>