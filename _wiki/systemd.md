---
layout: wiki 
title: systemd
tags: ["Software Engineering"]
last_modified_at: 2024/06/29 01:20:28
---

<!-- TOC -->

- [기본 명령](#기본-명령)
  - [crontab](#crontab)

<!-- /TOC -->

systemd는 Linux init system으로 timer도 제공하며, Next Generation Cron이라 일컫는다.

- systems는 initd를 대체하는 init 시스템이었지만 최근에는 로깅, 네트워크 구성, 네트워크 시간 동기화와 같은 기능을 포함하는 강력한 관리자다. (모던 리눅스 교과서)
- systemd와 상호 작용해 서비스를 관리하기 위해 사용하는 도구가 systemctl 이다.

# 기본 명령
타이머 조회:
```
$ systemctl list-timers
NEXT                        LEFT           LAST                        PASSED       UNIT                         ACTIVATES
Fri 2022-07-29 12:00:00 KST 8min left      Fri 2022-07-29 11:41:15 KST 9min ago     logrotate.timer              logrotate.service
Fri 2022-07-29 15:05:44 KST 3h 14min left  Fri 2022-07-29 03:46:38 KST 8h ago       ua-messaging.timer           ua-messaging.service
Fri 2022-07-29 17:05:27 KST 5h 14min left  Fri 2022-07-29 09:53:08 KST 1h 58min ago motd-news.timer              motd-news.service
...
```

타이머와 서비스는 우분투 기준 `/lib/systemd/system`에 있다. 예를 들어 `logrotate.timer`와 `logrotate.service`의 경우.

logrotate를 daily → hourly 변경할 때:
```
$ cat logrotate.timer
...
[Timer]
# run on the hour of every hour of every day
OnCalendar=*-*-* *:00:00
# OnCalendar=daily
```
설정 변경 후에는 다음과 같이 적용:
```
$ sudo systemctl daemon-reload
```

최종 실행 상태 확인
```
$ cat /var/lib/logrotate/status 
```
## crontab
하지만 service가 아닌 간단한 script는 여전히 crontab이 유용하다.
```
$ * * * * * TZ=Asia/Seoul /home/sangpark/check-ollama.sh >> /home/sangpark/check-ollama.logs 2>&1
```
`2>&1`는 stderr도 stdout으로 redirect하라는 의미