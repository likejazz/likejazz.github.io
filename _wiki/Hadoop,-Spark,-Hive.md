---
layout: wiki 
title: Hadoop, Spark, Hive
tags: ["Data Science & Engineering"]
last_modified_at: 2022/04/25 02:49:54
---

<!-- TOC -->

- [설치](#설치)
  - [Hadoop Client](#hadoop-client)
  - [Standalone 소스 설치](#standalone-소스-설치)
  - [Apache Spark](#apache-spark)
- [성능](#성능)
- [설정](#설정)
  - [JDK 버전](#jdk-버전)
  - [설정](#설정-1)
  - [PyCharm 연동](#pycharm-연동)
  - [실행](#실행)
- [Hive](#hive)
  - [Hive 조회 결과 GCS 업로드](#hive-조회-결과-gcs-업로드)

<!-- /TOC -->

# 설치
## Hadoop Client
~~클라우데라에 상세한 [설치 문서](https://www.cloudera.com/documentation/enterprise/latest/topics/cdh_ig_cdh5_install.html)가 있다.~~ 링크 깨짐. CentOS에서 yum으로 설치하고 싶었는데, 클라우데라에서 제공하는 repo가 있다. repo 등록 후 클라이언트 설치는 아래 명령으로 간단히 가능하다.

```	
sudo yum clean all; sudo yum install hadoop-client
```

노드 설정은 다른 클라이언트 노드의 `/etc/hadoop/conf`를 그대로 복사해서 설정했다. 원래 클라우데라 매니저에서 배포하는 것으로 보이는데 매니저의 위치나 배포 방법을 찾지 못했다. 사용자 디렉토리 설정을 따라가므로 `export HADOOP_USER_NAME=hanadmin`를 지정했다. 클라이언트는 별도로 데몬등을 구동할 필요는 없다.

native hadoop jar 오류가 발생했는데, 맥에서는 별도로 패치해서 바이너리 빌드가 필요했다. 에러가 보이지 않도록 log4j.properties에서 `log4j.logger.org.apache.hadoop.util.NativeCodeLoader=ERROR` 설정 했다.

## Standalone 소스 설치
~~[이 문서](https://www.cloudera.com/documentation/enterprise/5-5-x/topics/cdh_ig_yumrepo_local_create.html)에 있는 repo를 설치하고 `sudo yum install hadoop-conf-pseudo`로 설치했다.~~ 사용자 su 변경 문제로 설치에 실패했다.

Spark를 소스 설치하고, Hadoop 또한 [예전 문서](https://archive.cloudera.com/cdh5/cdh/5/hadoop/hadoop-project-dist/hadoop-common/SingleCluster.html#Standalone_Operation)를 참고해 소스 설치했다. 옛날 문서지만 오히려 더 직관적이어서 수월했다.

사내 솔루션인 sparser 관련해 다소 시행 착오가 있었으나 예전 서버에서 package 파일을 모두 가져왔고, HADOOP_CONF 관련한 설정을 수정(symlinks 처리)하여 모두 동작하도록 했다.

```
# 시작
./hadoop/sbin/start-dfs.sh
./hadoop/sbin/start-yarn.sh
./spark/sbin/start-all.sh (Spark)

# 중단
./spark/sbin/stop-all.sh
./hadoop/sbin/stop-yarn.sh
./hadoop/sbin/stop-dfs.sh
```
hdfs를 사용하지 않을 경우 Spark만 시작/종료 하면 된다.

## Apache Spark
맥북에서 Spark가 hdfs가 아닌 로컬에 저장되었는데, standalone 모드로 동작한다. 
```
export HADOOP_CONF_DIR="/usr/local/opt/hadoop/libexec/etc/hadoop"
``` 
설정을 해주니 hdfs를 디폴트로 인식한다.

brew에서는 Spark가 2.2가 최신인데, 처음에 Kerberos 인증 오류가 이 때문으로 의심되어 1.6 버전을 brew tap해서 설치했다. 그러나 이 문제가 아닌 JDK 버전 문제였고 다시 맥에서는 2.2 최신 버전으로 설치했다. 

# 성능
Spark Job이 맥에서는 서버에 비해 늦다. dead nodes가 없는데도 맥과 Jupyter에서는 Failed to connect 오류가 많이 발생한다.

매우 단순한 스크립트로 성능을 측정한다.
```
from pyspark import SparkContext
import json

sc = SparkContext()
lines = sc.textFile("kaon/docquery.json")
query = lines.map(lambda x: json.loads(x)["Query"])

print (query.count())
```
결과는 `611214`가 출력되어야 한다.

| 서버 | 환경 | 사양 | 성능 | 비고 |
| --- | --- | --- | --- | --- |
| s3-jupyter(OpenStack) | Jupyter | Intel Xeon E312xx (Sandy Bridge) x 4, 38G 램 | 18s | |
| MacBook Pro | pyspark | |  23s | 노드 접속 오류 많음 |
| s3-jupyter(OpenStack) | spark-submit | | 25s | took 13.133146 s (hdfs 로그 기준, 노드 접속 오류 많음) |
| search-s2-rcmadmin3 | spark-submit | Intel(R) Xeon(R) CPU E5-2620 v3 @ 2.40GHz x 24, 96G | 21s | took 10.803911 s |

`spark-submit`은 아래와 같이 실행했다.
```
spark-submit --master yarn-client \
             --name "perf-estimates" \
             --num-executors 50 \
             --executor-memory "8g" \
             --driver-memory "8g" \
             perf-est.py
```

s3-jupyter(Krane)에서 Jupyter는 python3를 기반으로 했으나, spark-submit 시에는 `export PYSPARK_PYTHON=python`로 2 버전대로 지정했다.

standalone mode와 yarn mode가 왜 비슷한 성능을 보이는지는 *추가로 확인이 필요*하다.
 
# 설정
## JDK 버전
`hadoop fs -ls`시 계속 Kerberos 인증 오류가 발생했는데, JDK 9의 문제였다. `brew cask uninstall java`로 기존에 설치했던 9 버전을 제거하고 OS에 기본으로 설치되어 있던 8 버전을 사용하니 문제가 없었다. [cask 이전 버전 설치](https://www.jverdeyen.be/mac/downgrade-brew-cask-application/)를 시도해보았으니 예전 버전의 바이너리가 Oracle에서 이미 삭제되어 있어서 설치가 되지 않았다. 아래 문서대로 `brew tap caskroom/versions`를 하니 설치가 잘 된다. [Apache Spark does not work with Java 9 yet. Install Java 8 back to get it running.](https://gist.github.com/lukewang1024/659ec27847169086dde8677e25156573
)

## 설정
`.zshrc`에 설정한 내용은 아래와 같다.
```
# Hadoop Configurations
export HADOOP_USER_NAME=hanadmin
export HADOOP_CONF_DIR="/usr/local/opt/hadoop/libexec/etc/hadoop"

export PYSPARK_PYTHON=python3
export SPARK_VERSION=`ls /usr/local/Cellar/apache-spark/ | sort | tail -1`
export SPARK_HOME="/usr/local/Cellar/apache-spark/$SPARK_VERSION/libexec"
export PYTHONPATH=$SPARK_HOME/python/:$PYTHONPATH
export PYTHONPATH=$SPARK_HOME/python/lib/py4j-0.10.4-src.zip
```

## PyCharm 연동
여러 문서를 참조하여 시도해보았으나 `ImportError: cannot import name 'SparkContext'`에서 벗어나는 방법을 아직 찾지 못했다. 위에서 언급한 로컬 맥북에서 실행시 성능 저하 문제와 함께 아직 로컬에서 제대로 활용할 수 있는 방법을 찾지 못했다. 현재는 원격 서버에 Jupyter를 설치하고 이를 통해 사용하고 있다.

## 실행
빠른 MR 테스트를 위해 hdfs를 사용하지 않으려 했으나 `file:/`을 아무리 지정해도 매 번 `hdfs:/`에서 찾아 실패했다. 그러나 `pyspark`에서는 로컬 파일에 잘 액세스 하여 IPython 처럼 interactive하게 디버깅이 가능하다.

`HADOOP_CONF_DIR`를 주석 처리하니 `file:/`에 액세스 가능하다. 그러나 여전히 정상적인 경로를 찾지 못한다.

# Hive
Hive vs. Spark  
가장 중요한건 속도차이다. Spark는 거의 실시간이다. 회사 Hive는 쿼리 하나에 몇 분씩 돌 정도로 엄청나게 느렸으나 반면 사용량이 적은 시스템은 Hive도 나쁘지 않은 속도를 보였다.

## Hive 조회 결과 GCS 업로드
Hive에서 쿼리 조회 결과를 hdfs에 저장, 이후 로컬에 가져와서 맥북을 경유하여 GCS에 업로드했다.
```console
# 원격 서버
$ ssh xxx@10.12.109.xxx

# 분석 결과 CSV export
$ hive -e "INSERT OVERWRITE DIRECTORY 'output'
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ',' 
select * from ignxxx.aas_6 limit 10000000"
# 위에 1천만 건 5.3G
 
# CSV 결과 조회
$ hdfs dfs -ls output
 
# 로컬로 가져오기
$ hdfs dfs -get output

# Append header(Linux only)
$ sed -i '1i HEADER' output.csv  # 헤더 파일은 Hive 웹에서 export로 미리 확보하고 있어야 함

# 맥북에서 streaming으로 GCS 업로드(5.3G 약 5분 소요, 로컬 디스크 공간 필요 없음)
$ scp xxx@10.12.109.xxx:output.csv /dev/stdout | gsutil cp - gs://stark-xxx/output.csv
```

원래 회사 유선망으로 100MB/s가 나오는데, 50MB/s 정도였고 remote server에서 가져오는 동안 멈춰있는 것으로 보인다.