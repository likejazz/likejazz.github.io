---
layout: wiki 
title: CNN, RNN
tags: ["Deep Learning"]
last_modified_at: 2021/06/08 13:03:45
---

<!-- TOC -->

- [CNN](#cnn)
    - [Deep Residual Network](#deep-residual-network)
    - [Fine-tuning Deep Learning Models](#fine-tuning-deep-learning-models)
        - [Fine-tuning Techniques](#fine-tuning-techniques)
- [RNN](#rnn)

<!-- /TOC -->

# CNN
## Deep Residual Network
통계학에서 residual <sup>잔차</sup>은 표본 <sup>sample</sup>에서의 오차를 말한다. 오차는 모수의 개념이므로 표본에서는 통계량의 개념을 갖는 다른 용어로 부르는데, 반면 ResNet에서 residual은 extra의 의미에 더 가깝다.

> Network depth is of crucial importance in neural network architectures, but deeper networks are more difficult to train. The residual learning framework eases the training of these networks, and enables them to be substantially deeper. [^fn-deep]

[^fn-deep]: <https://blog.waya.ai/deep-residual-learning-9610bb62c355>

residual이 학습을 쉽게 하여 더 깊은 네트워크 구성이 가능하게 한다. 비슷한 파라미터로 더 깊은 구성이 가능하다.

![](https://cdn-images-1.medium.com/max/1600/1*pUyst_ciesOz_LUg0HocYg.png)

## Fine-tuning Deep Learning Models
이미지 CNN, 여기서는 VGG의 경우 pre-trained 모델을 fine-tuning 하는 기법에 대해 설명한다.

> one would fine-tune existing networks that are trained on a large dataset like the ImageNet (1.2M labeled images) by continue training it (i.e. running back-propagation) on the smaller dataset we have. Provided that our dataset is not drastically different in context to the original dataset (e.g. ImageNet), the pre-trained model will already have learned features that are relevant to our own classification problem. [^fn-fine-tuning]

[^fn-fine-tuning]: <https://flyyufelix.github.io/2016/10/03/fine-tuning-in-keras-part1.html>

ImageNet 처럼 큰 데이터셋으로 학습된 모델에 우리가 가진 작은 데이터셋으로 학습을 계속해 pre-trained 모델을 fine-tuning 할 수 있다. 우리 데이터셋의 특징이 원래 데이터셋과(ImageNet)과 크게 다르지 않다면 pre-trained 모델은 우리의 분류 문제와 관련된 특징을 이미 학습했을 것이다.

### Fine-tuning Techniques
1. pre-trained 네트워크에서 ImageNet은 1000개 카테고리인데, 만약 우리가 10가지 카테고리라면 마지막 softmax를 10으로 변경한다.
1. Use a smaller learning rate. 랜덤 초기화와 달리 pre-trained 가중치는 이미 좋을 것이므로 너무 빨리 왜곡하지 않도록 initial learning rate를 1/10 수준으로 낮추는 것이 일반적이다.
1. freeze the weights of the first few layers. 처음 몇 레이어의 가중치 고정. 왜냐면 처음 몇 레이어는 curve, edge 같은 우리에게 여전히 도움되는 일반적인 정보를 학습하기 때문이다. 따라서 후속 레이어 학습에 보다 집중한다.

<img src="https://user-images.githubusercontent.com/1250095/48592808-7c4ece00-e98d-11e8-91b4-31f29ff100e2.jpeg" width="70%" />

(딥러닝 부트캠프 with 케라스, 2017)

<img src="https://user-images.githubusercontent.com/1250095/48592807-7bb63780-e98d-11e8-8da9-427037831b58.jpeg" width="70%" />

(딥러닝 부트캠프 with 케라스, 2017)

output layer를 제외한 모델을 feature extraction으로 활용해 SVM, XGBoost등을 적용할 수 있다.

# RNN
![](https://user-images.githubusercontent.com/1250095/47775617-81781000-dd33-11e8-8f58-9e431d8150e1.jpeg)
![](https://user-images.githubusercontent.com/1250095/47775618-81781000-dd33-11e8-9f3b-24cdf55a47de.jpeg)
![](https://user-images.githubusercontent.com/1250095/47775619-81781000-dd33-11e8-847e-6c88163e3c35.jpeg)
![](https://user-images.githubusercontent.com/1250095/47775620-81781000-dd33-11e8-9310-6a9ef4eb1128.jpeg)
![](https://user-images.githubusercontent.com/1250095/47775621-8210a680-dd33-11e8-99e5-fe43477491de.jpeg)
![](https://user-images.githubusercontent.com/1250095/47775622-8210a680-dd33-11e8-8e65-1a9521ede8a5.jpeg)

[^fn-coursera]

[^fn-coursera]: <https://www.slideshare.net/TessFerrandez/notes-from-coursera-deep-learning-courses-by-andrew-ng>

![](http://karpathy.github.io/assets/rnn/diags.jpeg)

1. Vanilla mode of processing without RNN, from fixed-sized input to fixed-sized output (e.g. image classification). 
1. Sequence output (e.g. image captioning takes an image and outputs a sentence of words). 비디오 캡션, 아티스트 기반으로 음악 플레이리스트 생성, 파라미터를 기반으로 한 멜로디 생성, 사진 속에서 보행자 위치 찾기
1. Sequence input (e.g. sentiment analysis where a given sentence is classified as expressing positive or negative sentiment). 날씨 예측, 음악 샘플 장르 구분, 영화 후기에 대한 감성 분석, 영화 시청 이력을 바탕으로 보고 싶어할 영화 확률 예측(데이터마이닝 에서는 collaborative filtering 구현)
1. Sequence input and sequence output (e.g. Machine Translation: an RNN reads a sentence in English and then outputs a sentence in French). 
1. Synced sequence input and output (e.g. video classification where we wish to label each frame of the video). NER 태깅

[^fn-karpathy]

[^fn-karpathy]: <http://karpathy.github.io/2015/05/21/rnn-effectiveness/>
