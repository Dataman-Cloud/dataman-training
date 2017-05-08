# Zookeeper文档
## 1 Zookeeper服务说明
### 1.1 概要
ZooKeeper是一个分布式开源框架，提供了协调分布式应用的基本服务，它向外部应用暴露一组通用服务——分布式同步（Distributed Synchronization）、命名服务（Naming Service）、集群维护（Group Maintenance）等，简化分布式应用协调及其管理的难度，提供高性能的分布式服务。
ZooKeeper本身可以以Standalone模式安装运行，不过它的长处在于通过分布式ZooKeeper集群（一个Leader，多个Follower），基于一定的策略来保证ZooKeeper集群的稳定性和可用性，从而实现分布式应用的可靠性。

### 1.2 详细说明
####  1.2.1 特性
集群中只要有过半的机器是正常工作的，那么整个集群对外就是可用的过半存活即可用。正是基于这个特性，建议是将ZK集群的机器数量控制为奇数较为合适。为什么选择奇数台机器，假如是4台机器构成的ZK集群，那么只能够允许集群中有一个机器down掉，因为如果down掉2台，那么只剩下2台机器，显然没有过半。而如果是5台机器的集群，那么就能够对2台机器down掉的情况进行容灾了。

####  1.2.2 性能
设置Java heap 大小。避免内存与磁盘空间的交换，能够大大提升ZK的性能，设置合理的heap大小则能有效避免此类空间交换的触发。在正式发布上线之前，建议是针对使用场景进行一些压力测试，确保正常运行后内存的使用不会触发此类交换。通常在一个物理内存为4G的机器上，最多设置-Xmx为3G。(注意这里如果zookeeper时非独立服务器，内存规划需要严谨)

#### 1.2.3 ZNODER结构说明
ZooKeeper 有一个类似于文件系统的数据模型，由 znodes 组成。可以将 znodes（ZooKeeper 数据节点）视为类似 UNIX 的传统系统中的文件，但它们可以有子节点。另一种方式是将它们视为目录，它们可以有与其相关的数据。每个这些目录都被称为一个 znode。

znode 层次结构被存储在每个 ZooKeeper 服务器的内存中。这实现了对来自客户端的读取操作的可扩展的快速响应。每个 ZooKeeper 服务器还在磁盘上维护了一个事务日志，记录所有的写入请求。ZooKeeper 服务器在返回一个成功的响应之前必须将事务同步到磁盘，所以事务日志也是 ZooKeeper 中对性能最重要的组成部分。
结构如下:

    /--
        mesos--
            mesos1
            mesos2
        marathon--
            marathon1
            marathon2

#### 1.2.4 zookeeper的ZNODE运作机制
读：
当客户端请求读取特定 znode 的内容时，读取操作是在客户端所连接的服务器上进行的。因此，由于只涉及集合体中的一个服务器，所以读取是快速和可扩展的。

写：
当客户端发出一个写入请求时，所连接的服务器会将请求传递给领导者。此领导者对集合体的所有节点发出相同的写入请求。如果严格意义上的多数节点（也被称为法定数量（quorum））成功响应该写入请求，那么写入请求被视为已成功完成。然后，一个成功的返回代码会返回给发起写入请求的客户端。如果集合体中的可用节点数量未达到法定数量，那么 ZooKeeper 服务将不起作用。

如何选择集群节点:
简单的说，Zookeeper 客户端将所有 Server 保存在一个 List 中，然后随机打乱，并且形成一个环，具体使用的时候，从0号位开始一个一个使用。
## 2 支持平台
### 2.1 对比表格
平 台      | 运行client| 运行server       | 开发环境| 生产环境
:---------:| :------: | :--------------:|:------:|:-----------:
GNU/Linux  | √        | √               | √      | √
Sun Solaris| √        | √               | √      | √
FreeBSD    | √        | ⅹ，对nio的支持不好| √      | √
Win32      | √        | √               | √      | ⅹ
MacOSX     | √        | √               | √      | ⅹ
### 2.2说明    
运行client是指作为客户端与 server 进行数据通信，而运行 server 是指将ZK作为服务器部署运行

