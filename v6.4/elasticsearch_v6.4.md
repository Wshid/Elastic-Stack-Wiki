### Elasticsearch 6.4
---
- type이 제거됨
    - 내부적 index로만 관리된다
- shard
    - 하나의 인덱스에 대하여 여러 노드에 분할하여 저장 가능
    - 클러스터의 모든 노드에서 호스팅 가능
- replica
    - 복제본
    - 하나의 인덱스에 대해 여러개로 복제하여 저장
    - 정확히 인덱스에 해당하는 샤드별 복제
- 버전 지원 관련
    - java 8, jdk 1.8.0_131이상
- 실행 방법
    ```
    ./elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name
    ```
- Exploring Your Cluster
    - REST API
        - 클러스터 및 노드의 상태 확인 가능
        - CRUD 및 인덱스에 관한 작업
        - 페이징, 정렬, 필터링 등 가능
    - Cluster Health
        - GET /_cat/health?v
            - _cat api
                - 클러스터 헬스 체크를 하기 위한 명령어
                    ```
                    curl -X GET "localhost:9200/_cat/health?v"
                    ```
        - 상태
            - Green
            - Yellow
                - 모든 데이터가 사용 가능하나, 일부 replica에서 할당되지 않음
            - Red
                - 특정 이유에 따라 몇 데이터가 활용 가능하지 않음
    - Index
        - List All Indices
            ```
            GET /_cat/indices?v
            ```
        - Create an Index
            ```
            PUT /customer?pretty
            GET /_cat/indices?v
            ```
            - pretty : 정리된 자료를 보기 위함
    - Index and Query a Document
        ```
        PUT /customer/_doc/1?pretty
        {
            "name": "Sion Kim"
        }

        
        GET /customer/_doc/1?pretty
        ```
        - index : customer, id = 1
        - 결과 내용에서 _doc이라는 type이 존재
    - Delete an Index
        ```
        DELETE /customer?pretty
        GET /_cat/indices?v
        ```
        ```
        PUT /customer/_doc/1
        {
            "name": "Sion Kim"
        }

        #RETURNED
        {
        "_index": "customer",
        "_type": "_doc",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "_shards": {
            "total": 2,
            "successful": 2,
            "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1
        }
        ```
        - <REST Verb> /<Index>/<Type>/<ID>
            - 6.0.0부터 type이 사라졌다고 하지만, 이와 같은 형태를 띈다.
    - Modifiying Your Data
        ```
        PUT /customer/_doc/1?pretty
        {
        "name": "John Doe"
        }
        ```
        - 아이디까지 모두 입력후, PUT METHOD를 이용하면 된다.
        - 이름에 해당하는 속성이 변경되게 된다.
        - 수정되는데에 조금의 시간이 소요될 수 있음
            - SQL과의 차이점(트랜잭션 바로 적용)
