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
- kibana w/ docker
    - reference : [Kibana w/ Docker](https://www.elastic.co/guide/en/kibana/current/docker.html)
    - docker-compose.yml
        - elasticsearch보다 간단하며, 기본 설정만 돌려도 된다
        - [detail option kibana](https://www.elastic.co/guide/en/kibana/current/settings.html)
        - volumes 설정이 먹지 않는다.
            - Not a Directory와 같은 에러 출력


