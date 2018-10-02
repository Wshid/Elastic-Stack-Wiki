### Configuration, v6.4
---
- Configuring Elasticsearch
    - 구성이 대부분 필요하지 않으며, [Cluster Update Setting API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)로 설정이 가능함
    - 설정 파일
        - elasticsearch.yml 
        - jvm.options
            - \- Xmx2g : JVM버전과 독립적 설정, Max Heap Size 2G
            - 8:\-Xmx2g : Java 8, Max Heap Size 2G
        - log4j2.properties
- Important Elasticsearch Configuration
    - path.data, path.logs
        - 기본적으로 **$ES_HOME/log/elasticsearch**와 같이 기입된다.
        - 버전 업그레이드 상에 지워질 위협이 존재하므로 다음과 같이 설정하여 피한다
        ```
        path:
            data:
                - /mnt/elasticsearch1
                - /mnt/elasticsearch2
                - /mnt/elasticsearch3
        ```
        - 위와 같이 다중 경로 설정도 가능하다.
    - cluster.name
        - 하나의 노드를 다른 노드와 공유할때, 클러스터 구성이 가능함
        - default : elasticsearch
        - 다른 환경에서 클러스터 이름이 중복되지 않도록 설정 해야 함
    - node.name
        - default: UUID first 7 character
        - Node ID는변하지 않음
        - Node Name으로 HOSTNAME도 사용 가능하다
            ```
            node.name: ${HOSTNAME}
            ```
    - network.host
        - default : 127.0.0.1 and [::1]
        - $ES_HOME 단일 노드의 동일한 위치에서 두개 이상의 노드 시작 가능
            - 권장하는 방법은 아님
        - [Special values for network.host](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#network-interface-values)
    - Discovery settings
        - Zen Discovery 사용
            - clustering 및 마스터 선정을 위해 사용
        - 두가지 설정
            - discovery.zen.ping.unicast.hosts
                - 자동적으로 9300 ~ 9305 포트를 스캔하여 노드를 클러스터 구성
                - 노드 목록을 제공해야 제대로 된 구성이 가능
                ```
                discovery.zen.ping.unicast.hosts:
                    - 192.168.1.10:9300
                    - 192.168.1.11 
                        # port 미지정시, port는 transport.profiles.default.port
                        # transport.tcp.port로 설정됨
                    - seeds.mydomaion.com
                        # 하나의 호스트가 여러 ip이름 가질 시, 모든 노드 탐색
                ```
            - discovery.zen.minimum_master_nodes
                - data loss를 막기 위해 마스터 적합 노드 설정
                - 이를 설정하지 않으며니, 네트워크 장애 발생시, 분리된 두개의 클러스터로 구성될 수 있음
                - 설정 공식 : **(master_eligible_nodes / 2)+1**
                    ```    
                       # 3개의 master eligible node가 있을 때, 최소 마스터 노드를 설정하려면
                       # (3/2)+1 = 2로 설정해야함
                       discovery.zen.minimum_master_nodes: 2
                    ```
        - Heap Size
            - default : Xmx=xMs=1G
            - 충분한 힙 사이즈가 설정되어야 함
            - Xms와 Xmx 설정을 해주어야 한다.
                - Minimum Heap Size, Maximum Heap Size
            - ES가 사용할 수 있는 memory가 많을 수록 좋다.
                - 하지만 GC발생시, 그만큼의 pause시간이 길어질 수 있다.
            - Xmx 설정시, Physical RAM의 50% 이하로 설정하여 테스트
            - Xmx 설정시, cutoff 보다 높게 설정하지 말 것
                - cutoff size는 약 32GB
                - 해당 로그를 확인하여(compressed oops)를 확인할 수 있음
                    ```
                        heap size [1.9gb], compressed ordinary object pointers [true]
                    ```
                - oops에 대한 임계값 이하로 유지할 수 있도록 시도할 것
                    - 제한 방법(JVM options)
                        ```
                        -XX:+UnlockDiagnosticVMOptions -XX:+PrintCompressedOopsMode
                        ```
                    - 로그 확인
                        ```
                        heap address: 0x000000011be00000, size: 27648 MB, zero based Compressed Oops
                        ```
                    - Zero-based compressed oops
                        ```
                        heap address: 0x0000000118400000, size: 28672 MB, Compressed Oops with base: 0x00000001183ff000
                        ```
                - compressed oops와 관련된 글 : [brunch](https://brunch.co.kr/@alden/35)
                    - OOP(Ordinary Object Pointer)
                        - Object에 접근하기 위한 메모리상의 주소
                        - 32비트 기반에서 Heap영역이 4GB이상일때, Object를 가리킬 수 없는 문제,
                            - 이를 극복하기 위해 Compressed OOP를 사용
                        - 32비트로 사용하지만 4GB이상을 가리킬 수 있다.
                        - Native OOP에 비해 8의 n배수, 8배 많은 주소공간 표현 가능
                        - left shift 연산을 사용한다.
                            ```
                            native oop = (compressed oop << 3)
                            ```
                            - shift 연산은 상대적으로 빠름, CPU의 부하가 적다.
                        - 이 때문에 32GB까지 원활하게 지원되나, 그 이상일 경우
                            - 64bit기반의 OOP를 사용하게 된다.
                                - 성능이 매우 떨어짐
                        - Compressed OOP의 사용 시점 확인하기(임계치 확인)
                            ```
                            java -Xmx32766 -XX:+PrintFlagsFinal 2> /dev/null | grep UseCompressedOops
                            ```
                    - Zero Based Heap Memory
                        - 64bit 시스템에서 Compressed OOP를 사용하는 시점이오면,
                            - 운영체제에게 Heap영역의 시작주소를 0에서 부터 시작하도록 요청
                            - 주소공간이 0부터 시작되면 shift연산이 필요 없음
                                - 아닐 경우, shift 연산 + base 기반 덧셈 연산 -> 주소 변환 부하 증가
                        - zero base 임계치는 JAVA_OPTS를 설정하여 실행
                    - 적당한 Heap Memory
                        - 너무 작은 메모리의 경우 OOM 발생
                        - 너무 큰 메모리의 경우, GC 발생시 긴 pause time 발생
                        - Rule
                            - 시스템 전체 메모리의 절반만 사용한다.
                            - Compressed OOP를 사용할 수 있도록 32GB 이하로 사용
                                - 테스트를 통해 Compressed OOP를 사용하는 임계치 확인 필요
                            - Zero Base Compressed OOP를 사용할 수 있는 임계치 확인
                    - 결론
                        - 시스템 전체 메모리의 절반 이하
                        - Zero based, Compressed OOP를 사용할 수 있는 임계치 보다는 낮은 값

            - jvm.options file 설정
                ```
                -Xms2g
                -Xmx2g
                ```
            - 환경변수로 설정하기
                ```
                ES_JAVA_OPTS="-Xms2g -Xmx2g" ./bin/elasticsearch 
                ES_JAVA_OPTS="-Xms4000m -Xmx4000m" ./bin/elasticsearch 
                ```
        - JVM Heap dump path
            - jvm.options 수정
            ```
            -XX:HeapDumpPath=...
            ```
            - 설정시 JVM에서 실행중인 인스턴스의 pid를 기반으로 Heap Dump File 생성
            - 이름 지정시, 이미 존재하는 파일일 경우 heap dump가 실패할 수 있음
        - GC logging
            - jvm.options 에서 설정 가능
            - default : 64M단위 순환, 최대 2G 용량
            ```
            ## GC configuration
            -XX:+UseConcMarkSweepGC
            -XX:CMSInitiatingOccupancyFraction=75
            -XX:+UseCMSInitiatingOccupancyOnly
            ```
        - Temp Directory
            - 기본적인 시스템 임시 디렉토리(/tmp)에 사용함
            - 보관하려면, 따로 temp directory를 설정하여 관리하는 것이 좋음
                - 단, elasticsearch를 사용하는 사용자만 설정하는 것이 좋음(권한 설정)
            - $ES_TEMPDIR 환경변수를 설정
        - JVM fatal error logs
            - jvm.options
                ```
                -XX:ErrorFile-...
                ```
        