- Modifying Your Data
    - Updating Document
        - 실제로 업데이트를 하는 것이 아님
        - 기존 문서를 삭제한 다음, 업데이트가 적용된 새 문서의 색인 생성
        - POST 명령을 사용한다.
            ```
            POST /customer/_doc/1/_update?pretty
            {
            "doc" : {"name": "Jane Doe"}
            }
            ```
            - 실제 결과상, PUT과 큰 차이가 보이지 않음
    - PUT과 POST의 차이
        - PUT
            - 전송 필드 및 전체 문서 업데이트
            - id를 무조건 지정해주어야 함
                ```
                PUT /customer/_doc/2?pretty
                {
                "name": "John Doe"
                }
                ```
        - POST
            - 전송한 필드만 업데이트
            - 문서 내의 다른 필드를 변경하지 않음
            - id를 지정하지 않아도 문서 정의 가능
                ```
                POST /customer/_doc
                {
                "foo" : "bar"
                }
                ```
                - POST를 PUT으로 변경시 에러 발생
    - Deleting Documents
        ```
        DELETE /customer/_doc/2?pretty
        ```
    - Batch Processing
        - _bulk API를 사용하여 다중 구문 처리가 가능하다
        - multiple operation일때 성능이 뛰어나다
        - Indexing
            - 검색 엔진에서 빠른 검색을 위해 데이터를 정리하는 방법
                - refer : [aws](https://docs.aws.amazon.com/ko_kr/elasticsearch-service/latest/developerguide/es-indexing.html)
        - Index
            - PUT과 같이 document의 내용을 이 자체로 바꾸게 된다.
            - 다른 필드에도 영향을 미친다는 의미
            ```
            POST /customer/_doc/_bulk?pretty
            {"index":{"_id":"1"}}
            {"name" : "Ho"}
            {"index":{"_id":"2"}}
            {"name" : "aa"}
            ```
        - Update & Delete
            - 일반적인 POST 연산과 동일하다
            ```
            POST /customer/_doc/_bulk?pretty
            {"update":{"_id":"1"}}
            {"doc":{"name":"John Doe becomes Jane Doe", "age" : "24"}}
            {"delete":{"_id":"2"}}
            ```
- Exploring Your Data
    - 샘플 데이터 구성
        - accounts.json 파일을 사용하여 샘플 데이터 인입
        ```
        curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
        curl "localhost:9200/_cat/indices?v"
        ```
        - accounts.json 파일 구성시, 마지막은 개행문자를 넣어주어야 한다.
            - _bulk api의 요구사항
            - 적용하지 않을경우 에러 발생
    - The Search API 
        - REST API를 사용한다.
            - _search API
        - 구문
            ```
            GET /bank/_search?q=*&sort=account_number:asc&pretty
            ```
            ```
            curl -X GET "localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty"
            ```
            ```
            GET /bank/_search
            {
                "query": {"match_all": {}},
                "sort": [
                    {
                    "account_number": {
                        "order": "desc"
                    }
                    }
                ]
            }
            ```
            - **q=\*** : 해당 인덱스 모든 문서에서 찾음
            - **sort=account_number:asc** account_number, ascending order
        - 결과 분석
            - 결과
                ```
                {
                    "took": 81,
                    "timed_out": false,
                    "_shards": {
                        "total": 5,
                        "successful": 5,
                        "skipped": 0,
                        "failed": 0
                    },
                    "hits": {
                        "total": 1000,
                        "max_score": null,
                        "hits": [
                        {
                            "_index": "bank",
                            "_type": "_doc",
                            "_id": "0",
                            "_score": null,
                            "_source": {
                            "account_number": 0,
                            "balance": 16623,
                            "firstname": "Bradshaw",
                            "lastname": "Mckenzie",
                            "age": 29,
                            "gender": "F",
                            "address": "244 Columbus Place",
                            "employer": "Euron",
                            "email": "bradshawmckenzie@euron.com",
                            "city": "Hobucken",
                            "state": "CO"
                            },
                            "sort": [
                            0
                            ]
                        },
                    ...
                ```
                - **took** : 걸린시간(ms)
                - **timed_out** : T/F, timeout
                - **_shards** : 샤드 몇개에서 찾았는지, 성공/실패 여부
                - **_hits** : 결과
                    - **hits.total** : 결과 개수
                    - **hits.hits** : 실제 결과, default=10
                    - **hits.sort** : 결과에 대한 정렬 키 
    - Introducing the Query Language
        - 전체 검색 기본
        - size, from-size 등의 여러 옵션을 줄 수 있다.
            ```
            GET /bank/_search
            {
            "query": {"match_all": {}},
            "from" : 10,
            "size" : 10
            }
            ```
                - id별로 정렬되서 10~19까지의 데이터를 요구하는 것이 아님
        - sort
            ```
            GET /bank/_search
            {
            "query": {"match_all": {}},
            "sort": [
                {
                "balance": {
                    "order": "desc"
                }
                }
            ]
            }
            ```
    - Executing Searches
        - _source
            - 실제 해당하는 데이터 목록은 _source 필드 내부에 존재
            - 이를 활용하여 원하는 attr만 추출하여 검색할 수 있다.
            ```

            ```
            GET /bank/_search
            {
            "query": {"match_all": {}},
            "_source": ["account_number", "balance"]
            }
            ```
            "hits": [
            {
                "_index": "bank",
                "_type": "_doc",
                "_id": "25",
                "_score": 1,
                "_source": {
                "account_number": 25,
                "balance": 40540
                }
            },
            ```
            - hits 하단의 _source에서 두가지 필드만 검출된다.
        - match
            - match_all의 경우 전 문서 검색
            - match를 사용하여 조건 검색이 가능하다.
                ```
                GET /bank/_search
                {
                "query": {"match": {
                    "account_number": 20
                }}
                }
                ```
        - bool
            - bool 쿼리를 사용하여 조합이 가능
            - must, should, must_not등이 올 수 있다.
            - must : 모두 포함(AND)
                ```
                GET /bank/_search
                {
                "query": {
                    "bool": {
                    "must": [
                        {"match": {
                        "address": "mill"
                        }},
                        {"match": {
                        "address": "lane"
                        }}
                    ]
                    }
                }
                }
                ```
            - should : 한가지 이상 포함(OR)
                ```
                GET /bank/_search
                {
                "query": {
                    "bool": {
                    "should": [
                        {"match": {
                        "address": "mill"
                        }},
                        {"match": {
                        "address": "lane"
                        }}
                    ]
                    }
                }
                }
                ```
            - must_not : 한 개이상 포함하지 않음(nor)
            - 두 가지 이상 조합이 가능하다.
                ```
                GET /bank/_search
                {
                "query": {
                    "bool": {
                    "must": [
                        {"match": {
                        "age": 40
                        }}
                    ],
                    "must_not": [
                        {"match": {
                        "state": "ID"
                        }}
                    ]
                    }
                }
                }
                ```
                - 나이가 40살 이면서, ID에 살지 않는 사람을 구하는 쿼리
                - bool하단의 must와 must_not은 AND로 연접되는듯 하다.
    - Executing Filters
        - score
            - 문서가, 지정한 검색어와 얼마나 일치하는지의 대한 점수
        - 불필요한 것에 대한 scoring은 성능 저하를 일으킨다
        - 따라서 bool 하단에 filter를 사용하여, 불필요한 내용을 사전 제거 하여 성능을 높일 수 있음
        ```
        GET /bank/_search
        {
        "query": {
            "bool": {
            #    "must": [
            #        {"match_all": {}}
            #    ], 
            "filter": {
                "range": {
                "balance": {
                    "gte": 20000,
                    "lte": 30000
                }
                }
            }
            }
        }
        }
        ```
        - 기본적으로 match_all 한결과에 filter가 적용된다.
        - 실제 위 쿼리에서 bool 하위에 match_all : {} 내용을 넣어도 결과는 같다.
- Executing Aggregations
    - SQL의 GROUP BY 절과 유사
    - 네트워크 트래픽을 피하면서, 효율적 쿼리가 가능
    - 구문
        ```
        GET /bank/_search
        {
        "size":0,
        "aggs": {
            "group_by_state": {
            "terms": {
                "field": "state.keyword"
            }
            }
        }
        }

        # same 
        # SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC LIMIT 10;

        ```
        - **size:0**
            - _search를 했을때 본문 결과가 기록된다.
                - hits.hits와 연관
            - 해당 내용의 사이즈를 0으로 둠으로써, aggre 내용만 확인하게 된다.
        - **group_by_state**
            - 단순 field 이름
            - SQL에서 AS <Field Name>과 동일
    - 중첩 구문
        - 해당 필드명, terms 하단에 추가적으로 기입하면 된다.
        ```
        GET /bank/_search/
        {
            "size":0,
            "aggs":{
                "group_by_state":{
                    "terms": {
                        "field": "state.keyword",
                        "size": 10
                    },
                    "aggs": {
                        "average_balance": {
                            "avg": {
                                "field": "balance"
                            }
                        }
                    }
                }
            }
        }
        ```
        - terms외에 range, avg등 여러가지 명령이 가능하다.
        - [aggregation reference guide](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-aggregations.html)
        - aggregation의 4가지 유형
            - Bucketing
                - 각 bucket이 문서 기준과 연관되어 있음
                - 연산의 결과는 해당 bucket 목록에 속하는 문서 집합 표시
            - Metric
                - 문서 집합에 대해서 연산
            - Matrix
                - 행렬 결과
                - 여러 필드에서 작동, 문서 필드에서 추출한 값을 기반으로 함
            - Pipeline
                - 다른 집계 및 관련 메트릭의 출력 집계
- Set up Elasticsearch
    - 





            




        
        









        





    
    
    
