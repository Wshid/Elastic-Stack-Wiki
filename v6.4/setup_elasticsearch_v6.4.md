### Set up Elasticsearch, v6.4
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
- Important System Configuration
    - 이상적으로 서버에서 단독으로 실행하고, 사용할 수 있는 모든 리소스를 사용해야 하는 것이 좋음
    - Elasticsearch가 기본적으로 허용하는 것보다, 많은 리소스 액세스를 위해 OS를 구성해야함
    - 고려해야할 것 5가지
        - Disable Swapping
        - Increase file descriptors
        - Ensure sufficient virtual memory
        - Ensure sufficient threads
        - JVM DNS cache settings
    - Development mode vs production mode
        - 기본적으로 ES는 development mode로 작업된다 가정
            - 에러가 로그 파일에 기록되며, ES Node를 시작하고 실행 가능
        - 네트워크 설정시(network.host), production mode로 인식
            - 경고가 Exception 처리 됨
                - ES가 시작되지 않음
                    - 잘못된 구성으로 인한 데이터 유실 방지 측면 때문
    - Configuring System settings
        - ulimit
            - 임시 설정시 사용
            - 리소스 제한을 변경할 때 사용한다.
            - superuser 권한이 필요하다.
            - open file handles 예시
                ```
                sudo su
                ulimit -n 65536
                su elasticsearch
                ```
        - /etc/security/limits.conf
            - 영구 설정시 사용
            - 파일을 여는 수의 최대값을 설정할 수 있음
            - limits.conf 설정 예시
                ```
                elasticsearch   -   nofile  65536
                ```
                - elasticsearch 유저의 권한 변경
                - 새로운 세션이 시작될때 적용된다.
            - 우분투의 경우, 부팅시 limits.conf를 무시하기 때문에
                - /etc/pam.d/su에서 주석 처리를 해야 한다.
                    ```
                    # session   required    pam_limits.so
                    ```
        - Sysconfig file
            - 시스템 구성파일
            - 시스템 설정 및 환경 변수 설정 가능
            ```
            #RPM
            /etc/sysconfig/elasticsearch

            #Debian
            /etc/default/elasticsearch
            ````
        - Systemd configuration
            - 시스템 제한 관련 파일
            - /usr/lib/systemd/system/elasticsearch.service
                - 기본적으로 제한이 걸려 있음
                - 해당 파일 제한을 무시하려면,
                    - **/etc/systemd/system/elasticsearch.service.d/override.conf** 파일을 생성하거나
                    - **sudo systemctl edit elasticsearch** 명령어 실행
                        - 그 이후 다음과 같이 설정
                            ```
                            [Service]
                            LimitMEMLOCK=infinity

                            sudo systemctl daemon-reload
                            ```
    - Disable swapping
        - 대부분 OS는 file systemc cache에 최대한 많은 메모리를 사용하려함
            - 사용되지 않는 것에 대해서는 memeory의 swap이 일어남
                - JVM Heap 또는 Executable Page가 Disk로 Swap 될 수 있음
        - Swapping은 노드 안정성(stability) 성능에 매우 안좋은 영향을 미친다.
            - 되도록 이를 피하는 것이 좋음
        - GC가 n분동안 일어나
            - node가 느리게 응답
            - cluster과의 연결 종료
                - 가 발생할 수 있음
        - Resilient distributed system에서는 OS가 Node를 죽이는 것이 더 효과적
        - Disable swapping 세가지 방법
            - Disable all swap files
                - ES는 box에서 돌아가는 프로그램
                - memory 사용량은 JVM 옵션에 의해 관리
                - 따라서 swap을 사용할 이유가 없음
                - 리눅스 설정방법
                    ```
                    sudo swapoff -a
                    ```
                    - 영구적으로 swap을 끄려면, /etc/fstab에서 swap관련 line을 주석 처리해야 함
                - 윈도우 설정 방법
                    - System Properties - Advanced - performance -> Advanced -> Virtual memory
            - Configure swappiness
                - vm.swappiness = 1
                    - 리눅스의 경우 sysctl로 확인 가능
                    - 1로 설정시, 스왑이 줄어드며, 전체 시스템이 비상상황일 때만 동작
            - Enable bootstrap.memory_lock
                - 프로세스 주소 공간을 RAM에 잠군다.
                    - 리눅스 : mlockall
                    - 윈도우 : VirtualLock
                - ES memory가 swap out 되지 않도록 설정
                - config/elasticsearch.yml
                ```
                bootstrap.memory_lock: true
                ```
                - **mlockall**이 사용 가능한 것보다 많은 메모리 할당시
                    - JVM이나 쉘 세션이 종료될 수 있음
                - ES시작 후, 확인 가능
                ```
                    GET _nodes?filter_path=**.mlockall

                    # return
                    {
                    "nodes": {
                        "LkkPkbE1Q6mAYPwSeCd48Q": {
                        "process": {
                            "mlockall": true
                        }
                        },
                        "7i_Wt4ZKSTKM33Y6nw_3Ww": {
                        "process": {
                            "mlockall": true
                        }
                        }
                    }
                    }
                ```
                - mlockall = false
                    - mlockall 요청이 실패했다는 의미
                        - 세부 로그에서 확인 가능함
                        - **Unbale to lock JVM Memory** 내용이 포함된 구절 찾기
                        - 보통, 사용자에게 memory lock 권한이 없을 경우 발생
                            - .zip || .tar.gz
                                ```
                                ulimit -l unlimited // root권한으로 ES 시작전 실행
                                
                                OR
                                
                                /etc/security/limits.conf // memlock을 unlimited로 설정
                                ```
                            - RPM and Debian
                                - **MAX_LOCKED_MEMORY** 를 **unlimited**로 설정
                                - system configuration file에서 설정
                            - systemd
                                - **LimitMEMLOCK**을 **infinity**로 설정
                        - mlockall이 /tmp와 같은 임시디렉토리에 실행 권한이 없을 경우(noexec)
                            - ES_JAVA_OPS에서 설정 가능
                                ```
                                    export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djava.io.tmpdir=/path/to/temp/dir"
                                    ./bin/elasticsearch
                                ``` 
                                - 혹은 JVM Flag나 jvm.options에서 설정 가능
    - File Descriptors
        - Linux나 macOS에서만 해당
            - Windows에는 해당 없음
                - JVM에서 사용가능한 리소스로만 제한된 [API](https://docs.microsoft.com/ko-kr/windows/desktop/api/fileapi/nf-fileapi-createfilea)사용
        - ES에는 많은 File Descriptor 또는 File Handle 사용
        - 부족할 경우, 데이터 손실이 이어질 수 있음
        - 65,536개 이상으로 설정해주어야 함
        - .zip & .tar.gz
            ```
            ulimit -n 65536 # as root
            
            OR

            /etc/security/limits.conf # nofile -> 65536
            ```
        - macOS
            - JVM 옵션에서 설정가능
                ```
                -XX:-MaxFDLimit
                ```
        - RPM, Debian
            - 이미 65536으로 설정되어 있음
        - [Node Stats API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-stats.html)를 사용하여 확인 가능
            ```
            GET _nodes/stats/process?filter_path=**.max_file_descriptors

            #Return
            {
                "nodes": {
                    "7i_Wt4ZKSTKM33Y6nw_3Ww": {
                        "process": {
                            "max_file_descriptors": 1048576
                        }
                    },
                    "LkkPkbE1Q6mAYPwSeCd48Q": {
                        "process": {
                            "max_file_descriptors": 1048576
                        }
                    }
                }
            }
            ```
    - Virtual Memory
        - 색인을 저장하기 위해 [mmapfs](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-store.html#mmapfs)directory 사용
            - mmapfs([refer](http://knight76.tistory.com/entry/elasticsearch-5-%EC%A0%80%EC%9E%A5%EC%86%8C-%ED%83%80%EC%9E%85store-type))
                - MMapDirectory 파일을 메모리(mmap)에 매핑하여
                    - fsd에 shard index를 저장
                - 메모리 매핑은, 매핑되는 파일의 크기와 동일한
                    - 프로세스에서 가상 메모리 주소 공간의 일부 사용
                - mmapfs를 읽기 위해 mmap system call 사용
                    - 저장시에, 랜덤 액세스 파일 사용
                - 잠금 기능이 없으므로, multi thread 사용시 확장이 가능함
                - OS에서 index file을 읽기 위해 mmap 사용시, 이미 캐시된 것처럼 보임
                    - 가상 공간에 매핑 되었음
                - Lucene index에서 파일을 읽을 때
                    - 해당 파일을 OS cache에 로드할 필요가 없음
                        - 접근이 더 빠르다는 장점
                - Lucene과 ES가 I/O 캐시에 직접 접근할 수 있어, index file에 빠른 접근이 가능
            - 64비트 환경에서 가장 잘 작동
            - 32비트 환경에서 다음 조건 만족시에만 사용
                - index가 충분히 적고
                - 주소 공간이 충분할 때
            - mmapfs를 사용하려면, **index.store.type**을 **mmapfs**로 설정

        - .mmap 카운트 제한이 낮으면, memory leak가 발생할 수 있음
        - 리눅스 설정
            ```
            sysctl -w vm.max_map_count=262144 # as root
            ```
            - 영구적으로 설정하려면, /etc/sysctl.conf에서 설정
                - reboot후, ```sysctl vm.max_map_count`` 확인
        - RPM이나 Debian은 자동 설정되어 있음
    



                                   






    










