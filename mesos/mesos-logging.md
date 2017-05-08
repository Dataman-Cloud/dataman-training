# LOGGING
mesos 处理日志根据会分很多类型，粗略的分以下几种：

- [内部组件](http://mesos.apache.org/documentation/latest/logging/#Internal)

    Mesos-master 和 Mesos-agent(slave) 节点程序自身日志
- [容器](http://mesos.apache.org/documentation/latest/logging/#Containers)
    
    执行器和任务
- 外部组件

    mesos 集群依赖的服务，如框架和zk等

## 内部组件
Mesos-master 和 Mesos-agent(slave) 使用的是 [Google 的日志库](https://github.com/google/glog)。有关配置选项请参考[配置文档](http://mesos.apache.org/documentation/latest/configuration/)。配置并没有明确可以通过环境变量进行配置。

Mesos-master 和 Mesos-agent(slave) 通过暴露 [/logging/toggle](http://mesos.apache.org/documentation/latest/endpoints/logging/toggle/) HTTP 节点进行日志详细查看。

    POST <ip:port>/logging/toggle?level=[1|2|3]&duration=VALUE
    
效果类似启动 Mesos-master 和 Mesos-agent(slave) 时设置 `GLOG_v` 环境变量。 `duration` 参数如果设置，将是持续多久显示这个级别日志，如果到期将返回到原来的日志等级。
## 容器
参考文档[容器文档](http://mesos.apache.org/documentation/latest/containerizer/), Mesos 对于容器内部运行的实例，不承担任何