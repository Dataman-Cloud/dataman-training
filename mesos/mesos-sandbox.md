# Mesos 沙盒
    文档信息
    创建人 庞铮
    邮件地址 zpang@dataman-inc.com
    建立时间 2016年6月1号
    更新时间 2016年6月1号

## 介绍
Mesos 沙盒是指给单个执行器保持特定文件的临时目录。每一个运行中的执行器都会将工作执行目录和沙盒设置成沙箱目录。
## 沙盒文件分为以下几种
- 在执行器启动任务之前，通过Mesos获取文件。
- 执行器或者任务的输出(包含"标准输入"和"标准输出")。
- 由执行器或者任务创建的文件，由一些例外。

        注:随着引进[持久化卷](http://mesos.apache.org/documentation/latest/persistent-volume/)，执行器和任务不应该在沙盒之外创建文件。然而一些容器不支持沙盒操作。
        
## 沙盒在哪里
沙盒位于 `mesos-slave` 本地工作目录中(可以由 `--work_dir` 参数指定)。要查找特定执行器的沙盒，必须知道 `mesos-slave`ID、框架ID、执行器ID、容器ID。

沙盒在 `Mesos-slave` 上的路径

    root ('--work_dir')
    |-- slaves
    |   |-- latest (symlink)
    |   |-- <agent ID>
    |       |-- frameworks
    |           |-- <framework ID>
    |               |-- executors
    |                   |-- <executor ID>
    |                       |-- runs
    |                           |-- latest (symlink)
    |                           |-- <container ID> (Sandbox!) 
可以通过 Docker-Name 反查路径，如`mesos-20151231-103945-119603978-5050-1-S2.f8fd95a8-9b56-45a4-9e1f-6a345cad01c4`
## 使用沙盒
    注:对于不是 Mesos 的其它程序，如执行器、任务等，沙盒应该被视为一个只读目录。这个不是通过强制权限限定的，但是如果在外面强行修改沙盒内容可能会导致执行器或任务出现故障。
- 通过文件目录游览
    
    可以通过刚才介绍的路径进行游览
- 通过 Mesos web UI 游览

    可以通过登录 Mesos web UI 进行查看和下载沙盒目录下所有数据。

- 通过 /file endpoint 游览

    可以通过 Mesos-slave Web UI 的 /files endpoint 获取文件信息
    
    - /files/debug.json
    
        返回一个json字典，是内部文件对系统目录的映射，用于快速获取沙盒中所有文件路径，其中包括所有执行器的stdout\err && mesos-slave 的日志。
    - /files/browse.json?path=(mesos任务路径，参考 debug.json 数据路径)
    
        返回一个json列表，包含该路径下的文件和目录信息(类似系统通常是 `ls -l` 命令显示)

    - /files/download.json?path=(mesos任务路径，参考 browse.json 具体文件路径)
    
        下载原始日志文件内容，在 Content-Type header 适当设置将可以兼容文件扩展名。
    - /files/read.json?path=(mesos任务路径，参考 browse.json 具体文件路径)
    
        读取指定的文件的一部分，返回包含 data 和 offset 的json对象。`注意这个接口被设计读取任何二进制文件，所以可能返回不支持json格式，可以使用download.json 代替。`
    - /help/files
    
        帮助索引，具体参数可以在这里查询。

## 沙盒大小
沙盒的最大空间取决于容器对于执行器隔离策略的设定:

- Mesos 自带容器
    
    为了向后兼容，自带容器默认情况下不支持磁盘配额。但可以通过在mesos-slave 上增加参数 ` --enforce_container_disk_quota` 和在`--isolation`上增加`posix/disk`，这样如果沙盒大小超过了设定将会被kill。
- Docker

    Docker 1.9.1 不强制也不支持磁盘配额。详见[Docker issue](https://github.com/docker/docker/issues/3804)
- [外部容器](https://github.com/docker/docker/issues/3804)

## 沙盒生命周期
沙盒的文件垃圾回收将由以下情况触发：

- 执行器被移除或者停止
- 执行任务框架被移除
- 执行器恢复任务过程中失败

`注意：在代理恢复执行器运行中的任务，除了最新运行的执行器，其它将被回收`

垃圾回收基于 `--gc_delay` 参数。默认情况周期为一周，在一周后将被删除沙盒内部文件。另外，可以使用`--disk_watch_interval` 和 `--gc_disk_headroom` 在可用磁盘进行垃圾文件的修剪。
