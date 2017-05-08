.##Apps
POST /v2/apps

Create and start a new application

创建并开始一个新的应用程序

Here is an example of an application JSON which includes all fields.

以下是一个发布应用程序的JSON例子， 其包括了全部的字段。

    {
    "id": "/product/service/myApp",
    "cmd": "env && sleep 300",
    "args": ["/bin/sh", "-c", "env && sleep 300"]
    "cpus": 1.5,
    "mem": 256.0,
    "ports": [
        8080,
        9000
    ],
    "requirePorts": false,
    "instances": 3,
    "executor": "",
    "container": {
        "type": "DOCKER",
        "docker": {
            "image": "group/image",
            "network": "BRIDGE",
            "portMappings": [
                {
                    "containerPort": 8080,
                    "hostPort": 0,
                    "servicePort": 9000,
                    "protocol": "tcp"
                },
                {
                    "containerPort": 161,
                    "hostPort": 0,
                    "protocol": "udp"
                }
            ],
            "privileged": false,
            "parameters": [
                { "key": "a-docker-option", "value": "xxx" },
                { "key": "b-docker-option", "value": "yyy" }
            ]
        },
        "volumes": [
            {
                "containerPath": "/etc/a",
                "hostPath": "/var/data/a",
                "mode": "RO"
            },
            {
                "containerPath": "/etc/b",
                "hostPath": "/var/data/b",
                "mode": "RW"
            }
        ]
    },
    "env": {
        "LD_LIBRARY_PATH": "/usr/local/lib/myLib"
    },
    "constraints": [
        ["attribute", "OPERATOR", "value"]
    ],
    "acceptedResourceRoles": [ /* since 0.9.0 */
        "role1", "*"
    ],
    "labels": {
        "environment": "staging"
    },
    "uris": [
        "https://raw.github.com/mesosphere/marathon/master/README.md"
    ],
    "dependencies": ["/product/db/mongo", "/product/db", "../../db"],
    "healthChecks": [
        {
            "protocol": "HTTP",
            "path": "/health",
            "gracePeriodSeconds": 3,
            "intervalSeconds": 10,
            "portIndex": 0,
            "timeoutSeconds": 10,
            "maxConsecutiveFailures": 3
        },
        {
            "protocol": "TCP",
            "gracePeriodSeconds": 3,
            "intervalSeconds": 5,
            "portIndex": 1,
            "timeoutSeconds": 5,
            "maxConsecutiveFailures": 3
        },
        {
            "protocol": "COMMAND",
            "command": { "value": "curl -f -X GET http://$HOST:$PORT0/health" },
            "maxConsecutiveFailures": 3
        }
    ],
    "backoffSeconds": 1,
    "backoffFactor": 1.15,
    "maxLaunchDelaySeconds": 3600,
    "upgradeStrategy": {
        "minimumHealthCapacity": 0.5,
        "maximumOverCapacity": 0.2
    }
    }
 id (String)

Unique identifier for the app consisting of a series of names separated by slashes. Each name must be at least 1 character and may only contain digits (`0-9`), dashes (`-`), dots (`.`), and lowercase letters (`a-z`). The name may not begin or end with a dash.

app的唯一标识符是由一些被斜线分开的名字组成。 每个名字必须至少有一个字符并且只有数字（`0-9`）， 破折号（`-`）， 点（`.`）和小写字母组成（`a-z`）。 不能够以破折号开始或结束。

The allowable format is represented by the following regular expression ^(([a-z0-9]|[a-z0-9][a-z0-9\\-]*[a-z0-9])\\.)*([a-z0-9]|[a-z0-9][a-z0-9\\-]*[a-z0-9])$

允许的格式由以下正则表达式表示 ^(([a-z0-9]|[a-z0-9][a-z0-9\\-]*[a-z0-9])\\.)*([a-z0-9]|[a-z0-9][a-z0-9\\-]*[a-z0-9])$


cmd (String)

The command that is executed. This value is wrapped by Mesos via `/bin/sh -c ${app.cmd}`. Either `cmd` or `args` must be supplied. It is invalid to supply both `cmd` and `args` in the same app.

