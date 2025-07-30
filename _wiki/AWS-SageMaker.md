---
layout: wiki 
title: AWS SageMaker
tags: ["Cloud"]
last_modified_at: 2024/10/24 00:35:36
---

- [AWS SageMaker](#aws-sagemaker)
  - [SageMaker Inference 테스트](#sagemaker-inference-테스트)
  - [Hugging Face SageMaker](#hugging-face-sagemaker)
    - [CLI 방식](#cli-방식)
    - [Inference SDK 예제](#inference-sdk-예제)
  - [Hugging Face SageMaker 문제점](#hugging-face-sagemaker-문제점)

# AWS SageMaker
- 학습 부터 배포까지 전체 데이터 플로우를 관리하는 주피터 노트북 기반 통합 제공한다. 
- 인퍼런스만 별도로 진행 가능(아래 참조)
- 모델 경량화 서빙은 SageMaker Neo

## SageMaker Inference 테스트
Inference만 별도 테스트 진행:
- Marketplace model packages에서 모델 구독. 여기서는 VARCO LLM KO-1.3B-IST 실험.
- Models에 모델 셋업
- Endpoint configurations에서 서버 설정. 서울 리전에서는 ml.g4dn.2xlarge만 가능. g5가 아직 지원되지 않지만 CLI를 이용해 편법으로 생성 가능하다.
- Endpoints에서 앞서 설정을 이용해 구성. 서비스 시작에 다소 시간이 소요된다.
```
$ aws sagemaker-runtime invoke-endpoint \
    --endpoint-name varco-endpoint \
    --body fileb://body \
    output_file.txt
```

CLI 호출은 REST가 아니라 파일 저장 방식이라 불편하다. SDK 사용(파이썬)을 권장한다.

```
# body
{
  "text": "세상에서 가장 위대한 운동 선수는 누구일까?",
  "request_output_len": 512,
  "repetition_penalty": 1.2,
  "temperature": 0.5
}
```

```
# output_file.txt
{"result":["세상에서 가장 위대한 운동 선수는 누구일까?\n\n### 응답:마이클 펠프스"]}
```

별도 인스턴스를 구동하는 provision 방식은 deploy에 한참 걸리며, 비용도 ec2 vanilla보다 더 비싸다.

## Hugging Face SageMaker

허깅페이스 이미지를 SageMaker에 올려서 서빙이 가능하다. [지원 DLC](https://github.com/aws/deep-learning-containers/blob/master/available_images.md). 패치한 경우 이미지를 수정하여 개인 ECR에 올리고 이 이미지를 SageMaker로 배포할 수 있다.

- PyTorch Inference DLC
- Text-Generation-Inference DLC(성능이 좋으나 현재 Llama 모델은 Tokenizer까지 맞춰야 한다)

모델은 S3에 tar.gz로 업로드 해서 다음과 같이 CLI로 생성한다.

### CLI 방식

SDK를 이용하면 VPC내에 LLM을 배포할 수 없다. 옵션이 존재하지 않음. philschmid의 글[^fn-vpc]에 가이드가 있으나 정작 VPC 관련 내용은 전혀 없다. 제목과 다른 내용으로, 실무 적용 사례가 아닌 가이드의 한계점이 엿보인다.

[^fn-vpc]: <https://www.philschmid.de/sagemaker-llm-vpc>

CLI에서는 다음과 같이 VPC를 명확하게 지정할 수 있다.

1. create-model
```bash
$ aws sagemaker create-model \
--model-name xxx-model-20230826 \
--primary-container 'Image=763104351884.dkr.ecr.ap-northeast-2.amazonaws.com/huggingface-pytorch-inference:2.0.0-transformers4.28.1-gpu-py310-cu118-ubuntu20.04,Mode=SingleModel,ModelDataUrl=s3://xxx-models/model.tar.gz,Environment={SAGEMAKER_CONTAINER_LOG_LEVEL=20,SAGEMAKER_REGION=ap-northeast-2}' \
--execution-role-arn arn:aws:iam::xxx:role/SageMakerExecutionRole \
--vpc-config 'SecurityGroupIds=sg-xxx,Subnets=subnet-xxx,subnet-xxx,subnet-xxx,subnet-xxx'
```
ap-northeast-2에서 `ml.g5.2xlarge` 인스턴스 생성이 막혀 있는데 CLI에서는 다음과 같이 강제 지정이 가능하다.
1. create-endpoint-configuration
```bash
$ aws sagemaker create-endpoint-config \
--endpoint-config-name xxx-configuration-20230826 \
--production-variants 'VariantName=default-variant,ModelName=xxx-model-20230826,InitialInstanceCount=1,InstanceType=ml.g5.2xlarge,InitialVariantWeight=1.0'
```

1. create-endpoint
```bash
$ aws sagemaker create-endpoint \
--endpoint-name xxx-20230826 \
--endpoint-config-name xxx-configuration-20230826
```

### Inference SDK 예제

```python
import sagemaker
import boto3

from sagemaker.huggingface import HuggingFaceModel
from sagemaker.huggingface.model import HuggingFacePredictor

ROLE_ARN = "arn:aws:iam::xxx:role/assume-chatbaker-sagemaker-inference"
ROLE_SESSION_NAME = "assume-chatbaker-sagemaker-inference"

session = boto3.Session(
    aws_access_key_id=os.getenv("AWS_ACCESS_KEY_ID"),
    aws_secret_access_key=os.getenv("AWS_SECRET_ACCESS_KEY"),
    region_name=os.getenv("AWS_REGION"))

def session_from_assume_role() -> boto3.Session:
    sts = session.client("sts")
    try:
        response = sts.assume_role(
            RoleArn=ROLE_ARN,
            RoleSessionName=ROLE_SESSION_NAME)
    except Exception as e:
        raise Exception(e)

    return boto3.Session(aws_access_key_id=response['Credentials']['AccessKeyId'],
                         aws_secret_access_key=response['Credentials']['SecretAccessKey'],
                         aws_session_token=response['Credentials']['SessionToken'])

predictor = HuggingFacePredictor(
    endpoint_name='chatbaker-d0.1.5-m0.0.1-r230814',
    sagemaker_session=sagemaker.Session(boto_session=session_from_assume_role()),
)

prompt = (f'{system_prompt}'
          f'<human>: 이틍 동안의 제주도 여행 계획 세워줘 <bot>:')

result = predictor.predict({
    'inputs': prompt,
    'parameters': {
        'do_sample': True,
        'temperature': 0.5,
        'repetition_penalty': 1.2,
        'top_p': 0.95,
        'top_k': 20,
        'max_new_tokens': 512,
    }
})
```

## Hugging Face SageMaker 문제점
- tgi 이미지는 `HF_MODEL_ID` 설정이 반드시 필요하다. safetensors를 생성해 model.tar.gz를 함께 만들고 위치를 `/opt/ml/model`로 지정한다. safetensors를 저장하지 않으면 read only file system이라며 실패한다. 또한 LlamaTokenizer를 사용하지 않으면 실패한다.
- sdk에서는 vpc 설정 옵션이 없다. cli에서 vpc 옵션을 부여할 수 있다.
- vpc를 설정하면 s3에 접근이 되지 않으므로 vpc > endpoint를 반드시 생성해야 한다.
  - 서비스는 s3를 검색하고 Endpoint type은 Gateway.
- 콘솔에서는 ap-northeast-2 리전에서 ml.g5.2xlarge가 선택이 되지 않는다. sdk 또는 cli에서만 강제 지정이 가능하다.
- 7B 모델을 단일 GPU에 올리기 위해서는 bfloat16 패치가 필요하다. `from_pretrained(..., torch_dtype=torch.bfloat16)` [스펙 산정 문서](https://www.philschmid.de/sagemaker-llama-llm).