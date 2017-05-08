# Marathon脚本发布应用指定servicePort问题解决
    文档信息
    创建人 张馨文
    邮件地址 xwzhang@dataman-inc.com
    建立时间 2015年7月9号
    更新时间 2015年7月9号
## 1 问题阐述
    1 使用Bamboo对Marathon的任务进行自动服务发现并更新Haproxy配置，需要解决Haproxy各别服务的端口转发问题。
    2 phantomjs服务在ha上的健康状态始终为灰色，但服务正常。
## 2 解决办法
### 1 修改Bamboo模版配置文件haproxy_template.cfg, 添加如下配置
    {{ range $index, $app := .Apps }}
    listen {{ $app.EscapedId }}_{{ $app.ServicePort }}
      bind *:{{ $app.ServicePort }}
      mode http
      {{ if $app.HealthCheckPath }}
      option httpchk GET {{ $app.HealthCheckPath }}
      {{ end }}
      balance leastconn
      option forwardfor
          {{ range $page, $task := .Tasks }}
          server {{ $app.EscapedId}}-{{ $task.Host }}-{{ $task.Port }} {{ $task.Host }}:{{ $task.Port }} {{ if $app.HealthCheckPath }} check inter 30000 {{ end }} {{ end }}
    {{ end }}   
### 2 配置Marathon任务发布脚本，例如:
    curl -v -X POST http://10.3.2.3:8080/v2/apps \
         -H Content-Type:application/json -d '{
             "id":"phantomjs",
             "container": {
                 "type": "DOCKER",
                 "docker": {
                     "image": "10.3.6.3:5000/phantomjs0708",
                     "network": "BRIDGE",
                     "portMappings": [
                     { "containerPort": 1111, "hostPort": 0, "servicePort": 9000, "protocol": "tcp"} <<---servicePort指定暴露的服务端口
                     ]
                 }
             },
             "ports": [9000],  <<---需指定暴露的服务端口, 和servicePort一致
             "cmd": "/usr/bin/phantomjs --proxy=10.3.1.3:33333 /usr/local/share/login-server.js 1111",
             "cpus": 0.1,
             "mem": 256.0,
             "instances": 10
         }'
    上述脚本servicePort指定服务发现中Haproxy对外暴露的服务端口，Bamboo读取Marathon服务的ServicePort是ports的值，
    指定servicePort同时要指定ports为同样端口，并且ports的值为List格式，如果不指定ports，ports默认为0, 
    如果不指定servicePort，则在10000-20000区间中自动分配端口号。
### 3 查看Bamboo模版配置文件发现，决定ha上服务健康状态的代码如下:
    {{ if $app.HealthCheckPath }}
    option httpchk GET {{ $app.HealthCheckPath }}
    {{ end }}

    是由于Marathon中的健康检查不合格因此导致ha上的健康状况不正常，解决办法就是给phantomjs添加健康状态检查，检查脚本如下:
    "healthChecks": [{
    "protocol": "HTTP",
    "path": "/?href=http://www.baidu.com",   
    "gracePeriodSeconds": 30,
    "intervalSeconds": 30,
    "portIndex": 0,
    "timeoutSeconds": 30,
    "maxConsecutiveFailures": 3
    }],
    重点是path参数，需要根据服务的功能特性给出合适的路径，使Marathon可以正常进行http请求。
    之前健康检查不过关就是因为没有了解应用的使用方法，缺少path参数导致。
