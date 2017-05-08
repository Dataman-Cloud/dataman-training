# 升级步骤
## mesos 系列升级
- 最新 release
    - mesos
        - 下载
        
            https://open.mesosphere.com/downloads/mesos/#apache-mesos-1.0.1
        - 验证
            
                sha256sum mesos-1.0.1-2.0.93.centos701406.x86_64.rpm        
    - marathon 
        - 下载
        
                # 查看版本
                rpm -Uvh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
                
                # 查看版本
                yum install marathon
                
                # 登录asia机器
                ssh -A asia
                sudo su -
                cd /data/mesosphere-rpm/marathon
                # 按照之前版本获取
                wget http://repos.mesosphere.com/el/7/x86_64/RPMS/marathon-XXX.el7.x86_64.rpm
                # 将 rpm 转移到 repo 主机
                # 登录 repo 主机(ucloud 生产跳板－> repo 主机)
                cd /data/nginx/repos/centos/7/0
                ＃ 将包放到这个目录后                    
        - 验证   
                
                md5sum marathon-XXX.el7.x86_64.rpm
## 升级数人云 repo
- 登录 repo 机器
    - 登录 ucloud 生产跳板
    - 登录 repo 机器
         - `ssh 123.59.42.206`
    - 切换 root
        - `sudo su -` 
        - `cd /data/download`    
    - 下载
    - 将包转移到对应目录
        - `mv mesos-1.0.1-2.0.93.centos701406.x86_64.rpm /data/nginx/repos/centos/7/0/`
    - 生成新repo list
        - `createrepo --update /data/nginx/repos/centos/7/0/`
        