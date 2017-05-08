# IP-per-task(1-容器 1-ip)
这个功能在 marathon 0.14 版本提供支持，该功能暂时被定义成实验性质，所以风险需要自己承担。后面的版本的功能和 API 将可能会被修改。具体查看[MARATHON-2790](https://github.com/mesosphere/marathon/issues/2709)。

当这个功能必须是 Mesos 0.26 或以上版本才支持，并且请确保安装了   Network Isolation 模块 和 IP Access Manager (IPAM) 模块和被适当的配置激活后，每个任务都可能会有自己的IP和网卡，这将极大的简化服务发现配置，只要使用 Mesos-DNS 配置即可。只需使用 DNS(A 纪录)加已知端口就可以访问服务了。

注意：如果任务要求使用 IP-per-task 模式，那么 mesos-slave 就不能对任务进行端口设置。

- 样例配置
    
        {
          "id": "/i-have-my-own-ip",
          // ... more settings ...
          "ipAddress": {}
        }

# Network security groups
如果支持网络访问管理(IPAM)，就可以在 ip 设置选项里使用网络安全组和使用标签进行细化配置。常见的网络安全组配置中，在任务和任务之间，仅允许网络流量设置。这很容易的管理你生产环境和测试环境之前的流量干扰。

- 样例
    
        {
          "id": "/i-have-my-own-ip",
          // ... more settings ...
         "ipAddress": {
             "groups": ["production"],
             "labels": {
                 "some-meaningful-config": "potentially interpreted by the IPAM"
                        }
                      }
        }
# Service Discovery
如果任务要求使用 IP-per-task 模式，那么 mesos-slave 就不能对任务进行端口设置，但还可以对使用的端口进行描述，这样这些信息将传递到 Mesos 并传给 Mesos-DNS 用来做服务发现使用。

- 样例

        {
          "id": "/i-have-my-own-ip",
          // ... more settings ...
          "ipAddress": {
              "discovery": {
              "ports": [
                        { "number": 80, "name": "http", "protocol": "tcp" }
                      ]
                            }
          // ... more settings ...
                      }
        } 
       