cmd 是被执行的命令。这个值被Mesos通过`/bin/sh -c ${app.cmd}`包裹。无论`cmd`或`args`必须提供。在同一个app中同时提供`cmd`和`args` 是无效的。


args (Array of Strings)

An array of strings that represents an alternative mode of specifying the command to run. This was motivated by safe usage of containerizer features like a custom Docker ENTRYPOINT. This args field may be used in place of cmd even when using the default command executor. This change mirrors API and semantics changes in the Mesos CommandInfo protobuf message starting with version `0.20.0.` Either `cmd `or `args `must be supplied. It is invalid to supply both` cmd `and `args `in the same app.


一组字符串数组是另一种运行命令的模式。 这是出于对容器特性安全的使用。当使用默认的执行命令时这个args字段被使用。 从 `0.20.0.`版本开始， `cmd `和 `args` 必须被提供， 在同一个app中提供`cmd`和`args` 是无效的。


cpus (Float)

The number of CPU`s this application needs per instance. This number does not have to be integer, but can be a fraction.

这个应用程序的每个实例所占用的CPU。这个数字不一定是整数，但可以是一个分数。

mem (Float)

The amount of memory in MB that is needed for the application per instance.
应用程序实例需要的内存（MB）。

ports (Array of Integers)

An array of required port resources on the host.

一组数组表示主机所需要的端口资源。

The port array currently serves multiple roles:

目前这个端口数组代表多个角色


- The number of items in the array determines how many dynamic ports are allocated for every task.
   
  数组中的项的数量决定了有多少动态端口分配给每一个任务。

- For every port that is zero, a globally unique (cluster-wide) port is assigned and provided as part of the app definition to be used in load balancing definitions. See Service Discovery Load Balancing doc page for details
 

  
  Since this is confusing, we recommend to configure ports assignment for docker containers in container.docker.portMappings instead, see Docker Containers doc page).
  
   我们建议配置端口通过 docker 容器中的 container.docker.portMappings 赋值.  请查阅 Docker Containers doc page.
   
   Alternatively or if you use the Mesos Containerizer, pass zeros as port values to generate one or more arbitrary free ports for each application instance. Each port value is exposed to the instance via environment variables $PORT0, $PORT1, etc. Ports assigned to running instances are also available via the task resource.
  
   或者如果你使用 Mesos 容器,  通过零端口值给每个应用程序实例生成一个或多个任意空闲的端口. 每个实例端口的值通过环境变量$PORT0, $PORT1 等暴露. 端口运行的实例也可以通过任务分配的资源.
  
   We will probably provide an alternative way to configure this for non-docker apps in the future as well, see Rethink ports API.
   
   在将来, 我们可能会提供了一个替代的方法来配置这个non-docker应用, 查阅 Rethink ports API.
   
   requirePorts (Boolean)
   
   Normally, the host ports of your tasks are automatically assigned. This corresponds to the requirePorts value false which is the default.
   
    正常情况下, 你任务运行的主机端口被自动分配. requirePorts 的默认是 false .

   If you need more control and want to specify your host ports in advance, you can set requirePorts to true. This way the ports you have specified are used as host ports. That also means that Marathon can schedule the associated tasks only on hosts that have the specified ports available.

   如果你需要更多的控制并希望提前指指定你的主机端口, 你可以将 requirePorts 设置成 true. 通过这种方法, 你已经指定的端口被作为主机端口.  Marathon能够调度相关的任务, 仅在主机上指定的端口可用.

   The specified ports need to be in the local port range specified by the --local_port_min and --local_port_max flags. See Command Line Flags doc page).
   
   指定的端口必须在本地端口 the --local_port_min 和 --local_port_max的范围内. 请查阅 Command Line Flags doc page .

   instances (Integer)
   
   The number of instances of this application to start. Please note: this number can be changed everytime as needed to scale the application.

   应用程序启动的实例数目. 请注意：这一数字可以根据需求扩展应用程序的数目。

   executor (String)
   
    The executor to use to launch this application. Different executors are available. The simplest one (and the default if none is given) is //cmd, which takes the cmd and executes that on the shell level.
    
    executor 被用来发布应用程序. 其中最简单是 //cmd, 

container (Object)

  Additional data passed to the containerizer on application launch. These consist of a type, zero or more volumes, and additional type-specific options. Volumes and type are optional (the default type is DOCKER). In order to make use of the docker containerizer, specify --containerizers=docker,mesos to the Mesos slave. For a discussion of docker-specific options, see the native docker document.
  
   其他的数据通过容器传送在应用程序发布. 由一个type ,零个或者多个 volume 组成, 额外特定类型选取. volume 和 type 是可选的(在Docker中默认的是type). 为了使用 docker 容器,  指定--containerizers=docker, mesos.  关于 docker-specific 的选项, 请查看 native docker document.

env (Object with String values)

Key value pairs that get added to the environment variables of the process to start.

一对 key 和 value 被添加到环境变量在进程开始的时候.

constraints

Valid constraint operators are one of ["UNIQUE", "CLUSTER", "GROUP_BY"]. For additional information on using placement constraints see the Constraints doc page.

 有效的约束是一个["UNIQUE", "CLUSTER", "GROUP_BY"]. 附加的信息的约束差查阅 Constrations doc page .

acceptedResourceRoles v0.9.0

   Optional. A list of resource roles. Marathon considers only resource offers with roles in this list for launching tasks of this app. If you do not specify this, Marathon considers all resource offers with roles that have been configured by the --default_accepted_resource_roles command line flag. If no --default_accepted_resource_roles was given on startup, Marathon considers all resource offers.
   
可选. 一组资源的角色. Marathon只考虑在列表中的资源选项来发布app的任务. 如果你不指定, Marathon 认为所有的资源通过--default_accepted_resource_roles 命令行配置.  如果在启动的时候没有 --default_accepted_resource_roles, Marathon考虑所有的资源提供. 
Example 1: "acceptedResourceRoles": [ "production", "*" ] Tasks of this app definition are launched either on "production" or "*" resources.

例子1:  "acceptedResourceRoles": [ "production", "*" ] 这个应用程序定义的任务是"production"或"*"资源.

Example 2: "acceptedResourceRoles": [ "public" ] Tasks of this app definition are launched only on "public" resources

例子2: "acceptedResourceRoles":["public"]: 这个应用程序定义的任务只是“公共”资源

Background: Mesos can assign roles to certain resource shares. Frameworks which are not explicitly registered for a role do not see resources of that role. In this way, you can reserve resources for frameworks. Resources not reserved for custom role, are available for all frameworks. Mesos assigns the special role "*" to them.

Backrroud: Mesos可以将角色分配给特定的资源. Framworks 不能明确注册的角色不能看到资源. 以这种方式, 你可以储备资源框架. 资源不保留自定义角色,  可用于所有框架. Mesos分配特殊的角色"*".

To register Marathon for a role, you need to specify the --mesos_role command line flag on startup. If you want to assign all resources of a slave to a role, you can use the --default_role argument when starting up the slave. If you need a more fine-grained configuration, you can use the --resources' argument to specify resource shares per role. The Mesos master needs to be started with--roles` followed by a comma-separated list of all roles you want to use across your cluster. See the Mesos command line documentation for details.

