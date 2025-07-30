---
layout: post
title: 엘라스틱서치 7을 위한 은전한닢 형태소 분석기
tags: ["Information Retrieval"]
last_modified_at: 2022/01/15 00:00:00
---

<div class="message">
한국어 형태소 분석기 은전한닢을 엘라스틱서치 7 이상에서도 사용할 수 있도록 패치를 진행하고 내용을 정리하였습니다.
</div>

<small>
*2022년 1월 15일 초안 작성*  
</small>

<!-- TOC -->

- [개요](#개요)
- [패치](#패치)
  - [Plugin Structure](#plugin-structure)
  - [Super Constructor Parameters](#super-constructor-parameters)
  - [ESLoggerFactory to Loggers](#esloggerfactory-to-loggers)
- [문서 및 설치](#문서-및-설치)
- [기타](#기타)
- [정리](#정리)
- [참고](#참고)

<!-- /TOC -->

# 개요

한국어 형태소 분석기 은전한닢이 2018년에 엘라스틱에서 [공식 한국어 분석 플러그인 노리를 공개](https://www.elastic.co/kr/blog/nori-the-official-elasticsearch-plugin-for-korean-language-analysis)한 이후 아쉽게도 더 이상 업데이트가 되지 않고 있습니다. 공식 홈페이지에는 6 버전까지만 설치 파일이 제공되었고, 7 이상에서는 설치가 되지 않는 상태였죠. 하지만 [Amazon OpenSearch Service는 은전한닢을 공식 분석기로 지원](https://aws.amazon.com/ko/blogs/korea/amazon-elasticsearch-service-now-supports-korean-language-plugin/)하고 있고, 형태소 분석 결과 또한 은전한닢이 검색엔진에 훨씬 더 적합한 형태로 출력됩니다. 따라서 은전한닢을 7 버전에서도 계속해서 사용할 수 있도록 패치를 진행해보았습니다.

- [엘라스틱서치 7을 위한 은전한닢 형태소 분석기 패치 바로가기](https://github.com/likejazz/seunjeon-elasticsearch-7)

# 패치
## Plugin Structure

```
ERROR: This plugin was built with an older plugin structure. Contact the plugin author to remove the intermediate "elasticsearch" directory within the plugin zip.
```

가장 먼저 마주한 에러는 older plugin structure로 되어 있다는 에러였습니다. 이전과 달리 7 이상에서는 더 이상 elasticsearch라는 디렉토리를 구성할 필요가 없습니다. 말 그대로 intermediate "elasticsearch" directory를 제거하고 다시 설치를 진행해봅니다.

## Super Constructor Parameters

구조를 맞추면 설치가 성공합니다. 이제 Elasticsearch가 제대로 시작 되는지 살펴보겠습니다.
```
fatal error in thread [elasticsearch[5001c0b3ce60][masterService#updateTask][T#1]], exiting
java.lang.NoSuchMethodError: 'void org.elasticsearch.index.analysis.AbstractTokenizerFactory.<init>(org.elasticsearch.index.IndexSettings, java.lang.String, org.elasticsearch.common.settings.Settings)'
	at org.bitbucket.eunjeon.seunjeon.elasticsearch.index.analysis.SeunjeonTokenizerFactory.<init>(SeunjeonTokenizerFactory.java:23)
```
메소드가 없다는 오류가 발생합니다. `SeunjeonTokenizerFactory.java:23`에서 호출하면서 발생한 에러이므로 해당 위치를 확인해보겠습니다.

```java
public class SeunjeonTokenizerFactory extends AbstractTokenizerFactory {
    private TokenizerOptions options;

    public SeunjeonTokenizerFactory(IndexSettings indexSettings,
                                    Environment env,
                                    String name,
                                    Settings settings) {
        super(indexSettings, name, settings);
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

        this.options = TokenizerOptions.create(name).
                setUserDictPath(getFullPath(env, settings.get("user_dict_path", null))).
                setUserWords(settings.getAsList("user_words", Collections.emptyList())).
                ...
```
Super constructor에서 에러가 발생합니다. 왜 그런지 확인해보기 위해 SeunjeonTokenizerFactory 클래스가 상속받은 Elasticsearch의 `AbstractTokenizerFactory.java`를 살펴봅니다.

```java
public AbstractTokenizerFactory(IndexSettings indexSettings, Settings settings, String name) {
```

`AbstractTokenizerFactory`는 파라미터 순서가 `IndexSettings, Settings, String` 순입니다. 마지막 2개가 순서가 바뀌어 있는데 왜 그런지 Elasticsearch의 히스토리를 뒤져봅니다. 그 결과, `String name`을 더 이상 쓰지 않겠다고 2017년 5월에 이름을 `name`에서 `ignored`로 변경[^fn-201705]했고, 그러다 2018년 7월에 완전히 삭제[^fn-201807]했습니다. `TODO` 상태를 1년 2개월 동안이나 유지하다가 결정을 내렸는데, 재밌게도 바로 1년 후인 2019년 7월에 `name`을 다시 되살리는 결정[^fn-201907]을 내립니다.

제거할때는 파라미터 중간에서 없앴지만 다시 되살릴때는 뒤에다 붙여 버립니다. 그래서 2년여 사이에 파라미터가 `IndexSettings, String, Settings` 이었다가 `IndexSettings, Settings, String`로 변경됩니다. 플러그인도 지난 2년여의 수정 사항을 반영하여 패치하였습니다.

```
// AS-IS
super(indexSettings, name, settings);
// TO-BE
super(indexSettings, settings, name);
```

[^fn-201705]: [Drop name from TokenizerFactory (#24869)](https://github.com/elastic/elasticsearch/commit/6d9ce957d4aa2916ba2c3f467bda70fdf6325647)
[^fn-201807]: [Handle TokenizerFactory TODOs (#32063)](https://github.com/elastic/elasticsearch/commit/ed3b44fb4cd6f957facf6d0f38daa08eeecd52ea#diff-c81258d347863d0695a7c53fc27ba2a277c75635d8d96a3a8cfaa27aa24cf332)
[^fn-201907]: [Add name() method to TokenizerFactory (#43909)](https://github.com/elastic/elasticsearch/commit/60b460d38a1d3a88e208fb7f1285640804866450#diff-c81258d347863d0695a7c53fc27ba2a277c75635d8d96a3a8cfaa27aa24cf332)

## ESLoggerFactory to Loggers

```
[error] /Users/likejazz/workspace/github.com/likejazz/seunjeon-elasticsearch-7/elasticsearch/src/main/scala/org/bitbucket/eunjeon/seunjeon/elasticsearch/TokenizerHelper.scala:10:8: object ESLoggerFactory is not a member of package org.elasticsearch.common.logging
[error] import org.elasticsearch.common.logging.ESLoggerFactory
[error]        ^
```
다시 sbt 빌드를 진행해보면 `ESLoggerFactory`가 `org.elasticsearch.common.logging`의 멤버가 아니라는 에러가 납니다. 이게 무슨 얘기인지 다시 Elasticsearch의 소스를 추적해봅니다.

이전 버전에서 존재하던 `ESLoggerFactory`를 현재 master 브랜치에서 더 이상 찾을 수가 없습니다. 그렇다면 지금은 어떤 클래스를 이용해 로그를 남기고 있을까요. 탐색해보니 `Loggers`라는 클래스를 이용하고 있었습니다. 히스토리를 살펴보니 2018년 11월에 `ESLoggerFactory`를 `Loggers`로 머지한다는 커밋[^fn-201811]을 찾아낼 수 있었습니다.

[^fn-201811]: [Logger: Merge ESLoggerFactory into Loggers (#35146)](https://github.com/elastic/elasticsearch/commit/348c28d1d1b38e49900caf9937221313cb87d57b#diff-496ddd720695319f62bf66aa7766d546f169e2a7b0e9b43b1b41d6ea230ccb73)

플러그인의 자바와 스칼라 코드 각각 `getLogger()` 메소드를 실행할때 `ESLoggerFactory` 대신 변경된 `Loggers` 클래스를 이용하도록 수정했습니다. 그리고 `getLogger()`는 기존에 `String`만 받던 메소드는 사라지고 반드시 클래스를 함께 입력받도록 수정됐기 때문에 플러그인에서 호출시 `getClass()`로 클래스를 함께 보내도록 했습니다.

```
// AS-IS
import org.elasticsearch.common.logging.ESLoggerFactory
val logger: Logger = ESLoggerFactory.getLogger(TokenizerHelper.getClass.getName)

// TO-BE
import org.elasticsearch.common.logging.Loggers
val logger: Logger = Loggers.getLogger(getClass(), TokenizerHelper.getClass.getName)
```

이제 빌드가 정상적으로 진행되고 로그가 정상적으로 출력됩니다.

# 문서 및 설치
원래 은전한닢은 C++로 개발되어 설치 과정이 무척 복잡했습니다. 또한 Memory Leak의 위험성을 내포하고 있었으나 이후 Scala 버전인 seunjeon으로 재작성되면서 GC의 도움을 받을 수 있게 됐고 설치 과정도 단순해졌습니다. 그러나 원본 repo의 문서에는 패키지 빌드 과정과 엘라스틱 설치 과정이 복잡하게 섞여 있었고, 문서의 내용 또한 옛날 사전을 빌드하는 상태에서 더 이상 업데이트가 되어 있지 않아 혼란을 줄 수 있었습니다. 따라서 기존에 보기 힘들었던 문서를 모두 정리하고, 사전을 포함한 최신 사항을 모두 반영했습니다.

또한 설치 스크립트를 제공하는 방식에서, 보다 쉽게 설치할 수 있는 URL 직접 설치 방식으로 개선하였습니다. 이 경우 마이너 버전업이 될 때도 매번 패키징을 해야 하는 불편함이 있으나 쉬운 설치가 훨씬 더 중요한 부분이라 생각했고, 패키징 및 테스트를 Docker를 이용해 최대한 자동화 하는 방식으로 번거로움을 덜었습니다. 앞으로 Elasticsearch 버전 업그레이드가 있을 때 마다 계속해서 패키징을 제공할 생각입니다.

# 기타
- Elasticsearch의 Heap을 512m로 설정해 사용하고 있었는데, seunjeon을 설치하고 구동하면 Out of memory로 죽는 경우가 발생했습니다. 1g에서는 정상 동작했고, 조금 더 여유있게 1200m로 설정해서 구동했습니다.
    - Elastic Cloud에서도 Gold 구독부터는 플러그인 설치가 가능합니다. 이때 최소 사양 1GB RAM에서 seunjeon 플러그인을 설치하면 out of memory로 구동에 실패하고 Pending 상태에서 복구 되지 않습니다. 다음과 같은 안내 메일을 9시간이 지난 새벽 2시 반(한국 시각)에 받았습니다.
> A node in your cluster XXX (XXX) ran out of memory at 17:30 on January 20, 2022. As a precaution, we automatically restarted the node instance-0000000001.
    - Force restart로 해결은 가능하나, 시스템에 직접 접근하거나 로그를 볼 수 없는 클라우드의 특성상 원인을 유추하지 못하면 해결이 어렵습니다. 또한 처음에 플러그인 설치 옵션이 비활성화 되어있어 Support center의 도움을 받아야 했습니다.
- 초기에 X-Pack에서 Auth 오류가 발생하는 문제가 있었으나 이 문제는 Enterprise Search와 함께 구동시 너무 빨리 호출이 진행되다 보니 seunjeon으로 인해 초기 구동이 늦어지면서 발생하는 오류로 보입니다. Elasticsearch가 정상 구동한 이후에는 더 이상 오류가 발생하지 않습니다. 또한 Enterprise Search와 함께 구동하지 않을때는 발생하지 않습니다.
- [OpenSearch 용도로 패치](https://bitbucket.org/soosinha/seunjeon-opensearch/)한 버전이 있습니다. OpenSearch는 7.10을 fork한 버전이라 수정 사항이 동일합니다. 저자는 upstream 반영을 위해 문의하였으나 아쉽게도 응답이 없었습니다.
- Scala 개발 환경을 구성하고 sbt 빌드하는 과정이 사실상 가장 힘든 부분이었습니다. 그러나 JDK 1.8 의존성 때문에 반드시 버전을 낮춰야 하는 부분 외에는 특별히 다른 문제는 없었습니다.

# 정리
무엇보다 이런 훌륭한 형태소 분석기를 만드신 은전한닢의 원저자 유영호님, 이용운님께 다시 한 번 감사드립니다. upstream이 활동을 재개하면 원저자분들과 상의하여 이후 forked repo의 존립 여부를 결정하도록 하겠습니다. Elasticsearch 7 용도로 리패키징한 플러그인은 [likejazz/seunjeon-elasticsearch-7](https://github.com/likejazz/seunjeon-elasticsearch-7)에 있습니다. Elasticsearch 7 이상에서 은전한닢을 사용하고자 하는 많은 분들께 도움이 되길 바랍니다.

# 참고
- [Official seunjeon Bitbucket](https://bitbucket.org/eunjeon/seunjeon/)
- [은전한닢 프로젝트](http://eunjeon.blogspot.com/)
- [seunjeon for OpenSearch](https://bitbucket.org/soosinha/seunjeon-opensearch/)
