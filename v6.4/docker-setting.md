### docker-setting
---
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

            sudo sysctl -w vm.max_map_count=262144
            ```


