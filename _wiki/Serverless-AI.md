---
layout: wiki 
title: Serverless AI
tags: ["Cloud"]
last_modified_at: 2024/10/24 00:35:44
---

<!-- TOC -->

- [AWS Personalize](#aws-personalize)

<!-- /TOC -->

# AWS Personalize
- [가이드](https://docs.aws.amazon.com/personalize/latest/dg/gs-prerequisites.html)에 따라 진행
- IAM 생성 권한이 없어서 Dataset 생성에서 더 이상 진행되지 않음.
- MovieLens 예제는 rating 컬럼을 제거하는데, 그렇다면 Get Recommendations에서 무작위로 보여주는게 아닌가? (원래 MovieLens 데이터는 rating, user, item 정보 모두 포함되어 있음) 나라면 10% 정도를 A/B 테스트 용도로 할당하고 피드백을 받아 1/10 보다 CTR이 높은 경우 대체하는 방식으로 구현할 것 같은데 비율 확인 필요. 여기서도 SDK 또는 JS 라이브러리로 사용자 이벤트를 추적하는 것 같다.
