---
layout: wiki 
title: Redis
tags: ["Software Engineering"]
last_modified_at: 2023/07/21 22:39:29
---

<!-- TOC -->

- [목차](#목차)

<!-- /TOC -->

# Amazon MemoryDB for Redis
Redis-Compatible IMDB. 가격은 ElastiCache보다 조금 더 비싸다. DB와 Redis를 접목한 개념이라는데, 일단 Redis API가 모두 동작한다.

## Setup redis-cli on EC2
```
$ sudo yum install make gcc openssl-devel -y
$ wget http://download.redis.io/redis-stable.tar.gz && \
    tar xvzf redis-stable.tar.gz && \
    cd redis-stable && \
    make BUILD_TLS=yes && \
    sudo make install
$ redis-cli -h clustercfg.chatbaker-memorydb.xxx.memorydb.ap-northeast-2.amazonaws.com \
    -p 6379 \
    --tls
```