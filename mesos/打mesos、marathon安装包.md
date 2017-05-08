# mesos 编译安装包
## 编译 centos-mesos
- 制作编译镜像

		vi Dockerfile
		
		FROM centos
		MAINTAINER prometheus zpang@dataman-inc.com

		# Install a few utility tools
		
		RUN yum install -y tar wget git &&  yum clean all

		# Fetch the Apache Maven repo file.
		
		RUN wget http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo -O /etc/yum.repos.d/epel-apache-maven.repo

		# Install the EPEL repo 
		
		RUN yum install -y epel-release  && yum clean all

		# Create a WANdisco SVN repo file to install the correct version:
		
		ADD repo /etc/yum.repos.d/wandisco-svn.repo

		# Parts of Mesos require systemd in order to operate. However, Mesos
		# only supports versions of systemd that contain the 'Delegate' flag.
		# This flag was first introduced in 'systemd version 218', which is
		# lower than the default version installed by centos. Luckily, centos
		# 7.1 has a patched 'systemd < 218' that contains the 'Delegate' flag.
		# Explicity update systemd to this patched version.
		
		RUN yum update systemd && yum clean all

		# Install essential development tools.

		RUN yum groupinstall -y "Development Tools" && yum clean all

		# Install other Mesos dependencies.
		
		RUN yum install -y apache-maven python-devel java-1.8.0-openjdk-devel zlib-devel libcurl-devel openssl-devel cyrus-sasl-devel cyrus-sasl-md5 apr-devel subversion-devel apr-util-devel && yum clean all

		RUN echo 'export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk' >> ~/.bash_profile

		# build package tools
		# install gems
		
		RUN yum install -y ruby ruby-devel ruby-rdoc rubygems rpm-build python-pip && yum clean all && \
				gem install fpm && \
				gem install json_pure && \
      			echo 'export MAINTAINER="system@dataman-inc.com"' >> ~/.bash_profile

		RUN mkdir -p /data/ && \
      			cd /data/ && \
        		git clone https://github.com/mesosphere/mesos-deb-packaging.git

		WORKDIR /data/mesos-deb-packaging

- 编译成镜像
		vi build.sh
		
		base_dir=$(cd `dirname $0` && pwd)
		cd $base_dir		
		docker build $opts -t demoregistry.dataman-inc.com/library/centos7-build-mesos .
		
- 执行编译
		vi run.sh
		
		docker rm -f build_mesos
		docker run -it --name build_mesos \
       	-e JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk \
       	-e MAKEFLAGS=-j8 \
       	-v /data:/dataman \
	        demoregistry.dataman-inc.com/library/centos7-build-mesos:20170325  \
	        build_mesos \
                --repo https://git-wip-us.apache.org/repos/asf/mesos.git?ref=1.2.0 \
                --nominal-version 1.2.0 \
                --build-version 0.1

## 参考
- mesos

		http://mesos.apache.org/gettingstarted/

## 系统编译

### ubuntu 

		apt-get install tar ruby ruby-dev wget git dpkg-dev automake gcc g++  build-essential rpm python-pip

		# Update the packages.
		$ sudo apt-get update

		# Install the latest OpenJDK.
		$ sudo apt-get install -y openjdk-7-jdk

		# Install autotools (Only necessary if building from git repository).
		$ sudo apt-get install -y autoconf libtool

		# Install other Mesos dependencies.
		$ sudo apt-get -y install build-essential python-dev python-boto libcurl4-nss-dev libsasl2-dev maven libapr1-dev libsvn-dev

		# 源码安装gem
		mkdir -p /data/tools
		cd /data/tools
		wget http://production.cf.rubygems.org/rubygems/rubygems-2.4.8.tgz &&\
		tar xzvf rubygems-2.4.8.tgz &&\
		cd rubygems-2.4.8 &&\
		ruby setup.rb

		# 修改gem源为淘宝源

		gem sources --remove https://rubygems.org/ 
		gem sources --remove http://rubygems.org/ 
		gem sources -a https://ruby.taobao.org/
		gem sources -l
		*** CURRENT SOURCES ***

		# pip切换到国内源
		
		mkdir -p ~/.pip
		cat > ~/.pip/pip.ini <<EOF
		[global]
		index-url = http://pypi.douban.com/simple
		EOF
	
		# 安装 gem 插件
		gem install fpm
		gem install json_pure

		# 设置fpm环境变量

		echo 'export MAINTAINER="system@dataman-inc.com"' >> ~/.bash_profile

		# 使用mesos-deb-packaging打mesos包

		# clone打包工具mesos-deb-packaging 

		cd /data/tools
		git clone https://github.com/mesosphere/mesos-deb-packaging.git
		cd mesos-deb-packaging

		# 修改 build_mesos 参数 deb包需要的依赖

		function deb_ {
  			local opts=( -t deb
               --deb-recommends zookeeper
               --deb-recommends zookeeperd
               --deb-recommends zookeeper-bin
               -d 'java-runtime-headless'
               -d libcurl3
               -d libsvn1
               -d libsasl2-modules 
               -d libcurl4-nss-dev # 添加的依赖包
               -d 'libstdc++6 > 5.1' # 添加的依赖包，需要把libstdc++6_5.1.0和gcc-5-base_5.1.0包放到dataman ubuntu repo里
               --after-install "$this/$asset_dir/mesos.postinst"
               --after-remove "$this/$asset_dir/mesos.postrm" )
  			rm -f "$this"/pkg.deb
  			fpm_ "${opts[@]}" -p "$this"/pkg.deb
		}

		# 编译打包
 
		./build_mesos --repo https://git-wip-us.apache.org/repos/asf/mesos.git?ref=0.23.0 --nominal-version 0.23.0 --build-version 0.1

# marathon 编译安装包

### 7.1 安装 sbt

#### ubuntu
```
echo "deb http://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-get update
sudo apt-get install -y sbt
```
#### centos
```
curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
sudo yum install -y sbt
```

### 7.2 使用marathon-pkg打包

git clone 源码及marathon-pkg

```
cd /data/rpmdata/
git clone https://github.com/mesosphere/marathon-pkg.git
cd marathon-pkg
git clone https://github.com/mesosphere/marathon.git
git submodule init
git submodule update
cd marathon
git checkout v0.9.0
```

修改./project/build.scala

```
125c125
<         case "log4j.properties"     => MergeStrategy.concat # 原内容
---
>         case "log4j.properties"     => MergeStrategy.first # 新内容
```

添加log4j.properties

```
cat > ./src/main/resources/log4j.properties <<EOF
log4j.rootLogger=INFO, stdout 

log4j.appender.stdout=org.apache.log4j.DailyRollingFileAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%d] %p %m %c:%L%n
log4j.appender.stdout.File=/var/log/marathon/marathon.log
EOF
```

#### centos
```
cd ..
make el 
```
#### ubuntu
```
make ubuntu
```