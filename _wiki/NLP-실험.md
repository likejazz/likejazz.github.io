---
layout: wiki 
title: NLP 실험
tags: ["NLP"]
last_modified_at: 2025/05/17 19:55:07
---

<!-- TOC -->

- [Pororo](#pororo)
- [실험](#실험)
    - [MNIST](#mnist)
    - [IMDB Sentimental Analysis](#imdb-sentimental-analysis)
    - [Sentimental Analysis with ELMo](#sentimental-analysis-with-elmo)
    - [News Aggregator Dataset](#news-aggregator-dataset)
    - [Siamese-LSTM](#siamese-lstm)

<!-- /TOC -->
# Pororo
kakaobrain/pororo에서 MRC 태스크는 최신 데비안 계열에서는 동작하지 않는다. 좀 더 구체적으로는 python-mecab-ko 패키지의 오류로 MRC 태스크[^fn-mrc]를 실행할 수 없었다.

[^fn-mrc]: <https://github.com/kakaobrain/pororo/blob/master/examples/reading_comprehension.ipynb>

```python
import pororo
from pororo import Pororo

mrc = Pororo(task="mrc", lang="ko")
```

```
Traceback (most recent call last):
  File "/home/gcp-user/.local/lib/python3.8/site-packages/pororo/tasks/machine_reading_comprehension.py", line 61, in load
    import mecab
ModuleNotFoundError: No module named 'mecab'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/gcp-user/.local/lib/python3.8/site-packages/pororo/pororo.py", line 203, in __new__
    task_module = SUPPORTED_TASKS[task](
  File "/home/gcp-user/.local/lib/python3.8/site-packages/pororo/tasks/machine_reading_comprehension.py", line 63, in load
    raise error.__class__(
ModuleNotFoundError: Please install python-mecab-ko with: `pip install python-mecab-ko`
```

설치 시도했으나 python-mecab-ko 설치에 실패한다. `libiconv.so`가 최신 버전에서 통합된 영향을 받는 것으로 보이나 정확한 원인은 알 수 없다. mecab 패키지도 문의에 응답이 없고 더 이상 유지보수 되지 않고 있다.

# 실험
2019년에 진행했던 다양한 NLP 실험

## MNIST
PyTorch를 이용한 MNIST 실험 결과 [1](https://github.com/yunjey/pytorch-tutorial/tree/master/tutorials/02-intermediate)

| 모델 | Test Accuracy |
| -- | -- |
| LSTM | 97.84 |
| BLSTM | 98.84 |
| CNN | 99.18 |

GPU에서는 CNN이 당연히 가장 빠르다.

## IMDB Sentimental Analysis
- [IMDB Bidirectional LSTM](https://github.com/fchollet/keras/blob/master/examples/imdb_bidirectional_lstm.py)  
Accuracy: 0.83
- [IMDB CNN LSTM](https://github.com/fchollet/keras/blob/master/examples/imdb_cnn_lstm.py)  
```
Epoch 2/2
25000/25000 [==============================] - 40s 2ms/step - loss: 0.1986 - acc: 0.9247 - val_loss: 0.3426 - val_acc: 0.8574
```
성능이 낮은 이유는 Keras 공식 예제는 `max_len`이 100으로 설정되어 있기 때문이다. 이를 400으로 늘리면 아래 Vanilla CNN과 같이 89%가 나온다.
```
Epoch 10/10
25000/25000 [==============================] - 4s 148us/step - loss: 1.6782e-05 - acc: 1.0000 - val_loss: 0.6014 - val_acc: 0.8939
```
- IMDB Vanilla CNN  
기본 예제가 이미 `max_len=400`으로 설정되어 있어 처음부터 성능이 더 좋게 나온다. GPU에서는 속도도 빠르다. 굳이 LSTM이나 여러 variants를 시도할 필요가 없다.
```
Epoch 20/20
25000/25000 [==============================] - 5s 208us/step
loss: 0.0106 - acc: 0.9998 - val_loss: 0.2991 - val_acc: 0.8987
```
CNN이 얕을수록 오히려 결과는 더 좋은데, 이는 분류 문제의 경우 독립 가정의 성능이 더 좋은 것으로 가정해볼 수 있다. 실제로 NB로 분류해도 90%가 나오는데, CNN이 얕다는 것은 NB와 거의 유사한 효과를 나타낼 것이고, 여러개의 필터로 문장 전체를 스캔하는 것 보다 이처럼 독립 가정에 가까울수록 훨씬 더 좋은 결과를 낸다고 볼 수 있다.

IMDB CNN 분류 예제에서 LSTM으로 해보면 학습이 되지 않는데(중간에 오버슈팅 발생) activation으로 인한 문제였다. activation을 제거하고 Dense로 연결하면 정상적으로 학습되며 LSTM의 결과(마지막 activation 필요)는 CNN 보다 조금 낮은 86~87% 근처에 수렴한다. 그러나 CNN 보다 LSTM이 학습이 더 잘되는 것 같다. CNN은 학습을 진행할수록 오히려 validation acc가 감소하는 경우가 잦다.

```
10 epochs/CNN/filters=300(3)/dense=250/GlobalMaxPooling1D 0.8936
20 epochs/CNN/filters=300(3,4,5)/dense=250/Flatten 0.8951 0.8681
20 epochs/CNN/filters=300(3,4,5)/dense=250/GlobalMaxPooling1D 0.8960
20 epochs/CNN/filters=300(3,4,5,6)/dense=250/GlobalMaxPooling1D 0.8941
20 epochs/CNN/filters=300(3,4,5,6,7)/dense=250/GlobalMaxPooling1D 0.8957
10 epochs/CNN/filters=300(3,3,3,3,3)/dense=250/GlobalMaxPooling1D 0.8942
20 epochs/CNN/filters=1000(3,3,3)/dense=250/GlobalMaxPooling1D 0.8887(max: 0.8987)
10 epochs/CNN/filters=1000(3,3,3,3,3)/dense=250/GlobalMaxPooling1D 0.8974
30 epochs/CNN/filters=300(3,3,3)/dense=250/Flatten 0.8573(max: 0.8877)
30 epochs/CNN/filters=300(3,3,3)/dense=250/Flatten/MaxPool1D 0.8671(max: 0.8765)
30 epochs/CNN/filters=300(3,4,5)/dense=250/Flatten/MaxPool1D 0.8666(max: 0.8748)
20 epochs/CNN/filters=300(3,3,3)/dense=250/GlobalMaxPooling1D 0.8878(max: 0.8959)
10 epochs/CNN/filters=300(3,3,3)/dense=250/GlobalMaxPooling1D/MaxPool1D 0.8800(max: 8961)
10 epochs/CNN/filters=300(3,3,3 sequential)/dense=250/GlobalMaxPooling1D 0.8460(max: 0.8664)
12 epochs/CNN/filters=1000(3,3,3)/dense=250/GlobalMaxPooling1D 0.8866(max: 0.8956)
12 epochs/CNN/filters=250(3,3 x 3)/dense=250/GlobalMaxPooling1D 0.8835(max: 0.8835) Reduce size due to resource problems.
12 epochs/CNN/filters=250(3,3,3 x 3)/dense=250/GlobalMaxPooling1D 0.8848(max: 0.8848)
20 epochs/CNN/filters=250(3,3,3,3,3 x 3)/dense=250/GlobalMaxPooling1D 0.8838(max: 0.8887)

30 epochs/LSTM(50)/dropout=0.3 0.8525
30 epochs/LSTM(50) 0.8452(max: 0.8675)
50 epochs/LSTM(50) 0.8545(max: 0.8684)
30 epochs/LSTM(100)/dropout=0.3 0.8603
30 epochs/LSTM(50)/no dropout,activation 0.8210
50 epochs/LSTM(50)/no dropout,activation 0.5049(overshooting, max:0.8160)
50 epochs/LSTM(50)/dropout=0.3/no activation 0.7164(max: 0.7626)
```

Flatten은 (None,595,300)을 (None,178500)으로 만들고 GlobalMaxPooling1D는 (None,300)으로 만든다. Flatten의 경우 CNN 초기 학습이 잘 안되면서 validation acc가 떨어지는 경우가 있다.

## Sentimental Analysis with ELMo
[튜토리얼](https://towardsdatascience.com/elmo-embeddings-in-keras-with-tensorflow-hub-7eb6f0145440)을 따라 [코드](https://github.com/likejazz/jupyter-notebooks/blob/master/deep-learning/elmo.py) 구현
```
# GPU: Tesla V100
Epoch 5/5
25000/25000 [==============================] - 587s 23ms/step - loss: 0.3722 - acc: 0.8304 - val_loss: 0.3904 - val_acc: 0.8206

```
## News Aggregator Dataset
[News Aggregator Dataset](https://www.kaggle.com/uciml/news-aggregator-dataset)을 이용한 multi-class 분류를 진행할때 [기존 머신러닝](https://nbviewer.jupyter.org/github/likejazz/jupyter-notebooks/blob/master/machine-learning/news-classification.ipynb)으로 Decision Trees 80%, Random Forest 84%, Multinomial Naive Bayes 90%를 기록했다.

```
# defaults
dropout=0.5/dense=250/GlobalMaxPooling1D

10 epochs/CNN/filters=250(3,3,3) 0.9485(max: 0.9505)
10 epochs/CNN/filters=250(3,3,3 x 3) 0.9447(max: 0.9482)
10 epochs/CNN/filters=250(3) 0.9479(max: 0.9510)
10 epochs/CNN/filters=250(3)/w2v 0.9483(max: 0.9496)
```

## Siamese-LSTM
[GPU에서 LSTM 학습 속도](http://likejazz.com/siamese-lstm/#%ED%95%99%EC%8A%B5-%EA%B2%B0%EA%B3%BC)와 상기 모델을 이용한 여러 파라미터의 실험 결과는 아래와 같다. train.csv를 이용해 word2vec을 직접 구축해보았으나 랜덤 임베딩과 별 차이가 없다. 보기에는 노이즈가 많아 보였지만 구글에서 제공하는 word2vec 모델은 1% 정도 성능이 더 높게 나온다. Kaggle Competition의 1등 결과는 log loss 0.1157로, acc로 환산하면 0.8577이 된다.

```
# defaults
w2v false, LSTM(50), max_seq=20, batch=1024

30 epochs 0.8075
50 epochs 0.8144
70 epochs 0.8156
100 epochs 0.8144

100 epochs/GRU 0.7357
200 epochs/GRU/max_seq=50/batch=4096 0.7402

100 epochs/LSTM(10) 0.7919
100 epochs/LSTM(100) 0.8251
100 epochs/LSTM(200) 0.8131
200 epochs/LSTM(200) overshooting

100 epochs/padding=post 0.7987
100 epochs/max_seq=10 0.8104
100 epochs/max_seq=30 0.8148
100 epochs/max_seq=40 0.8144
150 epochs/max_seq=40/batch=4096 overshooting(81.85)

30 epochs/w2v 0.8182
50 epochs/w2v 0.8263
100 epochs/w2v 0.8260
100 epochs/w2v/LSTM(100) 0.8279
100 epochs/w2v/max_seq=40 0.8218
150 epochs/w2v/max_seq=40/batch=4096 0.8232
200 epochs/w2v/LSTM(100)/max_seq=50/batch=4096 0.8271

50 epochs/w2v-self-trained 0.8211
50 epochs/w2v-self-trained/LSTM(100)/max_seq=50 0.8182
50 epochs/w2v 0.8237
50 epochs 0.8130
50 epochs/w2v 100dims 0.8024 dim100s is not good at learning properly.
50 epochs/w2v 500dims 0.8170 embedding time is very slow.

20 epochs/w2v-random/trainable=False 0.8083(max: 0.8092)
50 epochs/w2v-random/trainable=False 0.8151(max: 0.8160)
20 epochs/w2v-random/trainable=True 0.8267(max: 0.8267)
50 epochs/w2v-random/trainable=True 0.8378(max: 0.8383)
20 epochs/w2v-self-trained/trainable=False 0.8080(max: 0.8082)
20 epochs/w2v-self-trained/trainable=True 0.8363(max: 0.8363)
50 epochs/w2v-self-trained/trainable=True 0.8420(max: 0.8426)
20 epochs/w2v-google/trainable=False 0.8157(max: 0.8158)
50 epochs/w2v-google/trainable=False 0.8232(max: 0.8232)
20 epochs/w2v-google/trainable=True 0.8491(max: 0.8493)
50 epochs/w2v-google/trainable=True 0.8498(max: 0.8513)

CNN/filters=250/kernel_size=5 0.5722
CNN/filters=250/kernel_size=5/w2v 0.5643
CNN/filters=250/kernel_size=10/w2v 0.5681
CNN/filters=250/kernel_size=5/dropout=0.3 0.5768
CNN/filters=1000/kernel_size=5/dropout=0.3 0.5828
CNN/filters=1000/kernel_size=5/dropout=0.3/dense=(500,250) 0.3720
```