## 3 安装
### 3.1 ubuntu14.04 安装
    # Key
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF
    DISTRO=$(lsb_release -is | tr '[:upper:]' '[:lower:]')
    CODENAME=$(lsb_release -cs)
    
    # Add the repository
    echo "deb http://repos.mesosphere.io/${DISTRO} ${CODENAME} main" |
    sudo tee /etc/apt/sources.list.d/mesosphere.list
    
    # Install packages
    sudo apt-get update && apt-get -y install zookeeper zookeeper-bin zookeeperd
### 3.2 centos 安装

    yum install zookeeper-server
### 3.3 容器化

    FROM centos7
    MAINTAINER jyliu <jyliu@dataman-inc.colm>
    
    # install jdk
    # install epel
    RUN  yum install -y epel-release
    # install jdk
    RUN yum install -y java-1.8.0-openjdk

    # install zookeeper
    RUN yum install -y wget && \
        curl -o - http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz|tar -zxf - -C /opt \
        && ln -s  /opt/zookeeper-3.4.6 /usr/local/zookeeper

    # create URI dir
    RUN mkdir -p /data/run && \
        mkdir -p /data/logs && \
    # create zookeeper dir
        mkdir -p /data/zookeeper/zkLog && \
        mkdir -p /data/zookeeper/snapshot && \
    # setting zoo.cfg
        mv /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo.cfg

    RUN yum clean all

    ENTRYPOINT ["/usr/local/zookeeper/bin/zkServer.sh"]
    CMD ["start-foreground"]
## 4 配置
### 4.1 基础配置
核心参数配置文件(单一文件多服共享)
    
    vi /usr/local/zookeeper/conf/zoo.cfg
    dataDir=/usr/local/zookeeper/data
    dataLogDir=/data/zookeeper/logs
    clientPort=2181
    tickTime=2000
    initLimit=5
    syncLimit=2
    server.1=HDP245:2888:3888
    server.2=HDP246:2888:3888
    server.3=HDP247:2888:3888
参数说明
    
    dataDir：数据目录,myid – 这个文件只包含一个数字，和server id对应。snapshot. - 按zxid先后顺序的生成的数据快照。
    
    dataLogDir：事务日志目录含了所有ZK的事务日志
    
    clientPort：客户端连接端口
    
    tickTime：Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。单位毫秒
    
    initLimit：Zookeeper的Leader 接受客户端（Follower）初始化连接时最长能忍受多少心跳时间间隔数。当已超过 5个心跳的时间（也就是tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒
    
    syncLimit：表示 Leader 与 Follower 之间发送消息时请求和应答时间长度，最长不能超过多少个tickTime 的时间长度，总的时间长度就是 2*2000=4 秒。
    
    server.A=B：C：D：
    A 是一个数字，表示这个是第几号服务器；
    B 是这个服务器的 ip 地址；
    C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；
    D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。
    如果是伪集群的配置方式，由于 B 都是一样，所以不同的Zookeeper实例通信端口号不能一样，所以要给它们分配不同的端口号。

