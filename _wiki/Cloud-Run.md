---
layout: wiki 
title: Cloud Run
last-modified: 2020/12/04 02:19:06
---

<!-- TOC -->

- [Cloud Build](#cloud-build)
- [Container Registry](#container-registry)
- [Cloud Run](#cloud-run)
    - [Troubleshooting](#troubleshooting)

<!-- /TOC -->

가이드[^fn-guide]에 따라 진행. Dockerfile을 만드는데 까지는 동일하다.

[^fn-guide]: <https://cloud.google.com/run/docs/quickstarts/build-and-deploy>

# Cloud Build
그러나, 가이드에서는 로컬에서 docker를 build 하지 않고 다음과 같이 Cloud Build에 등록한다.
```
$ gcloud builds submit --tag asia.gcr.io/PROJECT_ID/helloworld
```

전송된 Dockerfile을 Kaniko로 빌드한다. `--tag` 지정으로 `docker build`가 수행된다. Cloud Build will run a remote `docker build -t $TAG .`[^fn-help]

[^fn-help]: `gcloud builds submit --help`

- `gcr.io`는 이미지를 미국의 데이터 센터에서 호스팅한다.
- `us.gcr.io`는 이미지를 미국의 데이터 센터에서, `gcr.io`에서 호스팅하는 이미지와 별도의 스토리지 버킷에 호스팅한다.
- `eu.gcr.io`는 유럽 연합 내 이미지를 호스팅한다.
- `asia.gcr.io`는 이미지를 아시아의 데이터 센터에서 호스팅한다.

# Container Registry

이후 자동으로 Container Registry에 업로드된다. 빌드 & 배포를 한 번에 처리한다. Cloud Build가 진행되기 전 소스코드는 tgz로 압축되어 GCS에 등록된다. Container Registry에 있는 이미지는 GCE, GKE, Cloud Run에 각각 편리하게 선택하여 배포 가능하다. GCE는 모든 포트가 맵핑되어 있으며, GKE는 직접 K8s 기본 클러스터까지 생성해준다.

# Cloud Run

```
$ gcloud run deploy --image asia.gcr.io/PROJECT_ID/helloworld --platform managed
```
`--platform managed`로 Cloud Run으로 실행한다.

기본 실행 region 설정은 다음과 같이 미리 지정할 수 있다.
```
$ gcloud config set run/region asia-northeast3
```

## Troubleshooting

Flask를 직접 실행하면 `/bin/sh`을 찾을 수 없다는 에러가 발생한다. 가이드대로 gunicorn을 사용해 실행해야 한다. (다소 번거로움) 그래도 안되는 경우 CMD를 풀어서 쓰라[^fn-workaround]는 workaround가 있다. 별다른 설정 없이 잘 실행되는 Go와 달리 Python/Flask는 다소 번거로운 작업이 필요하다. 특히 내가 만든 OAuth2 앱은 entrypoint가 main이 아니다 보니 실행하기가 번거롭고 아직 실행에 성공하지 못했다.

[^fn-workaround]: <https://stackoverflow.com/a/64084917>