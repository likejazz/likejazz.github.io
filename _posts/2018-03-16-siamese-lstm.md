---
layout: post
title: Siamese LSTM을 이용한 Quora 유사도 판별
tags: ["NLP"]
last_modified_at: 2018/03/16 12:06:24
---

<div class="message">
유사한 질문을 찾아내는 알고리즘을 고민하고 있던 중 마침 Kaggle에서 Quora의 유사 질문을 찾는 Quora Question Pairs Competition을 알게 되었고, 여기에서 평가가 매우 좋은 Siamese LSTM 네트워크를 이용한 문장의 의미론적 유사도를 판별하는 논문을 살펴보고 구현을 진행해본다.
</div>

<small>
*2018년 3월 16일 초안 작성*
</small>

<!-- TOC -->

- [서론](#서론)
- [데이터 및 모델](#데이터-및-모델)
  - [Kaggle's Quora Question Pairs](#kaggles-quora-question-pairs)
  - [MaLSTM](#malstm)
- [코드](#코드)
  - [모델](#모델)
  - [Keras 커스텀 레이어](#keras-커스텀-레이어)
- [학습 결과](#학습-결과)
- [결론](#결론)
- [참고](#참고)

<!-- /TOC -->

## 서론
[Siamese Recurrent Architectures for Learning Sentence Similarity](http://www.mit.edu/~jonasm/info/MuellerThyagarajan_AAAI16.pdf) 논문을 읽어보고 구현한 결과를 정리한다.

## 데이터 및 모델
### Kaggle's Quora Question Pairs
유사한 질문을 찾아내는 알고리즘을 고민하고 있던 중 마침 Kaggle에서 Quora의 유사 질문을 찾는 Competition을 진행했다는 것을 알게 됐다. Competition의 목표가 우리가 원하는 결과와 거의 유사했고, 여기에 올라온 다양한 접근 방법은 많은 도움이 되었다. 물론 1등 모델을 사용하면 성능은 가장 좋겠지만 무려 300여개의 앙상블로 구현했다고 하니, 상위권은 지나치게 impractical 했다.

### MaLSTM
재밌는 점은 상위권 대부분이 가르키는 모델이 있었는데, 바로 이 문서에서 리뷰할 논문에 나온 모델, MaLSTM 모델을 언급한다.

<img src="https://user-images.githubusercontent.com/1250095/38629567-5dcebe9e-3def-11e8-8c85-de15b8a16d73.jpeg" width="70%" />

MaLSTM은 맨하탄 거리<sup>Manhattan distance</sup>를 사용한 LSTM이라는 뜻인데, 무엇보다 모델이 단순하여 매우 실용적이다. LSTM 50개를 사용한게 전부인데, 여기서 핵심은 Siamese LSTM 네트워크에 맨하탄 메트릭<sup>Manhattan metric</sup>을 사용했다는 점이다.

<img src="https://cloud.githubusercontent.com/assets/9861437/20479493/6ea8ad12-b004-11e6-89e4-53d4d354d32e.png" />

similarity function은 단순한 구성으로, $$e^{-x}$$ 곡선을 따르도록 되어 있다. 즉, 거리가 0에 가까울 수록 결과는 1이 되고, 거리가 커질수록 결과는 0에 무한히 가까워 진다. 레이블 또한 의미론적으로 유사한 문장은 1이 부여되어 있고, 다른 문장은 0이 부여되어 별도의 activation function 없이 이 값을 일치시키는 형태로 레이어가 구성된다. cost function은 이 차이에 대한 MSE를 사용한다.

기존에 주로 사용하던 유클리드 거리는 크게 두 가지 문제가 있다고 한다.

1. 유사도를 판단하는 문제에 L2를 사용하면 objective function이 원치 않는 고원 형태<sup>undesirable plateaus</sup>가 될 수 있다. (Chopra, Hadsell, and LeCun, 2005) 즉, 학습이 늦고 minima를 찾기 힘들어 진다.
1. 학습 초기 단계에 L2는 유클리드 거리의 vanishing gradients로 인한 에러를 보정하기 어렵다. 의미론적으로 잘못 판단된 문장을 보정하기 어렵다고 한다.

논문에서는 다양한 실험을 했고, 맨하탄 거리를 사용하는 것이 코사인 유사도 등을 사용하는 것 보다 결과가 더 좋았다고 한다. 아래는 SICK 데이터셋에 포함된 연결 강도와 MaLSTM(맨 우측)의 비교다.

<img src="https://user-images.githubusercontent.com/1250095/37522462-572c2de8-2967-11e8-9bd0-d40a6d7e40b5.png" width="70%" />

MaLSTM은 실제 연결 강도와 매우 유사함을 확인할 수 있다. Richard Socher쪽에서 나온 논문인 Tree-LSTM과도 비교하는데, 당연히 이번에도 MaLSTM의 결과가 더 좋았다고 한다.

<img src="https://user-images.githubusercontent.com/1250095/37522514-85677910-2967-11e8-8878-e60ef7cf0fad.png" width="70%" />

Tree-LSTM은 부정의 의미를 판별하지 못한 문장도 MaLSTM이 제대로 판단하는걸 확인할 수 있다. (selectian bias가 아닌지 모르겠지만) 또한 SICK 데이터셋의 문장을 t-SNE로 표현해보면 아래처럼 동일 주제를 유사한 벡터로 잘 표현해낸다.

<img src="https://user-images.githubusercontent.com/1250095/37522577-b24d19f8-2967-11e8-8dae-2b6e737e1a5c.png" width="70%" />

텐서플로 구현이 논문과 함께 공개되었는데, 코드가 다소 지저분하고 정리되지 않아 단순한 모델임에도 불구하고 실험해보기가 쉽지 않았다. 논문을 함께 쓴 인도 학생도 깃헙을 공개했는데, 그쪽은 더 지저분.

Kaggle의 Quora Competition 쪽의 커널이 깔끔하게 잘 정리된게 많아서 그쪽을 계속 살펴보다가 한 데이터 과학자가 [미디엄에 Keras 구현](https://medium.com/mlreview/implementing-malstm-on-kaggles-quora-question-pairs-competition-8b31b0b16a07)을 올렸고 그걸 가장 많이 참조했다. 정말 알기쉽게 잘 설명하고 있다. Production을 목표로 하기 위해 Keras의 커스텀 레이어도 처음으로 만들어 봤고(생각보다 어렵지 않다) train/predict를 구분하고, 코드를 좀 더 정리해봤다.

## 코드

```python
# Load word2vec
print("Loading word2vec model(it may takes 2-3 mins) ...")
word2vec = KeyedVectors.load_word2vec_format(file, binary=True)

for index, row in df.iterrows():
    # Iterate through the text of both questions of the row
    for question in ['question1', 'question2']:

        q2n = []  # q2n -> question numbers representation
        for word in text_to_word_list(row[question]):
            # Check for unwanted words
            if word in stops:
                continue

            # If a word is missing from word2vec model.
            if word not in word2vec.vocab:
                if word not in vocabs_not_w2v:
                    vocabs_not_w2v_cnt += 1
                    vocabs_not_w2v[word] = 1

            # If you have never seen a word, append it to vocab dictionary.
            if word not in vocabs:
                vocabs_cnt += 1
                vocabs[word] = vocabs_cnt
                q2n.append(vocabs_cnt)
            else:
                q2n.append(vocabs[word])

        # Append question as number representation
        df.at[index, question + '_n'] = q2n

embeddings = 1 * np.random.randn(len(vocabs) + 1, embedding_dim)  # This will be the embedding matrix
embeddings[0] = 0  # So that the padding will be ignored

# Build the embedding matrix
for word, index in vocabs.items():
    if word in word2vec.vocab:
        embeddings[index] = word2vec.word_vec(word)
```

첫 실행시 시간이 다소 걸린다.

이는 구글의 word2vec 모델(압축해서 1.6G)을 로딩하는 시간인데, Production에는 이 부분의 최적화가 필요하다. 이후 불용어<sup>Stopwords</sup>를 제외하고, 모든 단어를 일련 번호로 표현하여 별도의 Pandas 컬럼에 업데이트 한다.

임베딩에는 word2vec의 300차원 임베딩이 들어가는데, 초기값은 랜덤하게 설정한다. 즉, 불용어를 제외한, word2vec 모델에 존재하지 않는 단어는 랜덤하게 임베딩된다.

### 모델
```python
# Since this is a siamese network, both sides share the same LSTM
shared_lstm = LSTM(n_hidden)

left_output = shared_lstm(encoded_left)
right_output = shared_lstm(encoded_right)
```

모델은 단순하지만 학습 시간이 많이 걸린다.

LSTM 자체가 구조의 복잡도로 인해 학습 속도가 늦고, 무엇보다 병렬 연산이 어려운 구조이기 때문에 GPU의 잇점을 살리기 어렵다. 그나마 여기서는 임베딩 할 문장 길이를 20개로 제한해 속도를 높였는데, 원래 논문에서는 문장의 임베딩을 가장 긴 문장의 길이로 처리하고 있고 이렇게 할 경우 학습이 훨씬 더 오래 걸린다. 임베딩 또한 300차원이라 적지 않은 입력 데이터가 들어가는데, 임베딩이 길면 대부분이 Zero Padding으로 처리되어 불필요한 학습 시간이 추가된다. 인풋 데이터는 left/right 양쪽 모두 고정 길이 벡터로 Zero Padding을 앞에<sup>pre</sup> 두어 이 경우 값을 무시하도록 했다.

### Keras 커스텀 레이어
맨하탄 거리를 계산하기 위해 Keras의 커스텀 레이어를 만들었다.

```python
class ManDist(Layer):
    """
    Keras Custom Layer that calculates Manhattan Distance.
    """

    # initialize the layer, No need to include inputs parameter!
    def __init__(self, **kwargs):
        self.result = None
        super(ManDist, self).__init__(**kwargs)

    # input_shape will automatic collect input shapes to build layer
    def build(self, input_shape):
        super(ManDist, self).build(input_shape)

    # This is where the layer's logic lives.
    def call(self, x, **kwargs):
        self.result = K.exp(-K.sum(K.abs(x[0] - x[1]), axis=1, keepdims=True))
        return self.result

    # return output shape
    def compute_output_shape(self, input_shape):
        return K.int_shape(self.result)
```

MATLAB에 맨하탄 거리를 계산하는 같은 메소드 명이 있어 동일하게 `ManDist` 레이어로 명명하여 맨하탄 거리를 계산했다. save 한 모델을 `load_model` 할때는 동일한 커스텀 레이어를 다시 지정해 주어야 모델 계산이 진행된다. 따라서 `util.py`에 별도로 정의했고 train/predict 모두 같은 모델을 `import` 하여 사용한다.

## 학습 결과
그렇게 50 epochs(NVIDIA Tesla P40 GPU x 2, batch size=1024*2)를 진행한 결과는 아래와 같다.
```
Epoch 50/50
363861/363861 [==============================] - 12s 33us/step - loss: 0.1172 - acc: 0.8486 - val_loss: 0.1315 - val_acc: 0.8229
Training time finished.
50 epochs in       601.24
```

Validation 셋으로 **82.29%**의 정확도가 나왔다. 

정확도는 Keras의 디폴트인 0.5를 기준으로 true/false만 판단하는데 그렇게 한 것 치고는 나쁘지 않다. odds ratio를 높게 설정한다던지 해서 out of domain 처리를 하면 훨씬 더 정확도를 높일 수 있을 것 같다. 참고로 학습 데이터인 Quora의 Question Pairs는 40만개 학습셋이 제공되었고, 이 중 10% 비율을 Validation에 할당하여 4만개로 평가 했다.

학습에는 NVIDIA Tesla P40 GPU 2장을 사용했는데, LSTM의 Sequential하게 처리되는 특성상 CPU에 비해 학습 속도가 높지 않다. 이 경우 배치 사이즈를 키우고 epochs를 늘리는 방향으로 GPU utilization을 높일 수 있다. 배치 사이즈가 커져도 속도 향상이 거의 없는 CPU와 달리 GPU는 배치 사이즈에 따른 속도 향상이 선형적으로 증가한다. 다만, [라지 배치에서는 모델의 품질 문제](https://stats.stackexchange.com/questions/164876/tradeoff-batch-size-vs-number-of-iterations-to-train-a-neural-network/236393#236393)가 있다고 하니 주의가 필요하다. 여기서는 배치 사이즈를 크게 하여 GPU의 잇점을 최대한 살리도록 했다.

## 결론
추후 Production을 위해 아래와 같은 최적화를 고민해볼 수 있다.
- 각 문장의 LSTM 결과를 미리 캐싱한다.
- near-duplicates 알고리즘을 사용해 후보군을 골라내어 확률이 높은 문장을 대상으로 비교 횟수를 줄인다.
- 임베딩을 최적화 한다.
- Keras로 빌드한 모델을 C++로 디코딩하여 CPU로 서비스 한다.

## 참고
전체 코드는 아래에서 확인할 수 있다.
- [Siamese-LSTM - GitHub](https://github.com/likejazz/Siamese-LSTM)
