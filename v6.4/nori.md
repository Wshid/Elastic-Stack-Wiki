### Korean(nori) Analysis Plugin
---
- Refer : [nori](https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-nori.html)
- nori
    - nori_tokenizer
    - nori_part_of_speech token filter
    - nori_readingform token filter
    - lowercase token filter
- nori_tokenizer
    - decompound_mode
        - compound token에 대해서 어떻게 다룰 것인지 대해 서술
            - none
                - 합성어(compound)에 대해 분리하지 않는다.
            - discard
                - 합성어를 분리한다.
                - default
            - mixed
                - 원본 값과, 분리된 값을 모두 리턴한다.
    - user_dictionary
        - nori는 기본적으로 mecab-ko-dic 사전을 기반으로 동작한다.
        - user_dictionary를 주어 동작할 수 있다.
            - custom nouns(NNG) 지정 가능
            - ```<token> [<token1> ... <token n>]```
                - first token
                    - 사용자 정의 명사
                        [...] 내부에 맞춤 세그먼트 입력
        - 예시
            ```
                $ES_HOME/config/userdict_ko.txt

                c++
                C샤프
                세종
                세종시 세종 시 #compound noun
            ```
    - analyzer
        ```
        PUT nori_sample
        {
        "settings": {
            "index": {
                "analysis": { #analyzsis 사용
                    "tokenizer": {
                        "nori_user_dict": {
                            "type": "nori_tokenizer",
                            "decompound_mode": "mixed",
                            "user_dictionary": "userdict_ko.txt"
                            }
                        },
                        "analyzer": {
                            "my_analyzer": {
                                "type": "custom",
                                "tokenizer": "nori_user_dict"
                            }
                        }
                    }
                }
            }
        }

        GET nori_sample/_analyze
        {
        "analyzer": "my_analyzer",
        "text": "세종시"  
        }
        ```
        


    