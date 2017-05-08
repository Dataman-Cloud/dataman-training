# Mesos-Docker-Exector
## Mesos-Slave on Docker 操作
- 启动 mesos-slave 
    - 将 docker 所需的资源，如:docker.socket 等挂载到 mesos-slave 中，供 mesos-slave 的 executor 调用 docker 使用
- mesos-docker-exector

        \_ mesos-docker-executor \
            --container=mesos-b7a6d26b-d5b9-4e94-b9b4-c95cceaa6306-S10.16cfaa0d-4d74-4328-b5df-3ec54fc03b5e \
            --docker=docker \
            --docker_socket=/var/run/docker.sock \
            --help=false \
            --launcher_dir=/usr/libexec/mesos \
            --mapped_directory=/mnt/mesos/sandbox \
            --sandbox_directory=/data/mesos/slaves/b7a6d26b-d5b9-4e94-b9b4-c95cceaa6306-S10/frameworks/b7a6d26b-d5b9-4e94-b9b4-c95cceaa6306-0000/executors/test-10.9511424763366447.ae9edd41-9f6c-11e6-a729-0242dd0ee78c/runs/16cfaa0d-4d74-4328-b5df-3ec54fc03b5e \
            --stop_timeout=0ns 
       
    - `container`
        
        mesos 容器id
    - `docker`
    
        docker 执行器
    - `docker_socket`
    
        docker socker 路径
    - `help`
    
        ?
    - `launcher_dir`
    
        mesos-slave 各种二进制执行文件路径
    - `mapped_directory`
    
        docker 内部映射 mesos 沙盒目录路径
    - `sandbox_directory`
    
        mesos 沙盒实体路径                        
    - `stop_timeout`
        
        ？
- mesos-docker-exector 启动的 docker 行为

        docker -H unix:///var/run/docker.sock run \
                --cpu-shares 10 \
                --memory 33554432 \
                -e MARATHON_APP_VERSION=2016-10-31T13:12:53.345Z \
                -e HOST=172.16.0.64 \
                -e MARATHON_APP_RESOURCE_CPUS=0.01 \
                -e MARATHON_APP_RESOURCE_MEM=16.0 \
                -e MARATHON_APP_DOCKER_IMAGE=offlineregistry.dataman-inc.com:5000/library/blackicebird/2048 \
                -e MARATHON_APP_LABEL_GROUP_ID=1 \
                -e MESOS_TASK_ID=test-10.2766033084872659.b484dffe-9f6c-11e6-a729-0242dd0ee78c \
                -e PORT=31382 \
                -e MARATHON_APP_LABEL_VCLUSTER=test \
                -e MARATHON_APP_LABEL_USER_ID=2 \
                -e PORTS=31382 \
                -e PORT_10091=31382 \
                -e MARATHON_APP_RESOURCE_DISK=0.0 \
                -e MARATHON_APP_LABELS=GROUP_ID USER_ID VCLUSTER \
                -e MARATHON_APP_ID=/test-10.2766033084872659 \
                -e PORT0=31382 \
                -e MESOS_SANDBOX=/mnt/mesos/sandbox \
                -e MESOS_CONTAINER_NAME=mesos-b7a6d26b-d5b9-4e94-b9b4-c95cceaa6306-S10.54b0f329-46b0-4a63-975b-90c2075f3d61 \
                -v /data/mesos/slaves/b7a6d26b-d5b9-4e94-b9b4-c95cceaa6306-S10/frameworks/b7a6d26b-d5b9-4e94-b9b4-c95cceaa6306-0000/executors/test-10.2766033084872659.b484dffe-9f6c-11e6-a729-0242dd0ee78c/runs/54b0f329-46b0-4a63-975b-90c2075f3d61:/mnt/mesos/sandbox \
                --net bridge \
                --label=APP_ID=test-10.2766033084872659 \
                --name mesos-b7a6d26b-d5b9-4e94-b9b4-c95cceaa6306-S10.54b0f329-46b0-4a63-975b-90c2075f3d61 offlineregistry.dataman-inc.com:5000/library/blackicebird/2048            