---
layout: wiki 
title: Elasticsearch 운영
tags: ["Information Retrieval"]
last_modified_at: 2022/02/26 14:17:31
---

<!-- TOC -->

- [매핑](#매핑)
- [검색](#검색)
  - [색인](#색인)
  - [분석](#분석)
  - [검색](#검색-1)
  - [랭킹](#랭킹)
    - [Painless](#painless)
  - [한글](#한글)
- [기타](#기타)
  - [설정](#설정)
  - [자소 분리](#자소-분리)

<!-- /TOC -->

# 매핑
es의 스키마리스 기능(동적 맵핑)은 필드가 잘못 지정될 수 있으므로 가급적 지양. 예전 버전은 색인시에도 boost가 가능했으나 혼동을 일으킬 수 있어 7.0부터 색인 시 boost 기능이 제거됐다.

# 검색
## 색인
인덱스는 Stack Management > Index Management에서 좀 더 편하게 확인 가능하다. 인덱스명을 클릭하고 Mappings에서 스키마 조회 가능.

인덱스 생성은 더 이상 type 지정이 필요 없고 `_doc` 기본 타입 또한 지정할 필요 없다. OpenSearch에는 seunjeon 플러그인이 미리 설치되어 있다. 다음은 형태소 분석, 키워드, ngram 필드 구성과 likes 품질 점수, geo_point로 구성된 인덱스:

```
PUT my-index-000001
{
  "settings": {
    "index": {
      "max_ngram_diff": 2,
      "analysis": {
        "char_filter": {
          "whitespace_remove": {
            "type": "pattern_replace",
            "pattern": " ",
            "replacement": ""
          }
        },
        "filter": {
          "synonym": {
            "type": "synonym",
            "synonyms": [
              "게하, 게스트하우스"
            ]
          }
        },
        "tokenizer": {
          "seunjeon_tokenizer": {
            "type": "seunjeon_tokenizer",
            "index_eojeol": true,
            "decompound": true,
            "pos_tagging": false,
            "user_words": [
              "스타벅스",
              "게하"
            ],
            "index_poses": ["UNK","EP","I","M","N","SL","SH","SN","V","VCP","XP","XS","XR"]
          },
          "ngram": {
            "type": "ngram",
            "min_gram": 1,
            "max_gram": 3,
            "token_chars": [
              "letter",
              "digit",
              "punctuation",
              "symbol"
            ]
          }
        },
        "analyzer": {
          "seunjeon": {
            "tokenizer": "seunjeon_tokenizer"
          },
          "seunjeon_search": {
            "tokenizer": "seunjeon_tokenizer",
            "filter": ["synonym"]
          },
          "ngram": {
            "char_filter": ["whitespace_remove"],
            "tokenizer": "ngram",
            "filter": ["lowercase"]
          },
          "keyword": {
            "char_filter": ["whitespace_remove"],
            "tokenizer": "keyword",
            "filter": ["lowercase"]
          }
        }
      },
      "similarity": {
        "less_length_norm_BM25": {
          "type": "BM25",
          "b": 0.2
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "seunjeon",
        "search_analyzer": "seunjeon_search",
        "similarity": "less_length_norm_BM25",
        "fields": {
          "token_count": {
            "type": "token_count",
            "analyzer": "seunjeon"
          },
          "keyword": {
            "type": "text",
            "analyzer": "keyword"
          },
          "ngram": {
            "type": "text",
            "analyzer": "ngram"
          },
          "ngram_token_count": {
            "type": "token_count",
            "analyzer": "ngram"
          }
        }
      },
      "likes": {
        "type": "integer"
      },
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```

토크나이저 옵션이 틀려도 아무런 경고가 나오지 않기 때문에 주의가 필요하다. [지원 옵션과 품사 참고](https://github.com/likejazz/seunjeon-elasticsearch-7).

실험 데이터 삽입:
```
POST my-index-000001/_bulk
{"index":{"_id":"1"}}
{"title":"이마트죽전점","likes":9,"location":{"lat":37.32523,"lon":127.11008}}
{"index":{"_id":"2"}}
{"title":"죽전이마트","likes":3,"location":{"lat":37.32732,"lon":127.10968}}
{"index":{"_id":"3"}}
{"title":"이마트역삼점","likes":12,"location":{"lat":37.49916,"lon":127.04842}}
```

Elasticsearch 7 이상에서 설치가 가능하도록 패치한 내용은 [likejazz/seunjeon-elasticsearch-7](https://github.com/likejazz/seunjeon-elasticsearch-7)으로 공개했다.

## 분석
설치된 플러그인 조회:
```
GET _cat/plugins?v
```
Elasticsearch는 아무것도 설치되어 있지 않으며 Amazon OpenSearch에는 여러 플러그인이 미리 설치되어 있다.

anaylzer 순서는 Character filter → Tokenizer → Token filters 순이다. lowercase는 character filter가 없지만 어차피 대소문자 여부가 tokenizing에 영향을 주지 않는다.

맵핑 없이 _analyze에서 tokenizer와 filter를 정의해 직접 테스트:
```
GET _analyze
{
  "text": "삼진주유소",
  "tokenizer": {
    "type": "jaso_tokenizer",
    "mistype": true,
    "chosung": false  
  },
  "filter": {
    "type": "edge_ngram",
    "min_gram": 1,
    "max_gram": 10
  }
}
```

검색 쿼리 분석:
```
# Search
GET my-index-000001/_validate/query?explain
{
  "query": {
    "script_score": {
      "query": {
        "bool": {
          "should": [
            {
              "multi_match": {
                "query": "이마트",
                "fields": [
                  "title", "title.keyword"
                ],
                "type": "most_fields"
              }
            },
            {
              "multi_match": {
                "query": "이마트",
                "fields": [
                  "title", "title.ngram"
                ],
                "type": "cross_fields"
              }
            }
          ]
        }
      },
      "script": {
        "lang": "painless",
        "source": "double distanceScore = decayGeoGauss(params.origin, params.scale, params.offset, params.decay, doc['location'].value); if (explanation != null) { explanation.set('_score(' + Math.round(_score * 10000) / 10000.0 + ') + script_score:'); } return _score + script_score;",
        "params": {
          "origin": "37.49930, 127.04818",
          "scale": "10km",
          "offset": "0km",
          "decay": 0.5
        }
      }
    }
  }
}
```

## 검색
```
# 전체 필드 검색
GET my-index-000001/_search
{
  "query": {
    "query_string": {
      "query": "이마트"
    }
  }
}

# 쿼리 분석 없이, 그대로 구문 매칭일때
# 그러나 text 타입은 이미 `[mary, bailey]` 2 tokens로 분석되어 색인되었기 때문에 term으로는 매칭되지 않는다. keyword 타입만 가능하다.
GET kibana_sample_data_ecommerce/_search
{
  "query": {
    "term": {
      "customer_full_name": "mary bailey"
    }
  }
}
```

`explain: true`일때 description에 수식을 보여준다. 예전에는 알아차리기가 어려워 점수를 잘못 계산하는 실수가 많았는데 랭킹 수식을 이렇게 나열해주니 바로 계산이 가능하고 실수를 파악할 수 있어 매우 유용하다.

query는 점수 계산, filter는 일치 여부만 포함

levenshtein distance 기반으로 검색하는 fuzziness 옵션 존재. 8.0부터는 벡터 kNN 결과도 추가된다. OpenSearch는 이미 지원. ngram을 사용하면 거의 fuzzy한 효과를 낸다.

검색 프로파일링 결과:  
<img width="80%" src="https://user-images.githubusercontent.com/1250095/155664355-952112a8-59bb-409b-9e55-4309a8a10289.png">


## 랭킹
[`function_score`를 이용](https://www.elastic.co/blog/found-function-scoring)해서 점수 보완. gauss, linear, exp 3가지 방식 지원.

<img src="https://user-images.githubusercontent.com/1250095/148712565-0d4e174e-c3ef-454c-9b3d-09566a7fb1a9.jpg" width="50%">

```
# title.keyword는 기존 boost(default 2.2)에 3을 곱한 6.6이 된다.
GET my-index-000001/_search
{
  "fields": [
    "title.token_count",
    "title.ngram_token_count"
  ],
  "query": {
    "function_score": {
      "query": {
        "multi_match": {
          "query": "죽전",
          "fields": [
            "title", 
            "title.keyword^3", 
            "title.ngram"
          ],
          "type": "most_fields"
        }
      },
      "functions": [
        {
          "gauss": {
            "location": {
              "origin": "37.49930, 127.04818",
              "scale": "10km",
              "offset": "0km"
            }
          },
          "weight": 10
        },
        {
          "field_value_factor": {
            "field": "likes",
            "factor": 1
          }
        }
      ],
      "score_mode": "sum", 
      "boost_mode": "sum"
    }
  },
  "highlight": {
    "fields": {
      "title*": {}
    }
  },
  "explain": true
}
```
점수는 10km 지점에서 0.5 decay(default)가 된다. 죽전과 역삼은 20km 거리이므로, 거의 0점이며 부스팅(`weight: 10`)을 받아도 1점이 되지 않는다. 전체 반영은 `boost_mode`에 따라 sum이다. likes는 비율대로(`factor: 1`) 점수를 부여 받으며 `function_score`간에는 `score_mode`에 따라 sum이다. (default는 multiply)

explain에서 `weight(name:love in 604)`[^fn-so]로 표시되는 604는 내부 document id다.

[^fn-so]: <https://stackoverflow.com/questions/25660053/dont-understand-value-in-elasticsearch-explain-result>

gaussian 수식:  

$$
S(doc) = e^{-\frac{\max(0, |value - origin|-offset)^{2}}{2\sigma^{2}}}
$$

$$
\sigma^{2}=\frac{-scale^{2}}{2\cdot\ln(decay)}
$$

$$2\sigma^{2}$$는 $$\frac{1}{2}$$곱할때 사라지지 않는지. 왜 분모에 굳이 2를 기입하는지 궁금.

<img width="70%" src="https://user-images.githubusercontent.com/1250095/148725940-45f99b14-5108-4fcc-9557-e54e85e8de8a.png">

### Painless
[시작 가이드](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/modules-scripting-using.html) 및 [전반적인 요약](https://www.elastic.co/guide/en/elasticsearch/painless/current/painless-walkthrough.html) 참고.

```
GET my-index-000001/_search?format=yaml
{
  "fields": [
    "title.token_count",
    "title.ngram_token_count"
  ],
  "query": {
    "script_score": {
      "query": {
        "multi_match": {
          "query": "이마트",
          "fields": [
            "title",
            "title.keyword",
            "title.ngram"
          ],
          "type": "most_fields"
        }
      },
      "script": {
        "lang": "painless",
        "source": """
// 거리 점수
double distanceScore = decayGeoExp(params.origin, params.scale, params.offset, params.decay, doc['location'].value);

// _explain 설명 추가
if (explanation != null) {
  explanation.set('_score(' + Math.round(_score * 10000) / 10000.0 + ') + script_score:');
}

// 종합 점수
return _score + distanceScore;
          """,
        "params": {
          "origin": "37.49930, 127.04818",
          "scale": "10km",
          "offset": "0km",
          "decay": 0.5
        }
      }
    }
  },
  "highlight": {
    "fields": {
      "title*": {}
    }
  },
  "explain": true
}
```

한계가 있어 [per-field similarity 계산이 안된다](https://discuss.elastic.co/t/return-per-field-similarity-for-each-of-many-fields/183254/1). 2019년 5월 이슈 제기, 여전히 문제가 해결되지 않았다.
> each search request will produce scores in a single way. 

또한 `combined_fields`는 analyzer가 다르면 사용할 수 없다.
> `combined_fields` requires that all fields have the same search analyzer.[^fn-combined]

[^fn-combined]: <https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-combined-fields-query.html>

새벽에 Painless로 여러가지 실험을 했는데, 다음날 EC2 instance가 다운되어 있다. EC2는 2가지 Status check가 이뤄지는데 이 중 2번째인 Instance reachability check failed가 기록되어 있어 ssh 접속이 되지 않는다. 확인 결과 log를 debug로 설정해 사용하다 disk full로 다운된 것으로 추정된다. modify volume으로 현재 attached 상태를 10G 증설하고, instance reboot 하니 늘어난 용량으로 인식된다.

## 한글
nori에 seunjeon과 동일한 분석실험 진행:

```
PUT my-index-000002
{
  "settings": {
    "index": {
      "analysis": {
        "tokenizer": {
          "nori_tokenizer": {
            "type": "nori_tokenizer",
            "decompound_mode": "mixed"
          }
        },
        "analyzer": {
          "korean": {
            "type": "custom",
            "tokenizer": "nori_tokenizer"
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "korean"
      }
    }
  }
}

POST my-index-000002/_bulk
{"index":{"_id":"1"}}
{"title":"나는 이마트죽전점에서 물건을샀다."}
```

`decompound_mode`는 복합명사 처리 방식인데 `mixed`는 분리하고 원본 데이터 유지. `discard`는 복합명사 분리 후 원본을 삭제한다. 품사 태깅은 `attributes`로 지원한다. [옵션 참고](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori-tokenizer.html)

Inverted Index 조회:
```
GET my-index-000001/_termvectors/1?format=yaml
{
  "fields" : ["title"],
  "offsets" : false,
  "payloads" : false,
  "positions" : false,
  "term_statistics" : false,
  "field_statistics" : false
}

// nori 결과
["ㅏㅆ", "나", "는", "다", "물건", "사", "샀", "에서", "을", "이마트", "점", "죽전"]

// seunjeon 결과
["나", "나는", "물건", "물건을", "사", "샀다", "이마트", "점", "점에서", "죽전"]
```

seunjeon은 어절 색인 옵션 때문에 복합명사가 그대로 색인되는 효과를 낸다. nori에서는 shingle token filter로 비슷한 효과를 낼 수 있다.

default seunjeon과 user_dict가 적용된 seunjeon은 다음과 같이 비교 가능:
```
GET _analyze
{
  "text": "케이마트",
  "tokenizer": "seunjeon_tokenizer"
}

GET my-index-000001/_analyze
{
  "text": "케이마트",
  "analyzer": "seunjeon" 
}
```

# 기타
logstash는 filebeat로부터 받은 로그 파일을 룰에 맞게 파싱해 json 문서로 만드는 역할을 한다. 파싱할 때는 다양한 패턴을 사용할 수 있으며 대부분 grok 패턴을 이용해 파싱 룰을 정한다. (p250, 기초부터 다지는 ElasticSearch 운영 노하우)

## 설정
재밌게도 로그 레벨 실시간 반영이 가능하다.
```
PUT /_cluster/settings
{
  "transient": {
    "logger.org.elasticsearch.tasks.TaskManager": "INFO",
    "logger.org.elasticsearch.search.query.QueryPhase": "TRACE",
    "logger.org.elasticsearch.search.fetch.FetchPhase": "INFO"
  }
}
```
QueryPhase만 추적하도록 설정. 또는 elasticsearch.yml에 다음과 같이 반영한다.
```yaml
logger.org.elasticsearch.search.query.QueryPhase: TRACE
```

이를 통해 Enterprise Search에서 들어오는 쿼리를 모니터링 할 수 있다.

## 자소 분리
자소 분리를 하나로 합치고자 하는 경우(p334, 엘라스틱서치 실무 가이드):
```java
Normalizer.normalize("ㅅㅏㅅㅓㅇㅈㅓㄴㅈㅏ", Normalizer.Form.NFC);
```