一个角色注册Marathon. 你需要在请动的时候指定 --mesos_role 标记. 如果你希望将一个slave 的所有资源给一个角色, 你在slave启动的时候使用 --default_role 标记. 如果你需要更细粒度的配置, 你可以使用--resources 参数来指定每个角色的资源. 如果你想跨越你集群, Mesos master需要在开始时, --roles 后跟一个逗号分隔所有的角色. 关于更多的请查阅 the Mesos command line documentation.

 labels (Object of String values)
 
Attaching metadata to apps can be useful to expose additional information to other services, so we added the ability to place labels on apps (for example, you could label apps "staging" and "production" to mark services by their position in the pipeline).
 
 附加元数据的app暴露信息其他的服务可能是有用的, 所哟我们增加板块在 app中. (例如: 你能app中"staging"和"production"标记服务在pipeline中).
 
uris (Array of Strings)

URIs defined here are resolved, before the application gets started. If the application has external dependencies, they should be defined here.

这里定义的URls解析, 在应用程序开始之前. 如果应用程序外部依赖, 他们应该在这里被定义.
dependencies (Array of Strings)

A list of services upon which this application depends. An order is derived from the dependencies for performing start/stop and upgrade of the application. For example, an application /a relies on the services /b which itself relies on /c. To start all 3 applications, first /c is started than /b than /a.

 这个应用程序所依赖的服务列表. 命令来源于依赖, 开始,停止和升级一个应用程序. 例如: 一个应用程序 a 依赖 依赖于c的 b. 启动这三个程序. 首先启动c 然后是 b, 最后是a.
 