### 4.2 参数详细说明
    clientPort  
    #客户端连接server端口，一般设置为2181。
    
    dataDir     
    #存储快照文件snapshot的目录。默认情况下，事务日志也会存储在这里。建议同时配置参数dataLogDir, 事务日志的写性能直接影响zk性能。
    
    dataLogDir  
    #事务日志输出目录。尽量给事务日志的输出配置单独的磁盘或是挂载点，这将极大的提升ZK性能。（No Java system property）

    autopurge.purgeInterval
    #ZK提供了自动清理事务日志和快照文件的功能，这个参数指定了清理频率，单位是小时，需要配置一个1或更大的整数，默认是0，表示不开启自动清理功能。(No Java system property) New in 3.4.0

    autopurge.snapRetainCount   
    #这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。(No Java system property) New in 3.4.0

    tickTime    
    #ZK中的一个时间单元。ZK中所有时间都是以这个时间单元为基础，进行整数倍配置的。例如，session的最小超时时间是2*tickTime。

    initLimit       
    #Follower在启动过程中，会从Leader同步所有最新数据，然后确定能够对外服务的起始状态。Leader允许F在initLimit时间内完成这个工作。通常情况下，不用太在意这个参数的设置。如果ZK集群的数据量确实很大了，F在启动的时候，从Leader上同步数据的时间也会相应变长，因此在这种情况下，有必要适当调大这个参数了。(No Java system property)

    syncLimit	
    在运行过程中，Leader负责与ZK集群中所有机器进行通信，例如通过一些心跳检测机制，来检测机器的存活状态。如果L发出心跳包在syncLimit之后，还没有从F那里收到响应，那么就认为这个F已经不在线了。注意：不要把这个参数设置得过大，否则可能会掩盖一些问题。(No Java system property)

    server.x=[hostname]:nnnnn[:nnnnn]	
    这里的x是一个数字，与myid文件中的id是一致的。右边可以配置两个端口，第一个端口用于F和L之间的数据同步和其它通信，第二个端口用于Leader选举过程中投票通信。(No Java system property)

    globalOutstandingLimit	
    #最大请求堆积数。默认是1000。ZK运行的时候， 尽管server已经没有空闲来处理更多的客户端请求了，但是还是允许客户端将请求提交到服务器上来，以提高吞吐性能。为了防止Server内存溢出，这个请求堆积数还是需要限制下的。(Java system property:zookeeper.globalOutstandingLimit.)

    preAllocSize    
    #预先开辟磁盘空间，用于后续写入事务日志。默认是64M，每个事务日志大小就是64M。如果ZK的快照频率较大的话，建议适当减小这个参数。(Java system property:zookeeper.preAllocSize)

    snapCount       
    #每进行snapCount次事务日志输出后，触发一次快照(snapshot), 此时，ZK会生成一个snapshot.*文件，同时创建一个新的事务日志文件log.*。默认是100000.（代码实现中会进行一定的随机数处理，避免所有服务器在同一时间进行快照而影响性能）(Java system property:zookeeper.snapCount)

    traceFile       
    #用于记录所有请求的log，一般调试过程中可以使用，但是生产环境不建议使用，会严重影响性能。(Java system property:?requestTraceFile)

    maxClientCnxns  
    #单个客户端与单台服务器之间的连接数的限制，是ip级别的，默认是60（3.4.0开始，默认值调整为60，之前的默认值是10），如果设置为0，那么表明不作任何限制。请注意这个限制的使用范围，仅仅是单台客户端机器与单台ZK服务器之间的连接数限制，不是针对指定客户端IP，不是ZK集群的连接数限制，不是单台ZK对所有客户端的连接数限制。指定客户端IP的限制策略，这里有一个patch，可以尝试一下：http://rdc.taobao.com/team/jm/archives/1334（No Java system property）

    clientPortAddress   
    #对于多网卡的机器，可以为每个IP指定不同的监听端口。默认情况是所有IP都监听clientPort指定的端口。New in 3.3.0

    minSessionTimeoutmaxSessionTimeout  
    #Session超时限制，如果客户端设置的超时时间不在这个范围，那么会被强制设置为最大或最小时间。默认的Session超时时间是在2 * tickTime ~ 20 * tickTime这个范围 New in 3.3.0

    fsync.warningthresholdms    
    #事务日志输出时，如果调用fsync方法超过指定的超时时间，那么会在日志中输出警告信息。默认是1000ms。(Java system property:fsync.warningthresholdms) New in 3.3.4

    leaderServes        
    #默认情况下，Leader是会接受客户端连接，并提供正常的读写服务。但是，如果让Leader专注于集群中机器的协调，那么可以将这个参数设置为no，这样一来，会大大提高写操作的性能。(Java system property: zookeeper.leaderServes)。

    group.x=nnnnn[:nnnnn]weight.x=nnnnn 
    #对机器分组和权重设置(No Java system property)

    cnxTimeout          
    #Leader选举过程中，打开一次连接的超时时间，默认是5s。(Java system property: zookeeper.cnxTimeout)

    zookeeper.DigestAuthenticationProvider.superDigest  
    #ZK权限设置相关，具体参见《使用super身份对有权限的节点进行操作》 和 《ZooKeeper权限控制》

    skipACL             
    #对所有客户端请求都不作ACL检查。如果之前节点上设置有权限限制，一旦服务器上打开这个开头，那么也将失效。(Java system property:zookeeper.skipACL)

    forceSync           
    #这个参数确定了是否需要在事务日志提交的时候调用FileChannel.force来保证数据完全同步到磁盘。(Java system property:zookeeper.forceSync)

    jute.maxbuffer      
    #每个节点最大数据量，是默认是1M。这个限制必须在server和client端都进行设置才会生效。(Java system property:jute.maxbuffer)

    myid文件
    每台机器的dataDir下，新建一个myid文件，里面存放一个数字，用来标识当前主机。
    zookeeper@zk01:$ echo "1" >> ~/usr/local/zookeeper/data/myid
    zookeeper@zk02:$ echo "2" >> ~/usr/local/zookeeper/data/myid
    zookeeper@zk03:$ echo "3" >> ~/usr/local/zookeeper/data/myid
    因为3个节点的启动是有顺序的，所以在陆续启动三个节点的时候，前面先启动的节点连接未启动的节点的时候会报出一些错误。可以忽略。

