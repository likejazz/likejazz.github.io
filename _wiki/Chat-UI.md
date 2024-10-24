---
layout: wiki 
title: Chat UI
tags: ["Large Language Model (LLM)"]
last_modified_at: 2024/02/18 01:14:34
---

- [실행](#실행)
  - [Proxy 설정](#proxy-설정)

# 실행
채팅 히스토리 저장을 위해 MongoDB를 구동해야 함. 이후 다른 docker에서 link로 연결해서 접속될 수 있도록 한다.
```
$ docker run -d -p 27017:27017 --name mongo-chatui mongo:latest
$ docker run --gpus all --rm -d \
    --shm-size=1g --ulimit memlock=-1 \
    --publish 2000:22 \
    --publish 7000-9000:7000-9000 \
    -v /home/hyperai/xxx:/home \
    -v /models:/models \
    --link mongo-chatui \
    hyperai/pytorch:24.02
```

```
# .env.local에 MONGODB_URL, MODELS, SERPER_API_KEY 설정
$ npm install
$ npm run dev -- --port 8888 --host
```

## Proxy 설정
```
$ ssh -N -L 8888:xx.xx.xx.xx:8888 xxx-user@xx.xx.xx.xx -p xxxxx
```

---
Ollama Qwen 72B-chat-v1.5-Q5_K_S도 한국어 성능이 만족스럽지 않음. chatPromptTemplate 구성이 까다롭다. Ollama CLI는 그나마 낫지만 A100에서도 많이 느리다. 프롬프트를 어느정도 따르지만 가끔 한글이 깨지고 갑자기 중간에 멈추고 이후에 실행이 되지 않는 경우가 있었다.