healthChecks

An array of checks to be performed on running tasks to determine if they are operating as expected. Health checks begin immediately upon task launch. For design details, refer to the health checks wiki page. By default, health checks are executed by the Marathon scheduler. In this case, the only supported protocol is COMMAND and each app is limited to at most one defined health check.

执行任务时执行的一系列检查，以确定它们是否按预期运行。健康检查任务启动后立即开始. 关于设计细节, 引用 wiki pag 的健康检查. 默认情况下, 健康检查被 Marathon 调度器执行. 在这种情况下, 只支持 COMMAND 协议, 每个应用程序仅仅限定于一个健康检查. 

An HTTP health check is considered passing if (1) its HTTP response code is between 200 and 399, inclusive, and (2) its response is received within the timeoutSeconds period.

 HTTP 检查检查中第一步是 HTTP 响应码是200和399之间, 第二步是在 timeoutSeconds 内收到响应. 
 
If a task fails more than maxConsecutiveFailures health checks consecutively, that task is killed causing Marathon to start more instances. These restarts are modulated like any other failing app by backoffSeconds, backoffFactor and maxLaunchDelaySeconds.

如果一个任务超过maxConsecutiveFailures健康检查连续失败, 这个任务被Marathon杀掉. 这些重启的调度机制就像任何其他应用程序失败一样,通过backoffSeconds, backoffFactor 和 maxLaunchDelaySeconds.

Health Check Options

command: Command to run in order to determine the health of a task. Note: only used if protocol == "COMMAND", and only available if Marathon is started with the `--executorhealth_checks` flag.

command: Command运行是为了确定一个任务的健康. 注意: 如果只使用协议命令protocol == "COMMAND", Marathon 开始 `--executorhealth_checks` 标记.

gracePeriodSeconds (Optional. Default: 15): Health check failures are ignored within this number of seconds of the task being started or until the task becomes healthy for the first time.

gracePeriodSeconds (默认是15):在任务开始的几秒钟内，健康检查失败被忽略, 或者任务第一次成为健康.

intervalSeconds (Optional. Default: 10): Number of seconds to wait between health checks.

intervalSeconds(默认是10): 等待健康检查的秒数. 

maxConsecutiveFailures(Optional. Default: 3) : Number of consecutive health check failures after which the unhealthy task should be killed.
 
maxConsecutiveFailures(默认 3): 连续的健康检查失败之后不健康的任务应该被杀掉.

protocol (Optional. Default: "HTTP"): Protocol of the requests to be performed. One of "HTTP", "TCP", or "Command".

protocol(默认是 "HTTP"):协议请求的形式,"HTTP", "TCP",或者"Command"

path (Optional. Default: "/"): Path to endpoint exposed by the task that will provide health status. Example: "/path/to/health". Note: only used if protocol == "HTTP".

path(选项默认是 "/"): 通过任务暴露路径的断点将提供健康状态. 例如: /path/to/health 注意:只使用如果协议= =“HTTP”。

portIndex (Optional. Default: 0): Index in this app's ports array to be used for health requests. An index is used so the app can use random ports, like "[0, 0, 0]" for example, and tasks could be started with port environment variables like $PORT1.

portIndex(默认是0) : 应用程序端口的索引可用于健康检查的需求. 使用一个索引, 这个应用程序可以使用随机的端口. 

timeoutSeconds (Optional. Default: 20): Number of seconds after which a health check is considered a failure regardless of the response.

timeoutSeconds(默认是20):一个健康检查被认为是一个失败的响应之后的秒数。

backoffSeconds, backoffFactor and maxLaunchDelaySeconds

