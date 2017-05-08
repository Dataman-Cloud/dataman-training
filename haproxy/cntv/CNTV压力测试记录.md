CNTV压力测试记录

1、优化内核参数 见sysctl.conf

内容复制到到 /etc/sysctl.conf 后 sysctl -p

2、优化文件打开数，进程数

在 /etc/security/limits.conf 最后添加 

```
*             soft    nofile          65535
*             hard    nofile          65535
```

在 /etc/profile 最后添加

ulimit -n 65535
ulimit -s 65535

执行 source /etc/profile

3、压力测试记录

```
并发 容器规格  响应时间（s）  qps      错误连接率     CPU LOAD
2000 单机非容器  0.65        1200      0%           100~持续增加

haproxy_template_keep.cfg 优化haproxy 非会话保持，backend server 连接做限制

2000 单机1容器  1~1.4       800~950    0%           10
2000 单机2容器  0.6~0.7     1200       0%           10~20
2000 单机3容器  0.6~0.7     1200       0%           10~20

简单优化haproxy （扩展后，连接没有自动打到新容器）
2000 单机1容器  0.6~0.7     1200       1~3%         100~持续增加
2000 单机2容器  0.7~1       1200       0~1%         100~持续增加


简单优化haproxy + backend server 连接做限制 （扩展后，连接没有自动打到新容器）
2000 单机1容器  1.5~1.6     750         0%           10
2000 单机2容器  0.7~0.8     1100        0%           10~20
2000 单机3容器  0.65~0.8    1100~1200   0%           20~25


优化haproxy + default优化 + backend server 连接做限制 （扩展后，连接请求没有自动打到新容器，需要重启压力测试工具master,请求才会打到新容器，有会话保持，适应大部分用户需求，最终选择此配置，见haproxy_template_product.cfg）
2000 单机1容器  1~1.2     900~950      0%           7
2000 单机2容器  0.6~0.7    1200        0%           20
2000 单机3容器  0.6~0.7    1200        0%           25~30


haproxy_template_http.cfg 简单优化haproxy + backend server 连接做限制 + default优化+关闭httpclose （没有会话保持）
2000 单机1容器  1          1000        0%           10
2000 单机2容器  0.6        1230        0%           20~25
2000 单机3容器  0.6~0.7    1230        0%           30
```
