---
layout: wiki 
title: NLP
tags: ["NLP"]
last_modified_at: 2025/06/08 19:30:27
---

<!-- TOC -->

- [NLU](#nlu)
- [Neural History of NLP](#neural-history-of-nlp)
  - [Neural language models](#neural-language-models)
  - [Multi-task learning](#multi-task-learning)
  - [Neural networks for NLP](#neural-networks-for-nlp)
  - [Sequence-to-sequence models](#sequence-to-sequence-models)
  - [Attention](#attention)
  - [Memory-based networks](#memory-based-networks)
  - [Pretrained language models](#pretrained-language-models)
- [LM Perplexity](#lm-perplexity)
- [BERT, RoBERTa, DistilBERT, XLNet — which one to use?](#bert-roberta-distilbert-xlnet--which-one-to-use)

<!-- /TOC -->

# NLU
문장 이해의 3가지 요소
1. 구문 분석 <sup>syntactic analysis(parsing)</sup>
1. 의미 분석 <sup>semantic analysis</sup>
1. 실용(화용) 분석 <sup>pragmatic analysis</sup>: 어떠한 문장(음성으로 말하는 어문)이 나타내는 의미를 알기 위해 다시 해석되는 과정. 관용적으로 하는 이야기를 재해석해서 이해하는 것.

<img src="https://www.tutorialspoint.com/artificial_intelligence/images/steps_in_nlp.jpg" width="20%"> ([AI - Natural Language Processing](https://www.tutorialspoint.com/artificial_intelligence/artificial_intelligence_natural_language_processing))

# Neural History of NLP
<img src="https://pbs.twimg.com/media/DobzavEXsAEkBVc?format=jpg" width="70%">

[A Review of the Neural History of Natural Language Processing](http://blog.aylien.com/a-review-of-the-recent-history-of-natural-language-processing/#2001neurallanguagemodels)

## Neural language models
- The first neural language model, a feed-forward neural network was proposed in 2001 by Bengio et al. 
    - Word embeddings: The objective of word2vec is a simplification of language modelling. word2vec의 목표는 랭기지 모델의 단순화
    - Sequence-to-sequence models: Such models generate an output sequence by predicting one word at a time. 한 번에 한 단어씩 예측하여 출력 시퀀스를 생성한다.
    - Pretrained language models: These methods use representations from language models for transfer learning. 트랜스퍼 러닝을 위해 구축된 랭기지 모델 표현을 사용한다.

Language Modeling은 sentence를 입력으로 한 the probability of the input sentence를 출력으로 한다. 동일한 방식으로 NLG로 활용 가능. 모든 조합 가능한 문장에 대해 prob를 계산할 수 있다면. (조경현, 2018)  
A language model captures the distribution over all possible sentences. 

$$p(X)=p((x_1,x_2,...,x_T))$$

## Multi-task learning
- [Multi-task learning](http://ruder.io/multi-task/) is a general method for sharing parameters between models that are trained on multiple tasks. 여러 작업에 대해 학습된 모델간에 파라미터를 공유하는 방식이다. By sharing representations between related tasks, we can enable our model to generalize better on our original task. 보다 더 잘 일반화 할 수 있다. 
- **Transfer Learning** (or Domain Adaptation 도메인 적응): Giving a set of source domains/tasks t1, t2, …, t(n-1) and the target domain/task t(n), the goal is to learn well for t(n) by transferring some shared knowledge from t1, t2, …, t(n-1) to t(n). 약간의 공유 지식을 전달하여 다른 도메인에 학습을 잘 하는 것이 목표. There are labeled training data for the source domain and few or no labeled examples in the target domain/task, but there are a large amount of unlabeled data in t(n). 타겟 도메인에 라벨링 되어 있지 않은 많은 데이터가 있다. [^fn-stats]

[^fn-stats]: <https://stats.stackexchange.com/a/255066/139406>

## Neural networks for NLP
RNN은 동적 입력 시퀀스를 다루는 곳 어디에나 쓰인다. Vanilla RNNs(Elman, 1990)은 LSTM(Hochreiter & Schmidhuber, 1997)으로 빠르게 전환됐다. BiLSTM(Graves et al., 2013), CNN(Kim et al., 2014)은 can also be combined and stacked.

## Sequence-to-sequence models
In 2014, Sutskever et al. proposed sequence-to-sequence learning.

## Attention
[Attention](http://ruder.io/deep-learning-nlp-best-practices/index.html#attention) (Bahdanau et al., 2015) is one of the core innovations in NMT. Multiple layers of self-attention are at the core of the Transformer architecture (Vaswani et al., 2017), the current state-of-the-art model for NMT.

## Memory-based networks
[Attention and Memory](http://www.wildml.com/2016/01/attention-and-memory-in-deep-learning-and-nlp/), Memory Networks (Weston et al., 2015).

## Pretrained language models
language models only require unlabelled text; training can thus scale to billions of tokens, new domains, and new languages. Pretrained language models were first proposed in 2015 (Dai & Le, 2015). Improvements with language model embeddings(ELMo) has archieved over state-of-the-art (Peters et al., 2018). [potential of pretrained language models](https://thegradient.pub/nlp-imagenet/)

# LM Perplexity
> Frederick Jelinek was a researcher in automatic speech recognition and He is well known for his oft-quoted statement, "Every time I fire a linguist, the performance of the speech recognizer goes up". (Wikipedia)

젤리넥은 조건부 엔트로피와 상대 엔트로피에서 출발해 언어 모델 복잡도 <sup>perplexity</sup>라는 개념을 정의함으로써 언어모델의 장단점을 직접 측정했다. 복잡도는 앞뒤 문맥이 주어졌다는 조건 아래, 문장 중 각 위치에서 평균적으로 선택 가능한 단어의 수를 말한다. 모델의 복잡도가 작을수록 위치별 단어가 확실하고 모델은 더 뛰어나다. (수학의 아름다움, 2014, 2019)

# BERT, RoBERTa, DistilBERT, XLNet — which one to use?
<img src="https://miro.medium.com/max/2486/1*5PzGl1dNt_5jMH3_mm3PpA.png" width="80%">[^fn-which]

[^fn-which]: <https://towardsdatascience.com/bert-roberta-distilbert-xlnet-which-one-to-use-3d5ab82ba5f8>
