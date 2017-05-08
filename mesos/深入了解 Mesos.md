# 深入了解 Mesos
## Mesos 系统试验
### 卸载 Mesos-Master 并重新安装
- 操作
    - 关闭所有 Mesos-Master
    - Mesos-Master 存储本地持久化目录被删除
    - Zookeeper 中的数据存在
    - 重启服务
- 结果
    - 服务无影响

### 删除 Mesos-Master Zookeeper 数据
- 操作
    - 关闭所有 Mesos-Master
    - 使用 zkCLI 登陆到 Zookeeper 中
    - 删除 /mesos 目录下面数据
    - 重启服务
- 结果
    - 服务恢复正常，所有 task 被重启

### Master-Leader 更换
- 操作
    - 关闭 Mesos-Master Leader
- 结果
    - 服务无影响                       

### 卸载 Zookeeper 并重新安装
- 操作
    - 关闭所有 Zookeeper
    - Zookeeper 存储本地持久化目录被删除
    - 重启服务
- 结果
    - marathon 全部故障，无法正常运行
    - 所有应用丢失
- 处理
    - 重启所有 marathon
    - 重发应用      
     
## Mesos 系统 Zookeeper 中的目录结构
- /(根目录)
    - mesos
        - json.info_0000000000 (mesos 启动建立、关闭删除，编号小的是 leader，同步锁机制)
                
                {"address":{"hostname":"192.168.1.232","ip":"192.168.1.232","port":5050},"hostname":"192.168.1.232","id":"bd5969e1-d8d3-492c-abfc-3b2367465209","ip":3892422848,"pid":"master@192.168.1.232:5050","port":5050,"version":"1.0.1"}
        - json.info_0000000003 
            
                {"address":{"hostname":"192.168.1.233","ip":"192.168.1.233","port":5050},"hostname":"192.168.1.233","id":"d579c316-0102-484d-932a-d6548bb11c16","ip":3909200064,"pid":"master@192.168.1.233:5050","port":5050,"version":"1.0.1"}
        - json.info_0000000002 
        
                {"address":{"hostname":"192.168.1.231","ip":"192.168.1.231","port":5050},"hostname":"192.168.1.231","id":"b6bcae59-1564-4fcc-9adb-d4d23eecdcc7","ip":3875645632,"pid":"master@192.168.1.231:5050","port":5050,"version":"1.0.1"}
        - log_replicas (mesos 持久化纪录)
            - 0000000001 
                
                    log-replica(1)@192.168.1.231:5050
            - 0000000000
            
                    log-replica(1)@192.168.1.232:5050
            - 0000000002       
            
                    log-replica(1)@192.168.1.233:5050
    - zookeeper
        - quota
    - marathon
        - leader
            - member_0000000003
            
                    192.168.1.231:8080
            - member_0000000001 (编号小的是 leader，同步锁机制)
            
                    192.168.1.232:8080
            - member_0000000002
            
                    192.168.1.233:8080
        - state
            - group:root:2016-12-13T06:36:14.897Z(组应用版本)
            
                    #group:root:2016-12-13T06:36:14.897Z$97c22d8a-23a3-4f54-b694-f26954b0d0ac?
                    /2016-12-13T06:36:14.897Z?
                    /cluster1-20480"
                    cpus		{?G?z??2*"
                    mem		0@2*"
                    disk		2*:
                    vclustecluster1:
                    typeswarmBZ2016-12-13T06:36:14.897Zh?qffffff??z	???????
                    >offlineregistry.dataman-inc.com:5000/library/blackicebird-2048 *
                    labelAPP_ID=cluster1-2048*
                    labelVCLUSTER=cluster10?
                    GROUP_ID1?

                    USER_ID1?
                    VCLUSTEcluster1??????????+??????+tcp
            - group:root (当前组)

                    group:root$6055b570-bcd3-494d-9f23-a4efb7973fbe?
                    /2016-12-13T06:36:14.897Z?
                    /cluster1-20480"
                    cpus		{?G?z??2*"
                    mem		0@2*"
                    disk		2*:
                    vclustecluster1:
                    typeswarmBZ2016-12-13T06:36:14.897Zh?qffffff??z	???????
                    >offlineregistry.dataman-inc.com:5000/library/blackicebird-2048 *
                    labelAPP_ID=cluster1-2048*
                    labelVCLUSTER=cluster10?
                    GROUP_ID1?

                    USER_ID1?
                    VCLUSTEcluster1??????????+??????+tcp                   
            - framework:id(调度器id)
                            
                    framework:id$36313331-3634-3634-2d36-3133302d3336+)bd5969e1-d8d3-492c-abfc-3b2367465209-0000
            - app:cluster1-2048(当前应用)

                    app:cluster1-2048$7dfcaef0-197c-467f-84e0-70c76c28cc9c?
                    /cluster1-20480"
                    cpus		{?G?z??2*"
                    mem		0@2*"
                    disk		2*:
                    vclustecluster1:
                    typeswarmBZ2016-12-13T06:36:14.897Zh?qffffff??z	???????
                    >offlineregistry.dataman-inc.com:5000/library/blackicebird-2048 *
                    labelAPP_ID=cluster1-2048*
                    labelVCLUSTER=cluster10?
                    GROUP_ID1?

                    USER_ID1?
                    VCLUSTEcluster1??????????+??????+tcp
            - app:cluster1-2048:2016-12-13T06:36:14.897Z(应用版本)

                    *app:cluster1-2048:2016-12-13T06:36:14.897Z$d5bcf6bf-5819-4e4d-ab02-fcc18af76e2e?
                    /cluster1-20480"
                    cpus		{?G?z??2*"
                    mem		0@2*"
                    disk		2*:
                    vclustecluster1:
                    typeswarmBZ2016-12-13T06:36:14.897Zh?qffffff??z	???????
                    >offlineregistry.dataman-inc.com:5000/library/blackicebird-2048 *
                    labelAPP_ID=cluster1-2048*
                    labelVCLUSTER=cluster10?
                    GROUP_ID1?

                    USER_ID1?
                    VCLUSTEcluster1??????????+??????+tcp
                         
            - internal:storage:version

                    internal:storage:version$33353334-3337-3635-2d36-3436332d363
            - task:cluster1-2048.726e0da5-c0fe-11e6-832f-a63ee8fdb797 (单个实例)

                    7task:cluster1-2048.726e0da5-c0fe-11e6-832f-a63ee8fdb797$64346133-3165-3865-2d32-6665622d3432?E
                    192.168.1.234??"26e0da5-c0fe-11e6-832f-a63ee8fdb797
                    vcluster*

                    cluster1(?????+0?????+B2016-12-13T06:36:14.897ZJ?D
                    4
                    2cluster1-2048.726e0da5-c0fe-11e6-832f-a63ee8fdb797?B[
                    {
                        "Id": "7e8f591d1c5dbe7d2b83f6111cca6441426d0ed9ad8a013d3b854fbdf6d83d7b",
                        "Created": "2016-12-13T06:36:16.065915942Z",
                        "Path": "nginx",
                        "Args": [
                            "-c",
                            "/etc/nginx/nginx.log.conf"
                        ],
                        "State": {
                            "Status": "running",
                            "Running": true,
                            "Paused": false,
                            "Restarting": false,
                            "OOMKilled": false,
                            "Dead": false,
                            "Pid": 1499,
                            "ExitCode": 0,
                            "Error": "",
                            "StartedAt": "2016-12-13T06:36:16.562848408Z",
                            "FinishedAt": "0001-01-01T00:00:00Z"
                        },
                    "Image": "sha256:960672cda0831825b6435d26011efb98fe0a4ceb44f08587c8b7ff6213e522d1",
                    "ResolvConfPath": "/data/docker/containers/7e8f591d1c5dbe7d2b83f6111cca6441426d0ed9ad8a013d3b854fbdf6d83d7b/resolv.conf",
                    "HostnamePath": "/data/docker/containers/7e8f591d1c5dbe7d2b83f6111cca6441426d0ed9ad8a013d3b854fbdf6d83d7b/hostname",
                    "HostsPath": "/data/docker/containers/7e8f591d1c5dbe7d2b83f6111cca6441426d0ed9ad8a013d3b854fbdf6d83d7b/hosts",
                    "LogPath": "/data/docker/containers/7e8f591d1c5dbe7d2b83f6111cca6441426d0ed9ad8a013d3b854fbdf6d83d7b/7e8f591d1c5dbe7d2b83f6111cca6441426d0ed9ad8a013d3b854fbdf6d83d7b-json.log",
                    "Name": "/mesos-97238888-a67b-45f6-9ae7-3d8bd97db41c-S1.1c70ce0f-4350-4c73-bc2e-eb6212a3c6dc",
                    "RestartCount": 0,
                    "Driver": "overlay",
                    "MountLabel": "",
                    "ProcessLabel": "",
                    "AppArmorProfile": "",
                    "ExecIDs": null,
                    "HostConfig": {
                        "Binds": [
                            "/data/mesos/slaves/97238888-a67b-45f6-9ae7-3d8bd97db41c-S1/frameworks/bd5969e1-d8d3-492c-abfc-3b2367465209-0000/executors/cluster1-2048.726e0da5-c0fe-11e6-832f-a63ee8fdb797/runs/1c70ce0f-4350-4c73-bc2e-eb6212a3c6dc:/mnt/mesos/sandbox"
                        ],
                        "ContainerIDFile": "",
                        "LogConfig": {
                            "Type": "json-file",
                            "Config": {}
                        },
                        "NetworkMode": "bridge",
                        "PortBindings": {},
                        "RestartPolicy": {
                            "Name": "no",
                            "MaximumRetryCount": 0
                        },
                        "AutoRemove": false,
                        "VolumeDriver": "",
                        "VolumesFrom": null,
                        "CapAdd": null,
                        "CapDrop": null,
                        "Dns": [],
                        "DnsOptions": [],
                        "DnsSearch": [],
                        "ExtraHosts": null,
                        "GroupAdd": null,
                        "IpcMode": "",
                        "Cgroup": "",
                        "Links": null,
                        "OomScoreAdj": 0,
                        "PidMode": "",
                        "Privileged": false,
                        "PublishAllPorts": false,
                        "ReadonlyRootfs": false,
                        "SecurityOpt": null,
                        "UTSMode": "",
                        "UsernsMode": "",
                        "ShmSize": 67108864,
                        "Runtime": "runc",
                        "ConsoleSize": [
                            0,
                            0
                        ],
                        "Isolation": "",
                        "CpuShares": 10,
                        "Memory": 33554432,
                        "CgroupParent": "",
                        "BlkioWeight": 0,
                        "BlkioWeightDevice": null,
                        "BlkioDeviceReadBps": null,
                        "BlkioDeviceWriteBps": null,
                        "BlkioDeviceReadIOps": null,
                        "BlkioDeviceWriteIOps": null,
                        "CpuPeriod": 0,
                        "CpuQuota": 0,
                        "CpusetCpus": "",
                        "CpusetMems": "",
                        "Devices": [],
                        "DiskQuota": 0,
                        "KernelMemory": 0,
                        "MemoryReservation": 0,
                        "MemorySwap": 67108864,
                        "MemorySwappiness": -1,
                        "OomKillDisable": false,
                        "PidsLimit": 0,
                        "Ulimits": null,
                        "CpuCount": 0,
                        "CpuPercent": 0,
                        "IOMaximumIOps": 0,
                        "IOMaximumBandwidth": 0
                    },
                    "GraphDriver": {
                        "Name": "overlay",
                        "Data": {
                            "LowerDir": "/data/docker/overlay/7d5b9cdb4867c1ea457d15a4f1d137e8ef0ded8e1821514b6948c54cfe72a352/root",
                            "MergedDir": "/data/docker/overlay/3cac539b9700a0601ae2f251b06fe4f05946c119986d383dd3af3e5b329f3b0c/merged",
                            "UpperDir": "/data/docker/overlay/3cac539b9700a0601ae2f251b06fe4f05946c119986d383dd3af3e5b329f3b0c/upper",
                            "WorkDir": "/data/docker/overlay/3cac539b9700a0601ae2f251b06fe4f05946c119986d383dd3af3e5b329f3b0c/work"
                        }
                    },
                    "Mounts": [
                        {
                            "Source": "/data/mesos/slaves/97238888-a67b-45f6-9ae7-3d8bd97db41c-S1/frameworks/bd5969e1-d8d3-492c-abfc-3b2367465209-0000/executors/cluster1-2048.726e0da5-c0fe-11e6-832f-a63ee8fdb797/runs/1c70ce0f-4350-4c73-bc2e-eb6212a3c6dc",
                            "Destination": "/mnt/mesos/sandbox",
                            "Mode": "",
                            "RW": true,
                            "Propagation": "rprivate"
                        }
                    ],
                    "Config": {
                        "Hostname": "7e8f591d1c5d",
                        "Domainname": "",
                        "User": "",
                        "AttachStdin": false,
                        "AttachStdout": true,
                        "AttachStderr": true,
                        "ExposedPorts": {
                            "80/tcp": {}
                        },
                        "Tty": false,
                        "OpenStdin": false,
                        "StdinOnce": false,
                        "Env": [
                            "MARATHON_APP_VERSION=2016-12-13T06:36:14.897Z",
                            "HOST=192.168.1.234",
                            "MARATHON_APP_RESOURCE_CPUS=0.01",
                            "MARATHON_APP_DOCKER_IMAGE=offlineregistry.dataman-inc.com:5000/library/blackicebird-2048",
                            "MARATHON_APP_LABEL_GROUP_ID=1",
                            "MESOS_TASK_ID=cluster1-2048.726e0da5-c0fe-11e6-832f-a63ee8fdb797",
                            "PORT=31307",
                            "MARATHON_APP_LABEL_VCLUSTER=cluster1",
                            "MARATHON_APP_RESOURCE_MEM=16.0",
                            "MARATHON_APP_LABEL_USER_ID=1",
                            "PORTS=31307",
                            "MARATHON_APP_RESOURCE_DISK=0.0",
                            "MARATHON_APP_LABELS=GROUP_ID USER_ID VCLUSTER",
                            "MARATHON_APP_ID=/cluster1-2048",
                            "PORT0=31307",
                            "MESOS_SANDBOX=/mnt/mesos/sandbox",
                            "MESOS_CONTAINER_NAME=mesos-97238888-a67b-45f6-9ae7-3d8bd97db41c-S1.1c70ce0f-4350-4c73-bc2e-eb6212a3c6dc",
                            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
                        ],
                        "Cmd": [
                            "nginx",
                            "-c",
                            "/etc/nginx/nginx.log.conf"
                        ],
                        "Image": "offlineregistry.dataman-inc.com:5000/library/blackicebird-2048",
                        "Volumes": null,
                        "WorkingDir": "",
                        "Entrypoint": null,
                        "OnBuild": null,
                        "Labels": {
                            "APP_ID": "cluster1-2048",
                            "VCLUSTER": "cluster1"
                        }
                    },
                    "NetworkSettings": {
                        "Bridge": "",
                        "SandboxID": "37778ff16144ece9efdfbdadb4039868ee60e9c1ed4abf308a8bf400e3757f05",
                        "HairpinMode": false,
                        "LinkLocalIPv6Address": "",
                        "LinkLocalIPv6PrefixLen": 0,
                        "Ports": {
                            "80/tcp": null
                        },
                        "SandboxKey": "/var/run/docker/netns/37778ff16144",
                        "SecondaryIPAddresses": null,
                        "SecondaryIPv6Addresses": null,
                        "EndpointID": "4d9d418e4aa7500f9bb24e28ca1e1d7b3fef6d2f3923bf167a96f682c74f5cce",
                        "Gateway": "172.17.0.1",
                        "GlobalIPv6Address": "",
                        "GlobalIPv6PrefixLen": 0,
                        "IPAddress": "172.17.0.22",
                        "IPPrefixLen": 16,
                        "IPv6Gateway": "",
                        "MacAddress": "02:42:ac:11:00:16",
                        "Networks": {
                            "bridge": {
                                "IPAMConfig": null,
                                "Links": null,
                                "Aliases": null,
                                "NetworkID": "887ec132b0f5d594f1673879c34b62bab3187ff75b9a0175ee5e0c0690020e73",
                                "EndpointID": "4d9d418e4aa7500f9bb24e28ca1e1d7b3fef6d2f3923bf167a96f682c74f5cce",
                                "Gateway": "172.17.0.1",
                                "IPAddress": "172.17.0.22",
                                "IPPrefixLen": 16,
                                "IPv6Gateway": "",
                                "GlobalIPv6Address": "",
                                "GlobalIPv6PrefixLen": 0,
                                "MacAddress": "02:42:ac:11:00:16"
                            }
                        }
                    }
                    }
                    ]
                    *)
                    '97238888-a67b-45f6-9ae7-3d8bd97db41c-S11C?/???A:4
                    2cluster1-2048.726e0da5-c0fe-11e6-832f-a63ee8fdb797HZ?
                                                      ??O????[3??
                                                                 b1
                    /
                     Docker.NetworkSettings.IPAddress
                                 172.17.0.22j
                    *
                    172.17.0.22R)
                    '97238888-a67b-45f6-9ae7-3d8bd97db41c-S1
                    
## Mesos 问题
### Marathon 无法选 leader 问题
- 故障原因 

    因为某种原因(服务器响应慢、网络设备故障等)导致zookeeper 对于 marathon 客户端响应超时，因 marathon 代码BUG导致在全部回收node 重新创建连接的时候创建了2次。 
- 故障现象： 
    - 打开marathon:8080 ui 

            HTTP ERROR: 503 

            Problem accessing /. Reason: 

            Could not determine the current leader 
    - 打开 marathon 日志   
        - Waiting for consistent leadership state. Are we leader?: false, leader: 表示无法选举 leader 
        - Candidate /marathon/leader/member_0000000002 waiting for the next leader election, current voting: [member_0000000001, member_0000000002] 读取到同步锁，会比真实的多，比如 master 是1个，那么 就是 2个。如果是master 是3个，那么就是4个。 
        - Client session timed out, have not heard from server in 7390ms for sessionid 0x15968cd89b90006, closing socket connection and attempting reconnect 产生原因，客户端超时
    - 解决方法
        - 使用 zkcli 进入 zookeeper 实例 

                docker exec -it dataman-zookeeper /bin/bash 
        - 查询挂点 

                ls /marathon/leader 
        - 删除挂点 

                rmr /marathon/leader 
        - 退出服务
        
                quit 
        - 服务恢复                                  