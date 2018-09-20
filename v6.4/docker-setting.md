### docker-setting
---
- reference : [ES with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/docker.html)
- docker image download
    ```
    docker pull docker.elastic.co/elasticsearch/elasticsearch:latest
    ```
- running command
    - development mode
        ```
        docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:latest
        ```
    - production mode
        - Linux
            - VM 설정을 해주어야 한다(vm.max_map_count)
            - 확인 방법
                ```
                $ grep vm.max_map_count /etc/sysctl.conf

                $ sysctl -w vm.max_map_count=262144
                ```
        - Mac
            - xhyve virtual machine
            - docker
            ```
            $ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
            # 이대로 실행시키면 cannot exec 메세지 발생

            ~/Library/Containers/com.docker.docker/Data/vms/0
            # 이 경로에 tty가 존재한다.
            
            cd ~/Library/Containers/com.docker.docker/Data/vms/0
            screen tty
            sudo sysctl -w vm.max_map_count=262144
            cat /proc/sys/vm/max_map_count # 262144인지 확인
            ```
            이후 screen에서 탈출하기 위해 'ctrl+A+\\'를 입력한다.
    - docker-compose.yml 파일 작성
        - docker-compose up을 하기 위한 설정 파일을 생성한다.
        - 설정파일을 수정후 구동시켜 본다
        - 추가 Optimization 과정은 이후 진행토록 한다.
    - xpack 설정 끄기(refer : [elatov](https://elatov.github.io/2017/09/run-elasticsearch-and-kibana-on-docker/))
        - 기본적으로 elasticsearch container에는 xpack이 구동된다.
        - docker-compose.yml의 환경변수를 수정한다.
            ```
            "XPACK_SECURITY_ENABLED=false"
            "XPACK_GRAPH_ENABLE=false"
            "XPACK_WATCHER_ENABLED=false"
            "XPACK_ML_ENABLED=false"
            "XPACK_MONITORING_ENABLED=false"
            "XPACK_MONITORING_UI_CONTAINER_ELASTICSEARCH_ENABLED"
            ```
        - docker-compose up을 하게 되면 설정이 초기화 되는 현상 발생, 다른 방법 강구
- kibana w/ docker
    - reference : [Kibana w/ Docker](https://www.elastic.co/guide/en/kibana/current/docker.html)
    - docker-compose.yml
        - elasticsearch보다 간단하며, 기본 설정만 돌려도 된다
        - [detail option kibana](https://www.elastic.co/guide/en/kibana/current/settings.html)
        - volumes 설정이 먹지 않는다.
            - Not a Directory와 같은 에러 출력
            - 로컬에 파일(kibana.yml)이 있어야 구성 가능하다.
- logstash w/ docker
    - reference : [Logstash w/ Docker](https://www.elastic.co/guide/en/logstash/current/docker.html)

- docker
    - docker __exec__ vs docker **attach**
        - exec의 경우 쉘 지정이 가능하며, 중단시에 컨테이너가 죽지 않는다.
        - attach의 경우 bash로 구동되며, 중단시에 컨테이너가 같이 stop된다.
            - reference : [code.i](https://code.i-harness.com/ko-kr/q/1d86c2e)
    - docker volume container
        - reference : [joinc](https://www.joinc.co.kr/w/man/12/docker/Guide/DataWithContainer)
        ```
        docker volume ls # 볼륨 컨테이너 목록 확인
        docker volume prune # 비사용 볼륨 컨테이너 제거
        docker volume inspect <container name> # 볼륨 상세 정보 확인
        ```
        - **prune을 함부로 사용해선 안된다. 기존 컨테이너가 망가지는 경우가 발생한다.**
        - **컨테이너가 망가질 경우, 제거 후, compose up을 해주어야 한다.**
        - **다른 컨테이너와 결부되어 있는가를 확인 후 제거해야한다.**
    - docker network
        - reference : [official docker](https://docs.docker.com/compose/networking/)
        ```
        docker network ls # 네트워크 어댑터 확인
        docker network prune # 비사용 어댑터 제거
        docker network inspect <network name> # 네트워크 상세 정보 확인

        


