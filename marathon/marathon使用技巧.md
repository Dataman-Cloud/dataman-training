#marathon使用技巧

## 1. API
### 重启 docker host模式任务，其中scale为tasks数量
    curl -XDELETE  http://marathon_ip:8080/v2/apps/${appid}/tasks?scale=tasks数量
    示例：
    curl -XDELETE  http://10.3.33.6:8080/v2/apps/dataman-nginx-test-bridge/tasks?scale=1