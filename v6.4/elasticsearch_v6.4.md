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
        





    
    
    
