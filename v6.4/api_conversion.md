### api-conversion
---
- Multiple Indices
    - index parameter를 multiple하게 줄 수 있다.
        ```test*, *test, te*t 등```
        ```test*,-test3``` : Exclude 연산
    - Query String parameter
        - ```ignore_unavailable```
            - 지정된 index 사용 불가시, 무시할지 결정
            - 존재하지 않는 인덱스 및 closed index를 포함한다
        - ```allow_no_indices```
            -  결과가 구체화 되지 않으면 실패할지 여부 결정
                ```foo*``` 명령시, 그런 인덱스가 존재하지 않으면 실패
        - ```expand_wildcards```
            - wildcard가 포함된 식이 확장되도록 제어
                - open : 오직 활성화 된 index만
                - close : only closed index만
                - open, close를 모두 주어, 전체 검색이 가능하다.