# Mesos-Slave-Flag
    文档信息
    创建人 庞铮
    邮件地址 zpang@dataman-inc.com
    建立时间 2016年6月1号
    更新时间 2016年6月1号
### [Mesos-slave 参数](http://mesos.apache.org/documentation/latest/configuration/)
查看参数方法:`http://mesos-slaveIP:5051/state.json`中的`flags`里的内容

- --appc_store_dir=VALUE

        dataman设置:
        无
- --attributes=VALUE 
    
        机器属性,rack:2 or rack:2;u:1
        dataman设置:   
        attributes: "ip:10.3.10.44",
- --authenticatee=VALUE
        
        跟 master 的认证机制 (默认crammd5) 或者使用 --modules 加载备用模块
        dataman设置:
        authenticatee: "crammd5",
- --[no]-cgroups_cpu_enable_pids_and_tids_count
        
        设置使用 cgroup 来计算容器中的进程和线程的数量。(默认:false)    
        dataman设置:
        cgroups_cpu_enable_pids_and_tids_count: "false",
- --[no]-cgroups_enable_cfs

        设置使用 cgroup 的CFS带宽限额来限制CPU使用量(默认:false)
        dataman设置:    
        cgroups_enable_cfs: "false",
- --cgroups_hierarchy=VALUE

        设置cgroup的根路径(默认: /sys/fs/cgroup)
        dataman设置:     
        cgroups_hierarchy: "/sys/fs/cgroup",
- --[no]-cgroups_limit_swap
        
        设置使用cgroup限制容器对于swap的使用(默认:false)
        dataman设置:    
        cgroups_limit_swap: "false",
- --cgroups_net_cls_primary_handle

         
        dataman设置:
        无
- --cgroups_net_cls_secondary_handles

         
        dataman设置:
        无        
- --cgroups_root=VALUE
        
        设置 cgroups 的根名称(默认:mesos)
        dataman设置:
        cgroups_root: "mesos",
- --container_disk_watch_interval=VALUE    
        
        容器进行硬盘配额查询的时间间隔(默认:15secs)
        dataman设置:    
        container_disk_watch_interval: "15secs",
- --container_logger=VALUE

        dataman设置:
        无
- --containerizer_path=VALUE

        采用外部隔离机制（--isolation=external）时候，外部容器机制执行文件路径
        dataman设置:
        无
- --containerizers=VALUE

        可用的容器实现机制，包括 mesos、external、docker(默认:mesos) 
        dataman设置:                         
        containerizers: "docker,mesos",
- --credential=VALUE

        设定证书的文件路径，证书列表可以是一个文本文件,也可以是一个json格式文档。    
        dataman设置:
        无
- --default_container_image=VALUE

        采用外部容器机制时，任务缺省使用的镜像
        dataman设置:
        无
- --default_container_info=VALUE

        设定默认容器信息的路径，信息保存在一个json格式文档中，当没有指定容器信息的时候，ExecutorInfo将使用这个默认值。
        dataman设置:
        无
- --default_role=VALUE

        谁定资源缺省分配的默认角色，当设置了--resources但没设定角色时，或者没设定--resources的时候，都将使用这个默认角色。(默认:*)
        dataman设置:       
        default_role: "*",
- --disk_watch_interval=VALUE
 
        设置监测磁盘空间使用状况的周期时间。 (默认:1mins)
        dataman设置:    
        disk_watch_interval: "1mins",
- --docker=VALUE

        设置docker 执行文件的路径。(默认:docker)
        dataman设置:
        docker: "docker",
- --[no-]docker_kill_orphans

        设置让 docker 管理器关闭已经空载的容器。当需要在同一个系统里面运行多个docker slave时，需要设置为 false，避免移除被其他slave使用的容器。同时，也需要给slave开启运行监测，以使得slave的识别名能够重用，否则slave上的docker任务在重启的时候将无法被清空。(默认:true)
        dataman设置:
        docker_kill_orphans: "true",
- --docker_remove_delay=VALUE

        移除 docker 前等待的时间(如 3 天，2 周 等)。(默认:6 小时)
        dataman设置:               
        docker_remove_delay: "1mins",
