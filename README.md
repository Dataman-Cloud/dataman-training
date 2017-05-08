# 1. mesos
## 1.1 学习mesos 以及相关组件   

 * [zookeeper](./zookeeper)
 * [mesos](./mesos)
 * [marathon](./marathon)
 * [bamboo](./bamboo)
 * [haproxy](./haporxy)

## 1.2 验收
```
1. 独立部署一套mesos环境
2. 说明bamboo，haproxy， marathon之间的关系
```
 

# 2. docker
目标： 了解docker基本概念，会用常用的docker命令


 * [docker](https://docs.docker.com/get-started/)
 * [docker 命令](https://yeasy.gitbooks.io/docker_practice/content/)
 * [dockerfile](http://blog.csdn.net/qinyushuang/article/details/43342553)
 * [docker与虚拟机](http://blog.csdn.net/cbl709/article/details/43955687)
 * [镜像，容器，镜像仓库](https://yeasy.gitbooks.io/docker_practice/content/basic_concept/)
 * [容器生命周期管理](http://blog.csdn.net/screaming/article/details/50685916)
 * [镜像仓库](https://github.com/vmware/harbor)
 * [docker-compse](https://docs.docker.com/compose/)

## 2.1 验收
### 2.1.1 docker 命令  

```
1. 运行一个ubuntu镜像
2. 运行一个tomcat，并将内部的8080映射到外部的8090，-p 与 -P 有什么差别？
3. 说明bridge，host的区别与使用场景
4. 什么场景下使用-v 挂盘？ 性能上有什么差别
5. 说明--restart都有哪些参数，具体使用方法
6. logs 命令都有哪些参数，如何使用
7. inspect 记录了哪些内容
8. info， ps， version 分别记录了什么信息
9. attach, exec 之间的区别
10. 从docker store 获取一个镜像，镜像存储在宿主机的什么地方，如何存储？
11. push 一个镜像到镜像仓库，思考镜像名与镜像仓库有什么关系？
12. 将容器制作成镜像，思考 commit 和dockerfile 的区别
13. 如何查看一个镜像
14. 如何删除一个镜像
15. 如何将镜像转移到另外一台主机上使用
16. 搭建一个镜像仓库
```

### 2.1.2 dockerfile
```
1. dockerfile的基本命令
2. cmd 和 Entrypoint 的区别
3. copy 和 add 的区别
4. 在实践的基础上，总计一份dockerfile的最佳实践
```

### 2.1.3 docker-compose
```
1. docker-compose 的基本命令
2. 说明docker-compose v1, v2, v3 之间的区别
2. 通过docker-compose 部署 一个LNMP 服务
```


# 3. 数人云平台部署
## 3.1 部署

参照数人云离线包里的安装文档部署


目标： 

	1. 独立部署数人云平台（测试环境，高可用环境）
	2. 能够处理安装部署中遇到的问题和故障
	3. 了解数人云平台每个组件的功能


## 3.2 验收: 
```
1. 独立部署数人云环境(测试部署，高可用部署)
2. 说明数人云平台安装部署过程中使用了哪些组件，每个组件的角色和功能
3. 使用host模式迁移，部署一套bbs系统
4. 使用nat模式部署2048，并且可以扩展
5. 通过现有的bamboo-haproxy方案实现2048的服务发现和负载均衡
6. 总结安装部署，以及迁移应用过程中遇到的问题
7. 输出一份数人云安装和以上服务使用的交付文档
```

# 4. 网络
## 4.1 calico

### 4.1.1 calico教程
 * [calico 教程](https://www.projectcalico.org/)

### 4.1.2 验收
```
1. 了解calico工作原理
2. 独立部署一个calico网络
```
 
 
# 5. 存储

 