## 5 服务操作
### 启动
    /opt/zookeeper/bin/zkServer.sh start
### 停止
    /opt/zookeeper/bin/zkServer.sh stop
### 查看状态
    /opt/zookeeper/bin/zkServer.sh status
### 前端运行
    /opt/zookeeper/bin/zkServer.sh start-foreground

客户端连接 ZooKeeper 集群
找一台机器，解压 zookeeper 压缩包不用配置，就可以使用java客户端连接ZooKeeper集群中的任意一台服务器了。
    
    $ ./bin/zkCli.sh -server zk01:2181
    $ ./bin/zkCli.sh -server zk01:2181
    $ ./bin/zkCli.sh -server zk01:2181
连接上以后，就可以执行各种命令，使用help查看帮助。
### zookeeper四字命令
命令：echo command|nc localhost 2181 && echo command | curl -s telnet://localhost:2181
    
    conf：输出server的详细配置信息
    cons：输出指定server上所有客户端连接的详细信息，包括客户端IP，会话ID等
    crst：功能性命令。重置所有连接的统计信息
    dump：这个命令针对Leader执行，用于输出所有等待队列中的会话和临时节点的信息
    envi：用于输出server的环境变量。包括操作系统环境和Java环境。
    ruok：用于测试server是否处于无错状态。如果正常，则返回“imok”,否则没有任何响应。
    stat：输出server简要状态和连接的客户端信息。
    srvr：列出server端详情
    srst：重置server的统计信息
    wchs：列出所有watcher信息概要信息，数量等
    wchc：列出所有watcher信息，以watcher的session为归组单元排列，列出该会话订阅了哪些path
    wchp：列出所有watcher信息，以watcher的path为归组单元排列，列出该path被哪些会话订阅着
    mntr：输出一些ZK运行时信息，通过对这些返回结果的解析，可以达到监控的效果


## 6 运维常识：
### 6.1 集群维度
在上面提到的“过半存活即可用”特性中。基于这个特性，如果想搭建一个能够允许F台机器down掉的集群，那么就要部署一个由2xF+1 台机器构成的ZK集群。因此，一个由3台机器构成的ZK集群，能够在down掉一台机器后依然正常工作，而5台机器的集群，能够对两台机器down掉的情况容灾。ZK集群通常设计部署成奇数台机器。为了尽可能地提高ZK集群的可用性，应该尽量避免一大批机器同时down掉的风险，换句话说，最好能够为每台机器配置互相独立的硬件环境。举个例子，如果大部分的机器都挂在同一个交换机上，那么这个交换机一旦出现问题，将会对整个集群的服务造成严重的影响。其它类似的还有诸如：供电线路，散热系统等。其实在真正的实践过程中，如果条件允许，通常都建议尝试跨机房部署。毕竟多个机房同时发生故障的机率还是挺小的。

### 6.2 单机维度
对于ZK来说，如果在运行过程中，需要和其它应用程序来竞争磁盘，CPU，网络或是内存资源的话，那么整体性能将会大打折扣。
首先来看看磁盘对于ZK性能的影响。客户端对ZK的更新操作都是永久的，不可回退的，也就是说，一旦客户端收到一个来自server操作成功的响应，那么这个变更就永久生效了。为做到这点，ZK会将每次更新操作以事务日志的形式写入磁盘，写入成功后才会给予客户端响应。明白这点之后，你就会明白磁盘的吞吐性能对于ZK的影响了，磁盘写入速度制约着ZK每个更新操作的响应。为了尽量减少ZK在读写磁盘上的性能损失，不仿试试下面说的几点：

