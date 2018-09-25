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





