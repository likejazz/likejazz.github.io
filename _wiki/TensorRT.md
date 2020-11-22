---
layout: wiki 
title: TensorRT
last-modified: 2020/11/22 14:45:50
---

<!-- TOC -->

- [TensorRT & Inferences](#tensorrt--inferences)
- [Tutorial](#tutorial)
- [Training](#training)
- [NLP w/ BERT](#nlp-w-bert)
- [Rapids](#rapids)
- [Optimization](#optimization)

<!-- /TOC -->

당분간 링크 중심으로 정리한다.

# TensorRT & Inferences
- [TensorRT Integration Speeds Up TensorFlow Inference](https://devblogs.nvidia.com/tensorrt-integration-speeds-tensorflow-inference/)  
TensorFlow v1.7 and above integrates with TensorRT 3.0.4. 
- [TensorRT를 활용한 딥러닝 Inference 최적화](https://www.slideshare.net/deview/232-dl-inference-optimization-using-tensor-rt-1-119162975)
- [NVIDIA TensorRT Inference Server Boosts Deep Learning Inference](https://devblogs.nvidia.com/nvidia-serves-deep-learning-inference/)
- [How to Speed Up Deep Learning Inference Using TensorRT](https://devblogs.nvidia.com/speed-up-inference-tensorrt/)

# Tutorial
- [Object Detection on GPUs in 10 Minutes](https://devblogs.nvidia.com/object-detection-gpus-10-minutes/)

# Training
- [Video Series: Mixed-Precision Training Techniques Using Tensor Cores for Deep Learning](https://devblogs.nvidia.com/video-mixed-precision-techniques-tensor-cores-deep-learning/)
- [Mixed-Precision Training of Deep Neural Networks](https://devblogs.nvidia.com/mixed-precision-training-deep-neural-networks/)

# NLP w/ BERT
- [NVIDIA Announces TensorRT 6; Breaks 10 millisecond barrier for BERT-Large](https://news.developer.nvidia.com/tensorrt6-breaks-bert-record/)
- [Real-Time Natural Language Understanding with BERT Using TensorRT](https://devblogs.nvidia.com/nlu-with-tensorrt-bert/)
    - [TensorRT/demo/BERT](https://github.com/NVIDIA/TensorRT/tree/release/5.1/demo/BERT) BERT example using TensorRT C++ API
    - [DeepLearningExamples/FasterTransformer](https://github.com/NVIDIA/DeepLearningExamples/tree/master/FasterTransformer) NVIDIA Korea는 이 버전으로 움직인다고.
- [ADDING A CUSTOM CUDA C++ OPERATIONS IN TENSORFLOW FOR BOOSTING BERT INFERENCE](https://on-demand-gtc.gputechconf.com/gtcnew/sessionview.php?sessionName=skr9108-adding+a+custom+cuda+c%2b%2b+operations+in+tensorflow+for+boosting+bert+inference) 이민석 과장님이 NVIDIA 2019에서 발표한 BERT 최적화 자료. 당시 현장에서 가장 인상적인 발표였다.

# Rapids
- [rapidsai/notebooks-contrib](https://github.com/rapidsai/notebooks-contrib/tree/master/getting_started_notebooks/intro_tutorials)

# Optimization
- [TensorFlow Performance Logging Plugin nvtx-plugins-tf Goes Public](https://devblogs.nvidia.com/tensorflow-performance-logging-plugin-nvtx-plugins-tf-public/) tf 프로파일링
    - [Keras 연동](https://nvtx-plugins.readthedocs.io/en/latest/templates/examples.html) 데이터셋은 Kaggle에 있음. 프로파일링을 위한 [NVIDIA Nsight Systems](https://developer.nvidia.com/nsight-systems)