使用单独的磁盘作为事务日志的输出（比如我们这里的ZK集群，使用单独的挂载点用于事务日志的输出）。事务日志的写性能确实对ZK性能，尤其是更新操作的性能影响很大，所以想办法搞到一个单独的磁盘吧！ZK的事务日志输出是一个顺序写文件的过程，本身性能是很高的，所以尽量保证不要和其它随机写的应用程序共享一块磁盘，尽量避免对磁盘的竞争。
尽量避免内存与磁盘空间的交换。如果希望ZK能够提供完全实时的服务的话，那么基本是不允许操作系统触发此类swap的。因此在分配JVM堆大小的时候一定要非常小心。

### 6.3 清理日志
dataDir目录指定了ZK的数据目录，用于存储ZK的快照文件（snapshot）。默认情况下，ZK的事务日志也会存储在这个目录中。在完成若干次事务日志之后（在ZK中，凡是对数据有更新的操作，比如创建节点，删除节点或是对节点数据内容进行更新等，都会记录事务日志），ZK会触发一次快照（snapshot），将当前server上所有节点的状态以快照文件的形式dump到磁盘上去，即snapshot文件。这里的若干次事务日志是可以配置的，默认是100000，参数snapCount.默认ZK是不会自动清理快照和事务日志。
ZK使用log4j作为日志系统，conf目录中有一份默认的log4j配置文件，这个配置文件中还没有开启ROLLINGFILE文件输出，配置下即可。log4j的详细介绍，http://logging.apache.org/log4j/1.2/manual.html#defaultInit
#### 6.3.1 清理脚本:
    
    #!/bin/bash
    #snapshot file dir
    dataDir=/home/yinshi.nc/test/zk_data/version-2
    #tran log dir
    dataLogDir=/home/yinshi.nc/test/zk_log/version-2
    #zk log dir
    logDir=/home/yinshi.nc/test/logs
    #Leave 66 files
    count=66
    count=$[$count+1]
    ls -t $dataLogDir/log.* | tail -n +$count | xargs rm -f
    ls -t $dataDir/snapshot.* | tail -n +$count | xargs rm -f
    ls -t $logDir/zookeeper.log.* | tail -n +$count | xargs rm -f

#### 6.3.2 工具清理
PurgeTxnLog的工具，http://zookeeper.apache.org/doc/r3.4.3/api/index.html
    
    java -cp zookeeper.jar:lib/slf4j-api-1.6.1.jar:lib/slf4j-log4j12-1.6.1.jar:lib/log4j-1.2.15.jar:conf org.apache.zookeeper.server.PurgeTxnLog<dataDir><snapDir> -n <count>
最后一个参数希望保留的历史文件个数，注意，count必须是大于3的整数。写成一个定时任务，以便每天定时执行清理。

#### 6.3.3 zk脚本
ZK自己脚本,类似上面在bin/zkCleanup.sh中，所以直接使用这个脚本也是可以执行清理工作的。

#### 6.3.4 参数(因为现在版本使用较高，所以使用这个方法清理)
从3.4.0开始，zookeeper提供了自动清理snapshot和事务日志的功能，通过配置 autopurge.snapRetainCount 和 autopurge.purgeInterval 这两个参数能够实现定时清理了。这两个参数都是在zoo.cfg中配置的：

    autopurge.purgeInterval     #参数指定了清理频率，单位是小时，需要填写一个1或更大的整数，默认是0，表示不开启自己清理功能。
    
    autopurge.snapRetainCount   #参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。

注意：
ZK程序日志默认是没有向ROLLINGFILE文件输出程序运行时日志的，需要自己在conf/log4j.properties中配置日志路径。没有特殊要求的话，日志级别设置为INFO或以上，日志级别设置为DEBUG的话，性能影响很大！

