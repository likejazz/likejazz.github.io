---
layout: wiki 
title: Firebase Cloud Messaging
last-modified: 2021/03/13 01:02:59
---

<!-- TOC -->

- [개요](#개요)
- [방법](#방법)
    - [Access Token 발급 받는 법](#access-token-발급-받는-법)

<!-- /TOC -->

# 개요
앱에 Firebase SDK를 설치하면 Firebase 서버를 이용해 푸쉬를 제공한다. 저녁 시간에 메시지가 도달하지 않는 경험을 했으나 한밤중에는 누락 없이 잘 도착했다. 단체 발송을 기존 token 단위 배치 발송에서 topic을 이용한 발송으로 변경하여 실험한다.

# 방법
```bash
ACCESS_TOKEN="ya29...8yNK8UlESsi4ye_i"
NOW=$(date +'%Y-%m-%d %H:%M:%S')

$ curl -i -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
        -H "Content-Type: application/json" \
        -d '{
  "message": {
    "notification": {
      "body": "푸쉬 바디",
      "title": "푸쉬 제목 '"$NOW"'"
    },
    "condition": "'"'"'weather'"'"' in topics || '"'"'sports'"'"' in topics"
  }
}' https://fcm.googleapis.com/v1/projects/PROJECT_NAME/messages:send
```

## Access Token 발급 받는 법
기존 방식은 Server Key로 가능하나 DEPRECATED 되면서 새로이 OAuth2가 적용됐다. SDK로 동작하기 때문에 REST로는 어렵지만 [구글 플레이그라운드](https://developers.google.com/oauthplayground/)에서 `email, https://www.googleapis.com/auth/firebase.messaging`을 입력하여 발급 받을 수 있다.[^fn-curl]

[^fn-curl]: <https://blog.mestwin.net/send-your-test-fcm-push-notification-quickly-with-curl/>