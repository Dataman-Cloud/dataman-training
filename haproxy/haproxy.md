# Haporxy
## 0 编译安装
### 0.1 下载
    wget http://www.haproxy.org/download/1.5/src/haproxy-1.5.15.tar.gz
### 0.2 编译准备
    yum install -y gcc \
                mack \
                pcre-devel \
                openssl-devel \
                zlib-devel \
### 0.3 编译
#### 0.3.1 解压
    tar xzvf haproxy-1.5.15.tar.gz && cd haproxy-1.5.15
#### 0.3.2 查看编译参数
    less  README
#### 0.3.3 准备
    # 创建用户
    groupadd haproxy
    useradd -M -g haproxy haproxy
    # 创建haproxy 需要目录
    mkdir -p /run/haproxy
    chown haproxy.haproxy /run/haproxy
    
    mkdir -p /var/lib/haproxy
    chown haproxy.haproxy /var/lib/haproxy
#### 0.3.4 实体编译脚本
    vi build.sh
        
    if [ -d haproxy-1.5.15 ];then
        rm -Rf haproxy-1.5.15
    fi

    tar xzvf haproxy-1.5.15.tar.gz && \
    cd haproxy-1.5.15 && \
    make TARGET=linux2628  \
        CPU_CFLAGS.generic=-O2 \
        ARCH_FLAGS.x86_64="-m64 -march=x86-64" \
        SPEC_CFLAGS=-fno-strict-aliasing \
        USE_PCRE=1 PCREDIR= \
        USE_PCRE_JIT=1 \
        USE_LINUX_SPLICE=1 \
        USE_LINUX_TPROXY=1 \
        USE_EPOLL=1 \
        USE_GETSOCKNAME=1 \
        USE_NETFILTER=1  \
        USE_LIBCRYPT=1 \
        USE_ACCEPT4=1  \
        USE_CPU_AFFINITY=1 \
        ASSUME_SPLICE_WORKS=1 \
        USE_OPENSSL=1 \
        USE_TFO=1 \
        USE_ZLIB=1 && \
    make install && \
    haproxy -vv
#### 0.3.4 参数说明
- USE_PCRE or USE_STATIC_PCRE 
    - 参数说明    
    
        PCRE(Perl Compatible Regular Expressions中文含义：perl语言兼容正则表达式)是一个用C语言编写的正则表达式函数库，由菲利普.海泽(Philip Hazel)编写。PCRE是一个轻量级的函数库，比Boost之类的正则表达式库小得多。PCRE十分易用，同时功能也很强大，性能超过了POSIX正则表达式库和一些经典的正则表达式库。增加 libpcre 支持，比其他的库快几倍。
    - 特殊说明
     
        Haproxy 中原先使用的是 USE_PCRE ，最新参数是 USE_STATIC_PCRE ，现在还兼容之前的 USE_PCRE 参数。
    - 参数方法
        
        USE_PCRE=1 or USE_STATIC_PCRE=1
    - 编译结果
        
        Built with PCRE version : 8.32 2012-11-30        
