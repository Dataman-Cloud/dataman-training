# Docker 升级 1.9.1 - 1.11.2
Docker 升级步骤

1. 关闭 mesos-slave

        docker stop dataman-slave
2. 升级 Docker
    - 执行安装脚本(这里这样执行是最新版)
    
            curl -fsSL https://get.docker.com/ | sh
    - 介入交互命令行
            
            docker (Y/I/N/O/D/Z) [default=N] ? N
    - Bug 无法正常退出手工退出
            
            Ctrl+c
    - 杀掉 Docker apt 任务
            
            ps aux|grep apt-get|grep docker-engine|awk '{print $2}'|xargs kill
    - 使用命令再次进入安装docker交互
    
            dpkg --configure -a
    - 默认回车完成安装              
3. Docker 启动后会自动转换原有镜像
        
        top 查看 iowait 参数
4. 转换完毕后需要调节配置的机器这里可以关闭调节配置并重启
5. 启动原先服务

# Docker 升级 1.11.2 - 1.12.1
1. ubuntu升级流程跟 1.9.1-1.11.2相同
2. centos升级没有提示交互指令，直接覆盖原有配置文件，需要重新修改docker配置

## 0.23.0 - 0.24.2
### 1.1 不变服务
- registry.shurenyun.com/centos7/zookeeper-3.4.6:20160419162459

### 1.2 原环境
- registry.shurenyun.com/centos7/mesos-0.23.0-marathon-0.9.1:omega.v0.2.12
- registry.shurenyun.com/centos7/mesos-0.23.0-master:omega.v0.2.12
- catalog.shurenyun.com/library/centos7-mesos-0.23.0-slave:v0.5.2016052300
- 发布一组测试 app

### 1.3 升级环境 
- demoregistry.dataman-inc.com/library/centos7-mesos-0.24.2-marathon-0.13.1:20160603162523
- demoregistry.dataman-inc.com/library/centos7-mesos-0.24.2-master:20160601155857
- demoregistry.dataman-inc.com/library/centos7-mesos-0.24.2-slave:20160601172443

### 1.4 升级步骤
按照官方步骤进行测试，测试目的主要是已发任务的管理情况

- 更换 master
        
        按照要求，先更换非 Leader 节点，启动正常后，通过 api 检查，再更换 Leader 节点。更换后查看发布的 App 正常。 
 
- 更换 slave

        更换所有 slave，如果遇到有任务的 slave,会重新发布任务，等过段时间原来的任务将被回收。
        这里切记一旦重新启动了slave，不管原来的任务实例是否还在，这个实例都将被清除出服务发现组件，在mesos集群中自重启动slave开始。 

- 更换 调度器

        同 master 方法，因为数人使用的是 Docker on Docker 的方法，所以替换 Mesos 库文件的做法在 Docker 制作就完成了。

## 注意事项
1. 注意如果发布的任务固定发送到某台机器，尤其是 host 模式或者是抢占资源任务情况下，升级前必须先暂停这些任务后再升级，否则会导致资源抢占无法处理。
2. 新版本需要测试已使用的非 marathon 调度器，如jenkins，chronos等，支持再更换
3. 还需要测试服务发现组件，如 bamboo ,mesosdns 
4. 升级 Docker 遇到升级完毕无法启动，重启后正常
5. 升级 Docker 遇到 zabbix repo 获取异常，跳过正常 
        
## 0.24.2 - 0.27.2
### 2.1 不变服务
- registry.shurenyun.com/centos7/zookeeper-3.4.6:20160419162459

### 2.2 原环境
- demoregistry.dataman-inc.com/library/centos7-mesos-0.24.2-marathon-0.13.1:20160603162523
- demoregistry.dataman-inc.com/library/centos7-mesos-0.24.2-master:20160601155857
- demoregistry.dataman-inc.com/library/centos7-mesos-0.24.2-slave:20160601172443
- 发布一组测试 appve

### 2.3 升级环境 Mesos 
- meos-slave
    - demoregistry.dataman-inc.com/library/centos7-mesos-0.27.2-slave
    - demoregistry.dataman-inc.com/library/centos7-mesos-0.26.1-slave (docker in docker 不可用)
    - demoregistry.dataman-inc.com/library/centos7-mesos-0.25.1-slave (docker in docker 不可用)
- mesos-master    
    - demoregistry.dataman-inc.com/library/centos7-mesos-0.27.2-master
    - demoregistry.dataman-inc.com/library/centos7-mesos-0.26.1-master
    - demoregistry.dataman-inc.com/library/centos7-mesos-0.25.1-master
- marathon    
    - demoregistry.dataman-inc.com/library/centos7-mesos-0.27.2-marathon-0.15.5

### 2.4 升级步骤
按照官方步骤进行测试，测试目的主要是已发任务的管理情况

- 更换 master
        
        所有 Master 均正常启动，并可以提供服务
- 更换 slave

        0.25.1 和 0.26.1 因为增加了 systemd_enable_support,所以无法在 Docker 环境下启动,增加 path 号称可以解决，但测试失败。
        0.27.2 因为支持了参数 systemd_enable_support 的关闭选项，所以可以启动，启动脚本增加 ENV 环境变量。
            -e MESOS_SYSTEMD_ENABLE_SUPPORT=false
- 更换 调度器

        调度器直接升级到了 15.5 版本，正常
               
    as