- --docker_sandbox_directory=VALUE
        
        沙盒映射到容器中的绝对路径(默认: /mnt/mesos/sandbox)
        dataman设置:
        docker_sandbox_directory: "/mnt/mesos/sandbox",       

- --docker_socket=VALUE

        设置将被 docker 执行器加载的 UNIX_socket 路径，以使 docker 命令行界面里可操作 docker 守护进程。这个路径必须是被控的 docker 镜像里用的。(默认:/var/run/docker.sock)       
        dataman设置:
        docker_socket: "/var/run/docker.sock",
- --docker_stop_timeout=VALUE

        设置docker停用到杀死一个实例的等待时间，默认0秒表示停止后立即杀死。 (默认:0ns)
        dataman设置:         
        docker_stop_timeout: "0ns",
- --docker_store_dir=VALUE

        dataman设置:
        无 
- --[no-]enforce_container_disk_quota

        是否启用容器配额限制，(默认:false)
        dataman设置:        
        enforce_container_disk_quota: "false",
- --executor_registration_timeout=VALUE        
        
        设置执行器向mesos-slave注册的超时时间，超过设定时间则被认为是挂起并被关闭(默认:5mins)
        dataman设置: 
        executor_registration_timeout: "5mins",
- --executor_shutdown_grace_period=VALUE        
        
        设置执行器关闭前的等待时间(默认5秒： 5secs)
        dataman设置:
        executor_shutdown_grace_period: "5secs",
- --fetcher_cache_dir=VALUE

        fetcher cache 文件存放目录，(默认:/tmp/mesos/fetch)    
        dataman设置:        
        fetcher_cache_dir: "/tmp/mesos/fetch",
- --fetcher_cache_size=VALUE

        fetcher 的 cache 大小(默认:2 GB)
        dataman设置:
        fetcher_cache_size: "2GB",        
- --frameworks_home=VALUE

        设置frameworks_home 路径，用于作为执行器URI的前缀。(默认:无)    
        dataman设置:
        frameworks_home: "",
- --gc_delay=VALUE

        多久清理一次执行应用目录，(默认:1weeks)
        dataman设置:        
        gc_delay: "1weeks",
- --gc_disk_headroom=VALUE

        设置执行器目录的纯空置率（就是完全没被用到的磁盘空间比例，）。纯空置率用于计算目录存续期，存续期计算方式:gc_delay * max(0.0, (1.0 - gc_disk_headroom - disk usage))
        dataman设置:        
        gc_disk_headroom: "0.1",
- --hadoop_home=VALUE
                
        hadoop 安装目录，默认为空，会自动查找 HADOOP_HOME 或者从系统路径中查找
        dataman设置:
        hadoop_home: "",
- --[no-]help

        输出帮助信息
        dataman设置:
        help: "false",
- --hostname=VALUE
    
        设置被控的主机名,若没有设置,则会用slve的IP地址来绑定主机名
        dataman设置:
        hostname: "10.3.10.44",
- --[no-]initialize_driver_logging
        
        设置是否初始化 google 日志调度器或执行器驱动，将分别记录不和master/slave日志放在一起。（默认：true） 
        dataman设置:    
        initialize_driver_logging: "true",
- --ip=VALUE

        程序监听的 IP 地址. 与 --ip_discovery_command 互斥. (master默认端口5050，slave默认端口5051)
        dataman设置:        
        ip: "10.3.10.44",