Configures exponential backoff behavior when launching potentially sick apps. This prevents sandboxes associated with consecutively failing tasks from filling up the hard disk on Mesos slaves. The backoff period is multiplied by the factor for each consecutive failure until it reaches maxLaunchDelaySeconds. This applies also to tasks that are killed due to failing too many health checks.

当启动一个病态的app时配置补偿行为, 这可以防止在slave中沙箱与连续的失败任务充满u硬盘.这也适用于任务杀由于没有太多的健康检查。

upgradeStrategy

During an upgrade all instances of an application get replaced by a new version. The upgradeStrategy controls how Marathon stops old versions and launches new versions. It consists of two values:

在升级期间的所有实例应用程序会被新版本所取代. upgradeStrategy控制马拉松停止旧版本和发布新版本。它包含两个值:

minimumHealthCapacity (Optional. Default: 1.0) - a number between 0and 1 that is multiplied with the instance count. This is the minimum number of healthy nodes that do not sacrifice overall application purpose. Marathon will make sure, during the upgrade process, that at any point of time this number of healthy instances are up.

minimunHeslthCapacity(默认是:1.0): 0和1之间的数字乘以实例计数. 这是健康的节点的最小数量,不牺牲整个应用程序的目的. 马拉松将确保,在升级过程中,在任何时间点这个健康的实例数量。

maximumOverCapacity (Optional. Default: 1.0) - a number between 0 and 1 which is multiplied with the instance count. This is the maximum number of additional instances launched at any point of time during the upgrade process.

mxmumOverCapacity(默认是1.0):0和1之间的数字乘以实例计数.这是其他实例启动的最大数量在任何时候在升级过程的时间。

The default minimumHealthCapacity is 1, which means no old instance can be stopped before another healthy new version is deployed. A value of 0.5 means that during an upgrade half of the old version instances are stopped first to make space for the new version. A value of 0 means take all instances down immediately and replace with the new application.

minimumHealthCapacity 默认是1, 意思是在新的版本部署之前, 没有旧的版本被停止. 值为0.5时表示,在旧版本升级一半的实例停止为新版本腾出空间。值为0是表示, 立刻用新的实例换掉就的任务. 

The default maximumOverCapacity is 1, which means that all old and new instances can co-exist during the upgrade process. A value of 0.1 means that during the upgrade process 10% more capacity than usual may be used for old and new instances. A value of 0.0 means that even during the upgrade process no more capacity may be used for the new instances than usual. Only when an old version is stopped, a new instance can be deployed.

maximumOverCapacity默认是1: 这意味着所有新旧实例可以在升级过程中共存。值为0.1时表示,在升级过程能力比平时多10%可以用于新旧实例. 值为0.0时表示,即使是在升级过程中没有更多的能力可能会比平常使用的新实例。只有当一个老版本是停了下来,一个新实例可以部署。

If minimumHealthCapacity is 1 and maximumOverCapacity is 0, at least one additional new instance is launched in the beginning of the upgrade process. When it is healthy, one of the old instances is stopped. After it is stopped, another new instance is started, and so on.

如果 miniHealthCapacity是1 和 maximumOvercapacity是0, 至少有一个额外的新实例在升级过程的开始。当它是健康的,停止旧实例之一. 之后, 开始另一个新的实例. 

A combination of minimumHealthCapacity equal to 0.9 and maximumOverCapacity equal to 0 results in a rolling update, replacing 10% of the instances at a time, keeping at least 90% of the app online at any point of time during the upgrade.

结合minimumHealthCapacity等于0.9和maximumOverCapacity等于0的结果在一个滚动更新. 更换一次10%的情况下,保持至少90%的在线应用程序在任何时候在升级的时间。

A combination of minimumHealthCapacity equal to 1.0 and maximumOverCapacity equal to 0.1 results in a rolling update, replacing 10% of the instances at a time and keeping at least 100% of the app online at any point of time during the upgrade with 10% of additional capacity.

结合minimumHealthCapacity等于1.0和maximumOverCapacity等于0.1的结果在一个滚动更新,更换一次10%的实例并保持至少100%的在线应用程序在任何时间点升级期间10%的额外能力。








   

    

   



   
    





