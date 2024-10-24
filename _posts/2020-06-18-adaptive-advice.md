---
layout: post
title: ! 'Adaptive Advice in Automobile 논문 리뷰'
tags: ["Machine Learning"]
last_modified_at: 2020/06/18 00:00:00
---

<div class="message">
AAAI 2015 워크샵에서 발표한 Adaptive Advice in Automobile Climate Control Systems 논문을 리뷰하여 정리한다.
</div>

<small>
*2020년 6월 18일 초안 작성*
</small>

<!-- TOC -->

- [정보](#정보)
- [용어](#용어)
- [리뷰](#리뷰)
    - [Introduction](#introduction)
    - [Climate Control System](#climate-control-system)
    - [Energy Consumption Model](#energy-consumption-model)
    - [Prediction Model](#prediction-model)
    - [Conclusions](#conclusions)

<!-- /TOC -->

## 정보
[AAAI 논문 링크](https://www.aaai.org/ocs/index.php/WS/AAAIW15/paper/viewFile/9832/10159)

## 용어
- CCS: Climate Control Systems
- MACS: MDP Agent for Climate control Systems
- SAP: Social Advice Providing agent

MACS는 억지로 만든 감이 있다. 초록에 MACS로 인해 에너지 소비를 33% 감소 시켰다는데, 자동으로 항상 약하게 틀어주면 에너지 소비가 감소되는거 아닌가.  
(그래도 Introduction에 보면 while keeping the driver comfortable 이라는 전제 조건이 있긴 하다)

## 리뷰
1저자인 Ariel Rosenfield는 이 내용을 기반으로 Predicting Human Decision-Making이라는 책[^fn-book]도 썼다. (2018)

[^fn-book]: <https://www.amazon.com/Predicting-Human-Decision-Making-Prediction-Intelligence/dp/1681732742>

### Introduction
Introduction에 나오지만 car's climate control system reduces the cars power efficiency by up to 10% 라고 한다. 그만큼 잘 만든 CCS가 중요하다는 얘기.

<img src="https://user-images.githubusercontent.com/1250095/84968838-57ff3880-b152-11ea-8d35-64493de8fc84.png" width="70%">

### Climate Control System
쉐보레 볼트 전기차로 실험했고, `(T, F, D, M)`으로 구성된 튜플을 활용했다.

<img width="70%" src="https://user-images.githubusercontent.com/1250095/84968847-5cc3ec80-b152-11ea-9a08-2e74991ed410.png">

- T 기온
- F 팬 강도 strength 1에서 6단계 까지
- D Air delivery 방향
  - 얼굴 face
  - 얼굴 & 발
  - 발
  - 앞유리 & 발  
  이 있는데, 이 논문에서는 얼굴(D: 0), 얼굴 & 발(D: 1) 두 가지 경우로 제한했다고 한다.
- M 모드 echo(M: 0), comfort(M: 1) 두 가지

추가로 중요한 외부 요인으로,
- E 외부 온도
- I 실내 온도(앞좌석)

We denote these two parameters together as world state v ∈ V , where V is the set of all possible world states.

### Energy Consumption Model
<img width="70%" src="https://user-images.githubusercontent.com/1250095/84968848-5d5c8300-b152-11ea-8926-e4ed11290697.png">

Given a setting `s`, we use subscript $$s_T$$ to refer to the temperature in that setting,  
$$s_F$$ to refer to the fan strength,  
$$s_D$$ for the air delivery and $$s_M$$ for the mode of the setting.

`e(T,F,D,M,E,I)`는 하나를 제외한 모든 features에 proportional <sup>비례</sup>하다. 단, T는 음수 값을 취하며 inversely proportional <sup>반비례</sup>하다.

<img width="70%" src="https://user-images.githubusercontent.com/1250095/84968849-5e8db000-b152-11ea-9e7e-a2dde710272e.png">

Advised Screen은 저렇게 cancel / accept를 선택하도록 되어 있다고 함. $$P(Accept\mid{x})$$, $$P(Reject\mid{x})$$의 확률을 각각 구했다.

### Prediction Model
<img width="70%" src="https://user-images.githubusercontent.com/1250095/84968850-5e8db000-b152-11ea-9f29-6f2baeac98d0.png">

Prediction Model은 MACS, SAP 두 가지 모델이고 여기에 Silent와 함께 비교. MACS는 MDP를 사용했다. 기본적으로 모두 KNN으로 학습. SVM과 Decision Tree 보다 성능이 더 좋았다고. 그래봤자 최종 성능이 78% 밖에 안되서 좋은 성능이라고 할 수 없다. 당시에는 Boosting이나 XGBoost를 아직 잘 쓰지 않던 시절이라 비교하지 못한 걸로 보인다. (XGBoost는 2014년에 릴리즈 됨)

MACS tries to solve the optimization problem given in:  
$$
\pi^{*}\left(v, s, h^{t, i-1}, t, i\right)=\underset{d}{\operatorname{argmin}} E C_{S}^{t, i}\left(v, s, h^{t, i-1}, d\right)
$$

Where $$\pi^{*}$$ is the advice function.

<img width="70%" src="https://user-images.githubusercontent.com/1250095/84968851-5f264680-b152-11ea-8d33-49075e24675b.png">

MACS에 대한 설명. DP로 구현했다고 한다. 입력 데이터는 `I=35, E=36, T=19, F=2, D=0, M=0`과 같은 식이고 출력 결과는 `T=22, F=1, D=0, M=0`과 같은 식이다.

<img width="70%" src="https://user-images.githubusercontent.com/1250095/84968852-5fbedd00-b152-11ea-9826-b1ae636c8a8e.png">

SAP 에이전트는 `u()`를 maximize한다. $$P(accept\mid{x})$$가 커야 하므로 `w`가 작을 수록 좋다. 마찬가지로 에너지 소비 함수 `e()`도 `-w` 이므로 `w`가 작아야 한다. `w`와 `u()`는 inversely propotional 관계다.

SAP modeling provides an advice that maximizes a social utility function which is a weighted sum of the agent and human’s utilities. SAP uses simulation runs of repeated human-agent interaction to identify the weights that maximize the agent’s utility over time.

### Conclusions
<img src="https://user-images.githubusercontent.com/1250095/84968854-60577380-b152-11ea-965e-48ce1bf4c719.png" width="70%">

지금까지 정리. SAP 모델이 MDP(MACS) 보다 성능이 더 좋다. 최종 성능은 78% (만족스러운 성능이 아니다) 또한 SAP agent was much more aggressive in its suggestions than MACS 라고.
