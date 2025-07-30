---
layout: wiki 
title: Keras
tags: ["Deep Learning"]
last_modified_at: 2023/11/26 17:49:02
---

<!-- TOC -->

- [Merge vs. Concatenate](#merge-vs-concatenate)
- [Optimizer](#optimizer)
- [Tokenizer](#tokenizer)
  - [keras.backend.ndim](#kerasbackendndim)
- [속도](#속도)
- [Remove TensorFlow CUDA Message](#remove-tensorflow-cuda-message)

<!-- /TOC -->

# Merge vs. Concatenate
`Merge`는 두 레이어의 element-wise sum을 한다.
```
x = Merge()([x1, x2])
```
`Merge`는 deprecated 되었다며 `merge` 사용을 유도한다. 그러나 이 또한 내부적으로 `Merge`를 호출하여 경고 메시지가 동일하게 출력된다. `merge` 함수를 사용하면 생성자를 기입하지 않아도 되어 조금 더 직관적이다.
```
x = merge([x1, x2])
```

아래와 같은 형태로 multiply 할 수도 있다.
```
attention_mul = merge([inputs, attention_probs], name='attention_mul', mode='mul')
```

`Concatenate`는 레이어를 길게 이어 붙인다.
```
x = Concatenate()([x1, x2])
```
마찬가지로 `concatenate`로 좀 더 직관적으로 사용할 수 있다.

# Optimizer
![](https://user-images.githubusercontent.com/1250095/48181704-e9e67300-e36b-11e8-8fee-cca7ea5f1bf0.jpeg)

(케라스 창시자에게 배우는 딥러닝, 2017)

# Tokenizer
```
from keras.preprocessing.text import Tokenizer

# corpus = "I like playing football with my friends"
corpus = ["I like playing football with my friends"]

tokenizer = Tokenizer()
tokenizer.fit_on_texts(corpus)
corpus_tokenized = tokenizer.texts_to_sequences(corpus)

# [[1, 2, 3, 4, 5, 6, 7]]
```

`corpus`가 string 일때는 char level, array 일때는 띄어쓰기 단위로 tokenize 된다.

## keras.backend.ndim
Returns the **number of axes** in a Tensor, as an Integer.

# 속도
Simple MNIST convnet[^fn-conv]

| 시스템 | Processing Unit | Speed |
| ----- | --------------- | ----- |
| 맥북 | CPU(2.6 GHz 6-Core Intel Core i7) | 15s/it |
| GCP | CPU(2x Intel(R) Xeon(R) CPU @ 2.20GHz) | 32s/it |
| GCP | GPU(Tesla T4) | 2s/it |

[^fn-conv]: <https://keras.io/examples/vision/mnist_convnet/>

# Remove TensorFlow CUDA Message
```
export TF_CPP_MIN_LOG_LEVEL=3
```