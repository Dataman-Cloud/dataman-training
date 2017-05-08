# Mesos-Master-Flag
    文档信息
    创建人 庞铮
    邮件地址 zpang@dataman-inc.com
    建立时间 2016年6月1号
    更新时间 2016年6月1号

### [Mesos-master 参数](http://mesos.apache.org/documentation/latest/configuration/)
查看参数方法:`http://mesos-master-IP:5050/state.json`中的`flags`里的内容

- --acls

        用于设置认证，一般是 JSON 格式的 ACLs 的字符串或者文件。
        dataman设置:
        无
- --allocation_interval=VALUE

        设置（成批）分配资源时的时间间隔。(比如 500ms, 1sec, 等). (默认:1secs)
        dataman设置:        
        allocation_interval: "1secs",
        
- --allocator=VALUE        

        分配机制，默认为 HierarchicalDRF
        dataman设置:
        allocator: "HierarchicalDRF",
        
- --[no-]authenticate

        是否允许非认证过的 framework 注册,如果是 true，只有被认证的才能注册(默认:false)       
        dataman设置:
        authenticate: "false",
- --[no-]authenticate_slaves

        是否允许非认证过的 slave 注册,如果是 true，只有被认证的才能注册(默认:false)        
        dataman设置:
        authenticate_slaves: "false",
- --authenticators=VALUE

        对 framework 或 salves 进行认证时的实现机制(默认:crammd5)
        dataman设置:
        authenticators: "crammd5",
        
- --cluster=VALUE

        设定在web界面里显示的集群别
        dataman设置:        
        cluster: "dataman-mesos",
- --framework_sorter=VALUE
        
        给定 framework 之间的资源分配策略,值和user_allocator一样。 (默认:drf)
        framework_sorter: "drf",
- --[no-]help

        输出帮助信息
        dataman设置:
        help: "false",
- --hostname=VALUE
    
        设置被控的主机名,若没有设置,则会用master的IP地址来绑定主机名
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
- --[no-]log_auto_initialize

        设置是否在注册时自动初始化 “同步日志”，若设为 flase，则以后首次使用的时候，需要手动创建。 (默认:true)
        log_auto_initialize: "true",
- --log_dir=VALUE

        设置master日志文件存储地址，默认方式下，不生成日志文件。
        dataman设置:        
        log_dir: "/data/logs",
- --logbufsecs=VALUE
        
        buffer 多少秒的日志，然后写入本地（默认:０）        
        dataman设置:
        logbufsecs: "0",
- --logging_level=VALUE
    
        设置记录的日志信息级别，可用的设置：'INFO'（一般信息）, 'WARNING'（警告）, 'ERROR'（出错）; 若使用 quiet 参数，将只会作用到　log_dir（若设置了的话）里的日志。（默认：INFO）
        dataman设置:
        logging_level: "WARNING",
- --max_slave_ping_timeouts=VALUE
        
        
        设置呼叫 slave 无回应的超时最大次数。超时的slave将被移除（默认:5次）
        dataman设置:
        max_slave_ping_timeouts: "5",       
- --slave_ping_timeout=VALUE

        设置master呼叫slave的超时时间。（默认:15secs）        
        dataman设置:        
        slave_ping_timeout: "15secs",
- --port=VALUE
    
        设置监听的端口
        dataman设置:    
        port: "5050",
- --[no-]quiet

        禁止输出日志到 sterr (默认:false)，只作用于 log_dir（若设置了的话）里的日志。 
        dataman设置:   
        quiet: "false",
- --quorum=VALUE

        master选举过半数的数量
        dataman设置:
        quorum: "2",
- --recovery_slave_removal_limit=VALUE

        限制注册表恢复后可以移除或停止的 slave 数目，超出后 master 会失败(默认:100%).此项设置可用于 当集群发生大规模被控崩溃的时候，让人工干预。
        dataman设置:
        recovery_slave_removal_limit: "100%",
- --registry=VALUE

        注册表的持久化策略，(默认:replicated_log)，还可以为 in_memory,生产必须默认属性
        dataman设置:
        registry: "replicated_log",
- --registry_fetch_timeout=VALUE

        获取注册信息的超时时间。(默认:1mins)
        dataman设置:        
        registry_fetch_timeout: "1mins",
- --registry_store_timeout=VALUE

        存储注册信息的超时时间。(默认:5secs)
        dataman设置:
        registry_store_timeout: "5secs",
- --[no-]registry_stric

        设置是否验证注册信息。若设置为 false，则代表接受所有的注册，包括：被控加入申请、被控读取申请、移除被控申请。这样设置不需系统重构，从而可令运行中的集群 保持持续运转。(实验性质，生产不可用) (默认:false)       
        dataman设置:
        registry_strict: "false",       
-  --[no-]root_submissions
        
        root 是否可以提交 frameworks。(默认:true)
        dataman设置:
        root_submissions: "true",
- --slave_reregister_timeout=VALUE

        新的 lead master 节点选举出来后，多久之内所有的 slave 需要注册，超时的 salve 将被移除并关闭，默认为 10mins
        dataman设置:
        slave_reregister_timeout: "10mins",
- --user_sorter=VALUE

        在用户之间分配资源的策略，默认为 drf
        dataman设置:        
        user_sorter: "drf",        
-  --[no-]version

        显示版本并退出（默认:false）        
        dataman设置:
        version: "false",        
- --webui_dir=VALUE        

        管理页面的网页文件的目录(默认：/usr/local/share/mesos/webui)
        dataman设置:
        webui_dir: "/usr/share/mesos/webui",
-  --work_dir=VALUE

        master 的工作目录
        dataman设置:        
        work_dir: "/data/mesos"
- --zk=VALUE        
        
        zookeepr 的接口地址，也可以用文件
        dataman设置:
        zk: "zk://10.3.10.3:2181,10.3.10.4:2181,10.3.10.44:2181/mesos",
-  --zk_session_timeout=VALUE

        设置ZooKeeper会话超时时间。(默认:10secs)
        dataman设置:         
        zk_session_timeout: "10secs"