---
layout: wiki 
title: Markov Decision Process
tags: ["Machine Learning"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [정의](#정의)
- [MP, MRP, MDP](#mp-mrp-mdp)
- [MDP vs FSM](#mdp-vs-fsm)
- [Reinforcement Learning](#reinforcement-learning)

<!-- /TOC -->

# 정의

> In mathematics, a Markov decision process (MDP) is a discrete-time stochastic control process. It provides a mathematical framework for modeling decision making in situations where outcomes are partly random and partly under the control of a decision maker. MDPs are useful for studying optimization problems solved via dynamic programming and reinforcement learning. (Wikipedia)

MDP는 이산 시간 확률 제어 프로세스다. 결과가 부분적으로 랜덤이고 부분적으로 의사 결정자가 통제하는 상황에서 의사 결정을 모델링하기 위한 수학적 프레임워크를 제공한다. MDP는 DP나 강화학습을 통한 최적화 문제를 연구하는데 유용하다.

순차적으로 행동을 결정해야 하는 문제를 풀기 위한 수학 모델이다. 

강화학습은 MDP로 정의된 문제를 푸는 것과 같으며, 다음과 같이 표시한다.

- $$S$$: State
- $$A$$: Action
- $$P$$: state transition Probability
- $$R$$: Reward
- $$\gamma$$: discount factor

가치함수  
에이전트가 어떤 정책이 더 좋은 정책인지 판단하는 기준이 가치함수다.
가치함수는 현재 상태로부터 정책을 따라갔을 때 받을 것이라 예상되는 보상의 합이다.

에이전트는 정책을 업데이트할 때 가치함수를 사용하는데 보통 상태가치함수($$V_\pi$$)보다 에이전트가 선택할 각 행동의 가치를 직접적으로 나타내는 행동가치함수($$Q_\pi$$), 큐함수를 사용한다.

$$
Q_{\pi}(s_{t}, a)=r(s, a, s^{\prime})+\gamma V_{\pi}(s^{\prime})
$$

$$r(s, a, s^{\prime})$$는 상태 $$s$$에서 행동 $$a$$를 선택해 상태 $$s^\prime$$으로 이동했을 때 받는 보상

벨만 방정식  
현재 상태의 가치함수와 다음 상태 가치함수의 관계식

# MP, MRP, MDP
『바닥부터 배우는 강화학습, 2020』
<img src="https://user-images.githubusercontent.com/1250095/97730319-0f4f8700-1b17-11eb-9cba-a3c2859cab70.png" width="60%">
<img src="https://user-images.githubusercontent.com/1250095/97730298-0959a600-1b17-11eb-9ce5-cc9ed20e817f.png" width="60%">
<img src="https://user-images.githubusercontent.com/1250095/97730312-0e1e5a00-1b17-11eb-962b-73506b1bcb06.png" width="60%">
에이전트가 포함된 MDP가 되었다. $$a_0$$의 확률을 생략한 이유는 100%이기 때문.
<img src="https://user-images.githubusercontent.com/1250095/97730316-0eb6f080-1b17-11eb-82bb-6b5aa697d739.png" width="60%">

# MDP vs FSM
Whilst a Markov chain is a finite state machine, it is distinguished by its transitions being stochastic, i.e. random, and described by probabilities. [^fn-so]

MDP는 확률적 전이라는 점이 다르다.

[^fn-so]: <https://stackoverflow.com/a/4880336>

# Reinforcement Learning
The environment is typically stated in the form of a Markov decision process (MDP), because many reinforcement learning algorithms for this context utilize dynamic programming techniques. [^fn-wiki]

강화 학습의 환경은 MDP 형태로 표현한다. 많은 강화 학습 알고리즘이 다이나믹 프로그래밍을 사용하기 때문이다.

[^fn-wiki]: <https://en.wikipedia.org/wiki/Reinforcement_learning>

The main difference between the classical dynamic programming methods and reinforcement learning algorithms is that the latter do not assume knowledge of an exact mathematical model of the MDP and they target large MDPs where exact methods become infeasible.

강화 학습을 사용하는 후자는 MDP의 정확한 수학적 모델을 가정하지 않고, 정확한 방법을 찾기 어려운 큰 MDP를 대상으로 한다.