- PCREDIR
    - 参数说明
        如果不加这个参数，因为默认 haproxy 选择只匹配 /usr/lib(32位),故编译不支持：
        
                /usr/bin/ld: cannot find -lpcreposix
                /usr/bin/ld: cannot find -lpcre
                collect2: error: ld returned 1 exit status
                或者
                Built without PCRE support (using libc's regex instead)
    - 参数方法
    
        这里因为新参数 USE_STATIC_PCRE 无法配合PCREDIR 使用，所以使用老参数 USE_PCRE 联合 PCREDIR 使用
        
            USE_PCRE=1 PCREDIR=                         
- USE_LINUX_SPLICE
    - 参数说明
    
      内核2.6版本后，添加的 splice() 调用是 Linux 中与 mmap() 和　sendfile() 类似的一种方法，但是有资料显示使用系统调用后，cpu 优化50%以上，跨磁盘拷贝优化30%－70%,原理是网卡之间基于内核之间传输数据而不需要在拷贝数据，减少拷贝数据的cpu占用。
      
      有些网卡可能会导致性能低，因为网卡驱动的管道大小被限制。(64k - 16 pages or 16 segments )        
    
    - 编译结果             
            
      无
      
    - haproxy 配置
            
            # 打开
            # 自动判断splice能否提供更好的性能
            option splice-auto
            # 对请求采用splice转发
            option splice-request
            # 对回复采用splice转发
            option splice-response        
            #global关闭
            nosplice
            #设置每个进程最大的pipe数，默认是maxconn/4, 由于一个pipe占用2个文件描述符，所以需要增大ulimit-n值，如果没有足够的pipe就会使用原有的模式
            maxpipes 256000
            
## 1 轮询算法
- 'roundrobin'(简单的轮询)
    
    表示简单的轮询，每个服务器根据权重轮流使用，在服务器的处理时间平均分配的情况下这是最流畅和公平的算法。
    
    该算法是动态的，对于实例启动慢的服务器权重会在运行中调整。
    
        192.168.10.5 It is work!
        192.168.10.3 It is work!
        192.168.10.5 It is work!
        192.168.10.3 It is work!
        192.168.10.3 It is work!
        192.168.10.3 It is work!
 - 'leastconn'(最少连接者先处理)
    
    连接数最少的服务器优先接收连接。leastconn建议用于长会话服务，例如LDAP、SQL、TSE等，而不适合短会话协议。如HTTP.
    
    该算法是动态的，对于实例启动慢的服务器权重会在运行中调整。

        192.168.10.5  It is work!
        192.168.10.3 It is work!
        192.168.10.5  It is work!
        192.168.10.3 It is work!

- 'static-rr'(根据权重)

    每个服务器根据权重轮流使用，类似roundrobin，但它是静态的，意味着运行时修改权限是无效的。另外，它对服务器的数量没有限制。    
    
    该算法一般不用；
    
        server dataman01 192.168.10.5:81 cookie app1inst3 check inter 2000 rise 2 fall 5  weight 1
        server dataman02 192.168.10.3:80 cookie app1inst4 check inter 2000 rise 2 fall 5  weight 5

        192.168.10.5 It is work!
        192.168.10.3 It is work!
        192.168.10.3 It is work!
        192.168.10.3 It is work!
        192.168.10.3 It is work!
        192.168.10.3 It is work!
- 'source'(根据请求源IP)

    对请求源IP地址进行哈希，用可用服务器的权重总数除以哈希值，根据结果进行分配。只要服务器正常，同一个客户端IP地址总是访问同一个服务器。如果哈希的结果随可用服务器数量而变化，那么客户端会定向到不同的服务器；
    
    该算法一般用于不能插入cookie的Tcp模式。它还可以用于广域网上为拒绝使用会话cookie的客户端提供最有效的粘连；该算法默认是静态的，所以运行时修改服务器的权重是无效的，但是算法会根据“hash-type”的变化做调整
    
        192.168.10.5  It is work!
        192.168.10.5  It is work!
        192.168.10.5  It is work!
        192.168.10.5  It is work!
        192.168.10.5  It is work!
- 'uri'(根据请求的URI)

    表示根据请求的URI左端（问号之前）进行哈希，用可用服务器的权重总数除以哈希值，根据结果进行分配。只要服务器正常，同一个URI地址总是访问同一个服务器。一般用于代理缓存和反病毒代理，以最大限度的提高缓存的命中率。该算法只能用于HTTP后端；
    
    该算法一般用于后端是缓存服务器；该算法默认是静态的，所以运行时修改服务器的权重是无效的，但是算法会根据“hash-type”的变化做调整。

        192.168.109.3 It is work!
        192.168.109.3 It is work!
        192.168.109.3 It is work!
        192.168.109.3 It is work!
        192.168.109.3 It is work!
        192.168.109.3 It is work!
- 'url_param'(根据请求的URl参数) 'balance url_param' requires an URL parameter name
    
    在HTTP GET请求的查询串中查找<param>中指定的URL参数，基本上可以锁定使用特制的URL到特定的负载均衡器节点的要求；

    该算法一般用于将同一个用户的信息发送到同一个后端服务器；该算法默认是静态的，所以运行时修改服务器的权重是无效的，但是算法会根据“hash-type”的变化做调整。

        balance url_param www.shurenyun.com
        
        192.168.109.5  It is work!
        192.168.109.3 It is work!
        192.168.109.5  It is work!
        192.168.109.3 It is work!
        192.168.109.5  It is work!
        192.168.109.3 It is work!
        192.168.109.5  It is work!
        192.168.109.3 It is work!
        192.168.109.5  It is work!
        192.168.109.3 It is work!
 
- 'hdr(name)' (根据HTTP请求头来锁定每一次HTTP请求)
 
    在每个HTTP请求中查找HTTP头<name>，HTTP头<name>将被看作在每个HTTP请求，并针对特定的节点；如果缺少头或者头没有任何值，则用roundrobin代替；

    该算法默认是静态的，所以运行时修改服务器的权重是无效的，但是算法会根据“hash-type”的变化做调整

        192.168.109.5  It is work!
        192.168.109.3 It is work!
        192.168.109.5  It is work!
        192.168.109.3 It is work!    
        192.168.109.5  It is work!
        192.168.109.3 It is work!
        192.168.109.5  It is work!
        192.168.109.5  It is work!
- 'rdp-cookie(name)' (很据cookie(name)来锁定并哈希每一次TCP请求)

    为每个进来的TCP请求查询并哈希RDP cookie<name>；该机制用于退化的持久模式，可以使同一个用户或者同一个会话ID总是发送给同一台服务器。如果没有cookie，则使用roundrobin算法代替；

    该算法默认是静态的，所以运行时修改服务器的权重是无效的，但是算法会根据“hash-type”的变化做调整。

        192.168.109.5  It is work!
        192.168.109.5  It is work!
        192.168.109.3 It is work!
        192.168.109.3 It is work!
        192.168.109.5  It is work!
        192.168.109.3 It is work!
        192.168.109.5  It is work!
        192.168.109.3 It is work!

## 2 优化
### 2.1 硬件优化
#### 2.1.1 Jumbo Frames(官方)
- 什么是jumbo frames

    早期10M网卡，frame大小是1500字节，去掉TCP/IP头40字节，有效数据是1460字节。100M和1000M网卡保持了兼容，也是1500字节。但是对1000M网卡来说，这意味着更多的中断和和处理时间。因此千兆网卡使用“Jumbo Frames”将frmae扩展至9000字节。为什么是9000字节，而不是更大呢？因为32位的CRC校验和对大于12000字节来说将失去效率上的优势，而9000字节对8KB的应用来说，比如NFS，已经足够了。

    Jumbo frames 是指比标准Ethernet Frames长的frame，即比1518/1522 bytes大的frames，Jumbo frame的大小是每个设备厂商规定的，不属于IEEE标准；Jumbo frame 在full-duplex 的Ethernet网络上运行；Jumbo frame定义了一个“link negotiation”协议，来和对端的设备协商，是否对端设备支持使用Jumbo frames；标准的以太网IP报文大小是：1500 bytes，不包含以太网头和FCS的18 bytes（6+6+2+4），如果包含以太网头和FCS，则为1518 bytes；Jumbo frame 一般指的是二层封装三层IP报文的值大于9000bytes的报文。

    Jumbo frames的提出背景：在1998年，Alteon Networks 公司提出把Data Link Layer最大能传输的数据从1500 bytes 增加到9000 bytes，这个提议虽然没有得到IEEE 802.3 Working Group的同意，但是大多数设备厂商都已经支持。
- 修改MTU值

    修改帧大小实际需要的操作就是修改MTU（Maximum Transmission Unit）值，即修改最大传输单元。修改方法如下：
    
    - ifconfig命令修改

            ifconfig ${Interface} mtu ${SIZE} up
            ifconfig eth1 mtu 9000 up
    
        对所有的linux 发行版本都有效。缺点就是重启后失效，需要在开机项中加载。

    - 修改配置文件
        - CentOS / RHEL / Fedora Linux下

                # vi /etc/sysconfig/network-scripts/ifcfg-eth0
                # 增加如下内容
                MTU="9000"
                # 保存后重启网卡生效
                # service network restart
                #启用IPv6地址的，修改IPv6 mtu的参数为
                IPV6_MTU="1280"
        - Debian / Ubuntu Linux下

                # vi /etc/network/interfaces
                #增加如下值
                mtu 9000
                #保存后，重启网络生效
                # /etc/init.d/networking restart
- 查看本机的mtu 

        # netstat -i
        Kernel Interface table
        Iface   MTU Met   RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
        docker0    1500 0         0      0      0 0             0      0      0      0 BMU
        eth0       1500 0  20937840513      0      0 0      21000920524      0      0      0 BMRU
        lo        65536 0  5336724254      0      0 0      5336724254      0      0      0 LRU

- 注意

    最后需要注意的是，在经过交换网络设备时，仅仅修改主机端的MTU值是不行的，还需要交换网络设备上开启jumbo frames功能。使用 ping 检查
    
        ping -i 9000 -f 10.2.10.10
        -l 指定包大小
        -f 选项为通知操作系统不能私自更改该数据包大小
        
#### 2.1.2 LRO enabled(官方)- 虚拟机不支持
- 什么是LRO-接收路径上的优化？ 

    LRO用于为主机显示要处理的较大数据“块”来减少CPU开销。 Large Receive Offload是一项通过减少CPU开销来增加高带宽网络连接呼入吞吐量的技术。 它通过在通过更高的网络栈之前，将来自单一流的多个传入数据包聚合到一个较大的缓冲器，从而减少要处理的数据包数量。
- 检查网卡驱动是否支持 LRO
        
        # 查询到网卡驱动名
        ethtool -i eno1
        driver: igb
        .....
        
        # 查看模块
        modinfo igb
        .....
        parm:           LRO:Large Receive Offload (0,1), default 0=off (array of int)
        ....
- 查看\启动\关闭 LRO
        
        #查看LRO/GRO当前是否打开
        ethtool -k eth0
        ....
        large-receive-offload: off [fixed] #这个状态是不支持
        ....
        
        #关闭LRO  
        ethtool -K eth0 lro off 
        
        #启动LRO  
        ethtool -K eth0 lro on 
        
        ethtool -K eth0 gro off 关闭GRO 

[Linux 下网络性能优化方法简析](http://www.ibm.com/developerworks/cn/linux/l-cn-network-pt/)        
#### 2.1.3 手动绑定网卡中断
当前大多数网卡都是支持硬件多队列的，为了充分发挥多核的性能，需要手动将网卡中断（流量）分配到所有CPU核上去处理； 

- 硬中断绑定

        #查看网卡中断：     
        cat /proc/interrupts 54: 188324418 0 IR-PCI-MSI-edge eth0-TxRx-0 55: 167573416 0 IR-PCI-MSI-edge eth0-TxRx-1 
    
        #绑定网卡中断到CPU核
        echo 01 > /proc/irq/54/smp_affinity
        echo 02 > /proc/irq/55/smp_affinity　　 
    
        #关闭系统自动中断平衡
        service irqbalance stop
- 软中断绑定    
        
        #如果网卡硬件不支持多队列，那就采用google提供的软多队列RPS； 配置方法同硬中断绑定； 查看软队列：
        cat /sys/class/net/eth0/queues/rx-0/rps_cpus cat /sys/class/net/eth0/queues/rx-1/rps_cpus
        #绑定软队列到CPU核
        echo 01 > /sys/class/net/eth0/queues/rx-0/rps_cpus
        echo 02 > /sys/class/net/eth0/queues/rx-1/rps_cpus
        
### 2.2 内核优化
TCP splicing + LRO enabled 官方数据可以cpu占用率下降20%(cookie和session处理有效)
#### 2.2.1 TCP splicing(官方)

#### 2.2.2 Libpcre
### 2.3 配置
    
    #proxy cfg
    global
        #log /dev/log    local0
        #log /dev/log    local1 notice
        log 127.0.0.1    local0 info
        log /dev/stdout  local0 info
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon
        
        # 限制整个haproxy的转发任务的进程最大链接
          maxconn 30000 # keepalive
        # maxconn 100000 # keepalive
        # maxconn 10000 # server-http-close
        # maxconn 10000 # http-close
         
        # process number
        nbproc {{ if (le .NBProc 1) }} 1 {{ else }} {{ .NBProc }} {{ end }}
        nbproc  1

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # Default ciphers to use on SSL-enabled listening sockets.
        # For more information, see ciphers(1SSL).
        # ssl-default-bind-ciphers kEECDH+aRSA+AES:kRSA+AES:+AES256:RC4-SHA:!kEDH:!LOW:!EXP:!MD5:!aNULL:!eNULL

    defaults
        mode                    http  #...............mode { tcp|http|health }...tcp...4......http...7......health............OK
        log                     global #...........................
        option                  httplog #............http............
        option                  dontlognull #....................................
        
        option         http-keep-alive #keepalive
        # option                http-server-close #.................................http......
        # option                httpclose        
        option forwardfor       except 127.0.0.0/8 #..............................
        option                  redispatch #serverId...........................,.......................................
        retries                 3 #3.....................................................................
        timeout http-request    10s  #............
        timeout queue           1m #............
        timeout connect         10s #............
        timeout client          1m #.....................
        timeout server          1m #.....................
        timeout http-keep-alive 10s #...............
        timeout check           10s  #............
        
        #限制每个转发任务的进程最大链接        
          maxconn                30000 # keepalive
        # maxconn                100000 # keepalive
        # maxconn                3000 # server-http-close
        # maxconn                10000 # http-close

        ## centos 7
        errorfile 400 /usr/share/haproxy/400.http
        errorfile 403 /usr/share/haproxy/403.http
        errorfile 408 /usr/share/haproxy/408.http
        errorfile 500 /usr/share/haproxy/500.http
        errorfile 502 /usr/share/haproxy/502.http
        errorfile 503 /usr/share/haproxy/503.http
        errorfile 504 /usr/share/haproxy/504.http

    # dataman stats port
    listen stats :9000
        mode http
        stats enable
        stats hide-version
        stats realm Haproxy\ Statistics
        stats uri /
        stats auth dataman:dataman
        
    # customized endpoints
       {{ range $index, $app := .Apps }}
            {{ $tasks := .Tasks }}
            {{ if $app.Env.BB_DM_ENDPOINTS }}
    # endpoints {{ $app.Env.BB_DM_ENDPOINTS }}
                {{ range $tcpIdx, $endpoint := Split $app.Env.BB_DM_ENDPOINTS "," }}
                    {{ $endpointSlices := Split $endpoint ":" }}
                    {{ $svcType := index $endpointSlices 0 }}
                    {{ $protocol := index $endpointSlices 1 }}
                    {{ $uri := index $endpointSlices 2 }}
                    {{ $port := index $endpointSlices 3 }}
    # len {{ len $endpointSlices }}
                    {{ $eslen := (len $endpointSlices) }}
    # svcType: {{ $svcType }} proto: {{ $protocol }} uri: {{ $uri }} port: {{ $port }}
                    {{ if not (eq $svcType "pri") }}
    #public service, skipped
                    {{ else if and (not (eq $uri "nil")) (eq $port "80") }}
    #pub:http:uri:80, skipped
                    {{ else if  eq $protocol "http" }}
    #http endpoint
    listen {{ $app.EscapedId }}-cluster-http-{{ $port }} :{{ $port }}
        mode http
        balance roundrobin
        cookie DM_LB_ID insert indirect nocache
                        {{ range $page, $task := $tasks }}
        server {{ $app.EscapedId }}-{{ $task.Host }}-{{ if eq 5 $eslen }}.....{{ index $endpointSlices 4 }}{{ else }}{{ index $task.Ports $tcpIdx }}{{ end }} maxconn 10
                        {{ end }}
                    {{ else if  eq $protocol "tcp" }}

    #tcp endpoint
    listen {{ $app.EscapedId }}-cluster-tcp-{{ $port }} :{{ $port }}
        mode tcp
        option tcplog
        balance leastconn
                        {{ range $page, $task := $tasks }}
                            {{ $bport := "" }}
        server {{ $app.EscapedId }}-{{ $task.Host }}-{{ if eq 5 $eslen }}{{ index $endpointSlices 4 }}{{ else }}{{ index $task.Ports $tcpIdx }}{{ end }} {{ $task.Host }}:{{ if eq 5 $eslen }}{{ index $endpointSlices 4 }}{{ else }}{{ index
                        {{ end }}
                    {{ else }}
    #bad endpoint
                    {{ end }}
                {{ end }}
            {{ end }}
       {{ end }}