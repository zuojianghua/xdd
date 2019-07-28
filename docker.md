# Docker学习笔记 stack & swarm

10台以下机器可使用docker swarm & portainer进行管理. 10台以上建议还是用k8s

## docker环境安装 centos7.x

### docker安装

```/bin/bash
# 删除旧版
sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine

# 更新源
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2

sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo

# 安装
sudo yum install docker-ce docker-ce-cli containerd.io

# 启动
sudo systemctl start docker

# 设置为开机启动
sudo systemctl enable docker
```

### swarm网络模式启用

```/bin/bash
# 启动swarm网络模式, 执行此命令的机器默认成文管理节点
docker swarm init

# 查看加入普通节点命令
docker swarm join-token worker

# 查看加入管理员命令
docker swarm join-token manager
```

```/bin/bash
# 如果有多台服务器, 加入到swarm网络
docker swarm join --token xxxxxxxx xxx.xxx.xxx.xxx:2377
```

### 指定节点名称, 某些服务可以只安装在固定节点上, 默认采用管理节点

```/bin/bash
# 查看已加入的节点, 及ID
docker node ls

# 将某台机器指定为中间件节点
docker node update --label-add midware=midware {node id}

# 查看节点label是否添加成功
docker inspect {node id}
```

## 服务部署
### 常用镜像

```/bin/bash
docker pull nginx
docker pull tomcat:8.5
docker pull redis
docker pull foxiswho/rocketmq:server-4.3.2
docker pull foxiswho/rocketmq:broker-4.3.2
docker pull zookeeper:latest
docker pull fabric8/java-centos-openjdk8-jre:latest
docker pull portainer/portainer
```

### 部署portainer，便于可视化管理

Step1 主机安装管理工具

```/bin/bash
# 安装portainer在集群的管理节点服务器，只需要安装一个
docker volume create portainerData
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainerData:/data portainer/portainer
```

Step2 从机安装管理工作代理

```/bin/bash
# 安装portainer代理在集群的工作节点，每个工作节点服务器需要安装一个
# 本目录下已经事先下载好了agent-stack.yml也可以直接使用
curl -L https://downloads.portainer.io/agent-stack.yml -o agent-stack.yml && docker stack deploy --compose-file=agent-stack.yml portainer-agent
```

Step3 进入portainer的endpoint页面，增加对集群内工作节点的代理管理

Endpoints > Add endpoint > 类型选择Agent > 填写name和Endpoint URL

Endpoint URL为```xxx.xxx.xxx.xxx:9001```从机IP + 9001端口号，请保证丛机9001端口没有被占用可以通过内网访问

### 部署

将服务配置整理为docker-stack.yml文件

```/bin/bash
docker stack deploy -c docker-stack.yml xxx
# 如需重新部署可直接再次执行
```

同一个stack的yml可以分开写，例如

* docker-stack-web.yml
* docker-stack-midware.yml
* docker-stack-workflow.yml
等等，保证在同一个swarm网络下即可

### 如需卸载服务

```/bin/bash
docker stack rm xxx
```

### 重启或更新指定的服务

```/bin/bash
# 每隔30秒依次强制重启1个redis的服务container
docker service update --force --update-parallelism 1 --update-delay 30s redis
```

## 一些奇奇怪怪的问题处理记录

### zookeeper无法启动

报java.io.IOException: Unreasonable length = 8688650 错误

清理zookeeper的日志目录, ```zkData-datalog:/datalog```  ```zkData-logs:/logs``` 这两个挂载目录, 可能是上次删除服务后遗留的文件有问题导致

### 查看服务器磁盘使用情况

```/bin/bash
# 查看docker容器占用的空间大小
docker system df

# 清理释放无用的docker文件
docker system prune

# 查看某目录下的文件大小
du -hs /var/lib/docker
du -hs /root/xxx  
```

### 服务器上编译前端代码内存溢出

```/bin/bash
# 调整编译脚本中的max_old_space_size值
npm run color-less && node --max_old_space_size=5048 ./node_modules/@angular/cli/bin/ng build --aot --base-href ./
```

### swarm节点显示down不能使用

**处理方法**

* 删除/etc/docker/daemon.json 或增加``` { "storage-driver": "devicemapper" } ```
* vim /etc/sysconfig/docker 增加``` OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false' ```
* 重启从机
* 主机上执行 ```docker node rm 从机的nodeid```
* 从机上执行 ```docker swarm leave -f```
* 如果不能正常退出swarm则删除/var/lib/docker/swarm 目录
* 重启docker服务 ```systemctl restart docker.service```
* 主机上查看加入swarm的token ```docker swarm join-token worker```
* 从机上重新加入swarm ```docker swarm join --token ... ```


## 相关文档

* [docker centOS安装] https://docs.docker.com/install/linux/docker-ce/centos/
* [docker stack] https://docs.docker.com/engine/reference/commandline/stack/
* [portainer 管理工具] https://www.portainer.io/installation/