- --isolation=VALUE

        隔离机制，例如 posix/cpu,posix/mem（默认:cgroups/cpu,cgroups/mem)(可用 --with-network-isolator 开启网络设置或 'external', 或可用载入一个 --modules 隔离模块。注：只对Mesos容器管理器有效）        
        dataman设置:
        isolation: "cgroups/cpu,cgroups/mem",
- --launcher_dir=VALUE

        设置 Mesos 可执行程序路径(默认:/usr/local/lib/mesos)
        dataman设置:
        launcher_dir: "/usr/libexec/mesos",
- --log_dir=VALUE

        设置slave日志文件存储地址，默认方式下，不生成日志文件。
        dataman设置:        
        log_dir: "/data/logs",
- --logbufsecs=VALUE
        
        buffer 多少秒的日志，然后写入本地（默认:０）        
        dataman设置:
        logbufsecs: "0",
- --logging_level=VALUE
    
        设置记录的日志信息级别，可用的设置：'INFO'（一般信息）, 'WARNING'（警告）, 'ERROR'（出错）; 若使用 quiet 参数，将只会作用到　log_dir（若设置了的话）里的日志。（默认级别：INFO）
        dataman设置:
        logging_level: "WARNING",
- --master=VALUE
        
        设置连接主控的方式。有三种方式:
        host:port 
        zk://host1:port1,host2:port2,.../path 
        zk://username:password@host1:port1,host2:port2,.../path
        并把上面描述放到文件里设置第三种:
        file:///path/to/file
        dataman设置:
        master: "zk://10.3.10.3:2181,10.3.10.4:2181,10.3.10.44:2181/mesos",

- --oversubscribed_resources_interval=VALUE

        设置资源汇报间隔时间：SLAVE会把目前估算的已用(也就是可被回收)及还没用的过剩资源总量，定期告诉主控。(默认:15secs)   
        dataman设置:
        oversubscribed_resources_interval: "15secs",
- --perf_duration=VALUE
    
        设置一个perf采样统计周期，这个值必须低于perf_interval。(默认:10secs)
        dataman设置:
        perf_duration: "10secs",
- --perf_interval=VALUE        
    
        设置 perf 统计周期间的间隔，系统会根据这个间隔时间，获取perf的数据，并把最近的数据返回而不是展现。可见，这个perf_interval和资源监控间隔是不一样的。(默认:1mins)    
        dataman设置:
        perf_interval: "1mins",
- --port=VALUE
    
        设置监听的端口
        dataman设置:    
        port: "5051",
- --qos_correction_interval_min=VALUE
    
        设置根据 QoS 控制器要求调整任务的的最小间隔周期。QoS 控制器会根据监测到的任务运行性能进行调整，slave 会轮询并据此调整。(默认:0ns)    
        qos_correction_interval_min: "0ns",
- --[no-]quiet

        禁止输出日志到 sterr (默认:false)，只作用于 log_dir（若设置了的话）里的日志。 
        dataman设置:   
        quiet: "false",
- --recover=VALUE
        
        恢复更新状态后是否与老的 executors 重新连接。recover 可用的值有
        reconnect：与老的还存活的 executors 重新连接。
        cleanup：杀掉所有的老的还存活的 executors 并退出，当 slave 不兼容或 executor 更新时，使用这个选项。
        dataman设置:    
        recover: "reconnect",
- --recovery_timeout=VALUE

        slave 恢复超时时间，超时则所有相关的执行器都将自行退出，(默认:15mins)        
        dataman设置:
        recovery_timeout: "15mins",
- --registration_backoff_factor=VALUE        
        
        slave 跟一个新的 master 进行注册时候的重试时间间隔算法的因子，(默认: 1secs)，采用随机指数算法,从[0, b]中任选，最长 1mins
        dataman设置:
        registration_backoff_factor: "1secs",
- --resource_monitoring_interval=VALUE
        
        周期性监测执行应用资源使用情况的间隔，(默认:1secs)
        dataman设置:        
        resource_monitoring_interval: "1secs",
- --[no-]revocable_cpu_low_priority

        通过 revocable CPU 以相对低的优先级运行 containers 。目前只支持 cgroups/cpu isolator( 默认: true )       
        dataman设置:
        revocable_cpu_low_priority: "true",
    
- --[no-]strict
        
        如果是true,只恢复没有任何出错时的状态.反之，所有错误将被忽略，尽可能的恢复最近的状态。
        dataman设置:
        strict: "true",
- --[no-]switch_user        
    
        用提交任务的用户身份来运行，(需要 setuid 操作权限) (默认:true)
        dataman设置:
        switch_user: "true",

-  --[no-]version

        显示版本并退出（默认:false）        
        dataman设置:
        version: "false",
-  --work_dir=VALUE

        slave 的工作目录
        dataman设置:        
        work_dir: "/data/mesos"
        
