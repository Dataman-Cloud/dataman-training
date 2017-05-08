# 改造计划 Docker 和 Mesos 计划
本次升级的工作重点:
## 1 Docker 升级步骤
为了简化复杂度，进行分层升级，Docker 先升级。
### 1.1 关注点
1. 版本升级到1.11
2. 关闭 Docker 日志输出(现阶段暂时不关闭，等完全切换后可关闭)

### 1.2 解决问题
1. Docker pull 问题解决
2. Mesos 系统双重日志问题(启动程序关闭 docker 日志)
3. 新版本特性热加载

### 1.3 Docker 数人云平台环境升级顺序
测试环境 -> Dev 升级 -> Demo 升级 -> Prod 升级

- 测试环境(金烨已经完成)

    测试转换 Docker 版本对已有镜像的影响和兼容度
- Dev 升级(金烨已经完成)
    
    因 Dev 属于单机环境，数人云组件兼容性测试
- Demo 升级(金烨已经完成)

    Demo 是仿生产环境，所以需要在这里测试不下线升级的方法。
    
    1. 所有多点服务升级(逐一)
    2. 单点服务修改成多点服务
        - VIP 方式部署模块 
            - redis
            - rabbitmq
        - 改成 nat 模式部署
            - harbor
            - drone
    3. 程序本身不支持多点的服务
        - alrte
        - drone
        - infulxDB        
- Prod 升级(金烨已经完成)

    生产环境升级，使用 Demo 测试的方案进行升级。

## 2 Mesos
### 2.1 关注点
1. 版本升级到最低0.24.2最高最新
2. 将 Mesos-slave 的工作目录挂载出来
3. 积累升级 Mesos 系统经验
4. Marathon 加参数支持一周期大数量实例秒启
5. Jenkins 对于 Mesos 新版本支持问题
6. bamboo 对 Mesos 新版本支持
7. mesos-dns 对 Mesos 新版本支持

### 2.2 解决问题
1. Mesos-slave 任务异常以及日志收集改造问题
2. 支持命令行健康检查，以便于支持 marathon put 操作
3. Mesos 新版本特性使用经验累积

### 2.3 新增参数
- 目录挂载

        -v /data/mesos:/data/mesos
- 定时回收失败实例(暂时不修改，这里的日志是有必要保留的6个小时的)

        --docker_remove_delay=1hrs(默认6hrs)
- 不使用swap

        cgroups_limit_swap
- 去掉打印到内部的日志(包括master && slave)

        不设置:log_dir 

### Mesos 升级方案
1. 进行 Jenkins-Master 的版本测试(查看 Mesos 插件支持情况)
- 查询 Mesos 兼容版本列表
- 升级模拟集群(测试挂载出来slave 上服务升级的反应，想法：适当调节 Master 的应用敏感度来适应固定服务的升级)
- 升级 dev slave
- 升级 demo slave
- 升级 master(逐一升级)
- 升级 marathon(逐一升级)
- 检查升级结果(注意)
- 循环第二步一只升级到 Jenkins 兼容的版本

### Mesos 升级列表
### Mesos
[mesos](http://mesos.apache.org/documentation/latest/upgrades/)

[mesosphere](http://open.mesosphere.com/downloads/mesos/#apache-mesos-0.24.0)

| mesos 老版本 | mesos 新版本 | 最新版本  |
| ------------ | ------------- | ------------ |
| 0.22.x | 0.23.x  | 0.23.1 |
| 0.23.x | 0.24.x  | 0.24.2 |
| 0.24.x | 0.25.x  | 0.25.1 |
| 0.25.x | 0.26.x  | 0.26.1 |
| 0.26.x | 0.27.x  | 0.27.2 |
| 0.27.x | 0.28.x  | 0.28.2 |
### [Marathon 官方升级列表](https://github.com/mesosphere/marathon/releases?after=v1.0.0-RC1)
| mesos 版本 | marathon 老版本 | marathon 新版本 |
| ------------ | ------------- | ------------ |
| 0.22.1 | 0.9.1  | 0.9.2 |
| 0.22.1 | 0.9.0  | 0.10.0 |
| 0.22.1 | 0.10.0  | 0.10.1 |
| 0.23 | 0.10.0  | 0.11.0 |
| 0.23 | 0.11.0  | 0.11.1 |
| 0.23 | 0.11.1  | 0.13.0 |
| 0.23 | 0.13.0  | 0.13.1 |
| 0.26 | 0.13.0  | 0.14.0 |
| 0.26 | 0.14.0  | 0.14.1 |
| 0.26 | 0.14.0  | 0.15.0 |
| 0.26 | 0.15.0  | 0.15.1 |
| 0.26 | 0.15.1  | 0.15.2 |
| 0.26 | 0.15.2  | 0.15.3 |
| 0.26 | 0.15.3  | 0.15.4 |
| 0.26 | 0.15.4  | 0.15.5 |
| 0.28 | 0.15.3  | 1.0.0-RC1 |
| 0.28 | 1.0.0  | 1.1.0 |
| 0.28 | 1.1.0  | 1.1.1 |