### 6.4 进程保护(数人使用 docker 封装，这里可以忽略)
Daemontools(http://cr.yp.to/daemontools.html) 
SMF（http://en.wikipedia.org/wiki/Service_Management_Facility）

### 6.5 监控
- ZK 4字命令返回内容进行解析，可以获取ZK运行时信息。
- 用jmx也能够获取一些运行时信息，详细可以查看这里：http://zookeeper.apache.org/doc/r3.4.3/zookeeperJMX.html
- 淘宝网已经实现的一个ZooKeeper监控——TaoKeeper，已开源，在这里： http://rdc.taobao.com/team/jm/archives/1450
    主要功能如下:
    - 机器CPU/MEM/LOAD的监控
    - ZK日志目录所在磁盘空间监控
    - 单机连接数的峰值报警
    - 单机Watcher数的峰值报警
    - 节点自检
    - ZK运行时信息展示

### 6.6 故障处理
加载数据出错
ZK在启动的过程中，会根据事务日志中的事务日志记录，从本地磁盘加载最后一次提交时候的快照数据，如果读取事务日志出错或是其它问题（通常在日志中可以看到一些IO异常），将导致server将无法启动。碰到类似于这种数据文件出错导致无法启动服务器的情况

处理顺序：
1. 确认集群其它机器是否正常工作，方法是使用“stat”这个命令来检查：echo stat|nc ip 2181
- 如果确认其它机器是正常工作的（集群中过半机器可用），可以开始删除本机的一些数据了，删除$dataDir/version-2和$dataLogDir/version-2 两个目录下的所有文件。
- 重启server。重启之后，机器就会从Leader那里同步到最新数据，然后重新加入到集群中提供服务。

## 7 快速模式
### 7.1 tar包单机模式（7步）
Step1：配置JAVA环境。检验方法：执行java –version和javac –version命令。

Step2：下载并解压zookeeper。
    wget http://archive.apache.org/dist/zookeeper/stable/zookeeper-3.4.6.tar.gz && tar -zxvf zookeeper-3.4.6.tar.gz -C /opt &&  ln -s /opt/zookeeper-3.4.6 /usr/local/zookeeper && chown -R zookeeper:hadoop /opt/zookeeper*

Step3：重命名 zoo_sample.cfg文件
    mv /usr/local/zookeeper/conf/zoo_sample.cfg  /usr/local/zookeeper/conf/zoo.cfg 

Step4：vi zoo.cfg，修改
    dataDir=/usr/local/zookeeper/data 

Step5：创建数据目录：
    mkdir /usr/local/zookeeper/data

Step6：启动zookeeper：执行
    /usr/local/zookeeper/bin/zkServer.sh start 

Step7：检测是否成功启动：执行
    /usr/local/zookeeper/bin/zkCli.sh or echo stat|nc localhost 2181 

### 7.2 tar包集群模式（8步）
Step1：配置JAVA环境。检验方法：执行java –version和javac –version命令。

Step2：下载并解压zookeeper。
    wget http://archive.apache.org/dist/zookeeper/stable/zookeeper-3.4.6.tar.gz && tar -zxvf zookeeper-3.4.6.tar.gz -C /opt &&  ln -s /opt/zookeeper-3.4.6 /usr/local/zookeeper && chown -R zookeeper:hadoop /opt/zookeeper*

Step3：重命名 zoo_sample.cfg文件
    mv /usr/local/zookeeper/conf/zoo_sample.cfg  /usr/local/zookeeper/conf/zoo.cfg 

Step4：vi zoo.cfg，修改
    dataDir=/usr/local/zookeeper/data server.1=1.2.3.4:2888:3888 server.2=1.2.3.5:2888:3888 server.3=1.2.3.6:2888:3888
    这里要注意下server.1这个后缀，表示的是1.2.3.4这个机器，在机器中的server id是1

Step5：创建数据目录：
    mkdir /usr/local/zookeeper/data

Step6：标识Server ID.
	在/usr/local/zookeeper/data目录中创建文件 myid 文件，每个文件中分别写入当前机器的server id，例如1.2.3.4这个机器，在/usr/local/zookeeper/data目录的myid文件中写入数字1.

Step7：启动zookeeper：执行
    /usr/local/zookeeper/bin/zkServer.sh start 

Step8：检测是否成功启动：执行
    /usr/local/zookeeper/bin/zkCli.sh or echo stat|nc localhost 2181 


## 参考文档
[官方](http://zookeeper.apache.org)
[阿里团队](http://jm-blog.aliapp.com/?p=2318)
[IBM Zookeeper基础知识](http://www.ibm.com/developerworks/cn/data/library/bd-zookeeper/)