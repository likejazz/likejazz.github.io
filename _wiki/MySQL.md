---
layout: wiki 
title: MySQL
tags:  ["Software Engineering"]
last_modified_at: 2025/06/08 19:30:21
---

<!-- TOC -->

- [설치](#설치)
  - [한글 문제](#한글-문제)
  - [MariaDB 최신 버전 설치](#mariadb-최신-버전-설치)
  - [로깅](#로깅)
  - [테이블 생성](#테이블-생성)
  - [사용자 및 권한 추가](#사용자-및-권한-추가)
- [운영](#운영)
  - [로그 전환](#로그-전환)
  - [재시작](#재시작)
- [Lost connection to MariaDB when adding full text index](#lost-connection-to-mariadb-when-adding-full-text-index)

<!-- /TOC -->

# 설치
## 한글 문제
DB 생성시 `CREATE DATABASE mydatabase CHARACTER SET utf8 COLLATE utf8_general_ci;`로 하고, 파이썬에서 접속시 `charset=utf8`를 명시한다. 테이블에 삽입시 `.encode()`를 할 필요는 없다.

## MariaDB 최신 버전 설치
기존 yum에 등록된 MariaDB의 버전이 너무 낮아서 JSON 컬럼등 최신 기능을 이용하기 위해 버전업 진행  
`/etc/yum.repos.d/MariaDB.repo`에 repository 등록
```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
proxy=http://proxy.test.io:3128
```

`sudo yum install MariaDB-server`로 설치

## 로깅  
```
$ sudo mkdir -p /var/log/mysql/
$ touch /var/log/mysql/mysql.log
$ sudo chown mysql.mysql /var/log/mysql/mysql.log
```

```
[mysqld]
general_log_file = /var/log/mysql/mysql.log
general_log = 1
```
http://stackoverflow.com/a/650265/3513266  
mysql.log의 owner를 mysql로 하여 쓰기 권한을 부여한다.
`SELECT`를 포함한 DB에 유입되는 모든 쿼리를 모니터링 할 수 있어 에러 디버깅에 매우 유용하다. 그러나 성능이 저하되므로 디버깅 용도로만 사용.

## 테이블 생성
```
CREATE DATABASE table CHARACTER SET utf8 COLLATE utf8_general_ci;
```

## 사용자 및 권한 추가
```
GRANT ALL PRIVILEGES ON table.* To 'user'@'%' IDENTIFIED BY 'user';
GRANT ALL PRIVILEGES ON table.* To 'user'@'user.test.io' IDENTIFIED BY 'user';
GRANT ALL PRIVILEGES ON table.* To 'user'@'localhost' IDENTIFIED BY 'user';
FLUSH PRIVILEGES;
```

# 운영
## 로그 전환
디버그 용도로 모든 로그를 `/var/log/mysql/mysql.log`에 쌓고 있는데 모든 SQL이 기록되므로 주기적으로 FLUSH하여 용량을 확보해야 한다.

```
$ cd /var/log
$ sudo mv mysql.log mysql.log.1
$ sudo touch mysql.log
$ sudo chown mysql.mysql mysql.log
$ mysql -uroot
MariaDB [(none)]> FLUSH LOGS;
```

만약 권한이나 로그 파일로 FLUSH를 실패했을 경우 데몬을 재시작 해야 한다. 
```
sudo service mariadb restart
```

MariaDB의 실행 로그는 아래에 별도로 남는다.
```
$ sudo tail -f /var/log/mariadb/mariadb.log
```

## 재시작  
```
$ sudo systemctl restart mariadb
```

# Lost connection to MariaDB when adding full text index
MySQL에서 FULLTEXT 인덱스를 설정하는데 자꾸만 Lost connection 에러 발생, MariaDB 이며 [비슷한 질문](http://stackoverflow.com/questions/33993616/error-2013-lost-connection-to-mariadb-when-adding-full-text-index)이 SO에 있으나 답변이 없다.
