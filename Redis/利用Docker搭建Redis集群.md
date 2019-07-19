# Redis集群搭建

## 运行Redis镜像

分别使用以下命令启动3个Redis

> docker run --name redis-6379 -p 6379:6379 -d hub.c.163.com/library/redis
> docker run --name redis-6380 -p 6380:6379 -d hub.c.163.com/library/redis
> docker run --name redis-6381 -p 6381:6379 -d hub.c.163.com/library/redis

使用`docker ps`命令，查看是否启动成功
![](D:/note/image/324.png)

## 配置Redis集群

分别使用`dokcer inspect 容器ID`命令，查看3个Redis内网IP地址
![](D:/note/image/325.png)
在Networks栏，可以看见该容器的Docker内网IP地址。

> redis-6379：172.17.0.2:6379
> redis-6380：172.17.0.3:6379
> redis-6381：172.17.0.4:6379

进入Docker容器内部

> 使用redis-6379为主机，其余两台为从机
> 使用 docker exec -ti 容器ID /bin/bash 分别进入三个Redis容器
> 进入容器后，使用 redis-cli 命令，连接redis服务端
> 连接服务后，使用 info replication 查看当前机器的角色
> 未配置前，三台redis均为 master主机

使用上面的方法，分别进入 redis-6379、redis-6380、redis-6381容器内部，并连接redis服务端

> 分别在redis-6380和redis-6381使用 SLAVEOF 172.17.0.2 6379 命令
> 在redis-6379 使用 info replication 命令，验证主从关系是否配置成功

![](D:/note/image/326.png)
这样，redis的集群环境就搭建好了，本机测试无问题，这里就不演示了。为了保证redis集群的高可用，下面开始配置redis哨兵模式。

# Redis哨兵模式

## 配置Redis哨兵

Redis哨兵配置，有两种方案

> 方案一：基于现有的3台Redis容器服务，互相启动一个Redis哨兵
> 方案二：重新再启动3台Redis容器服务，分别启动一个Redis哨兵

方案二会额外的新增3个Redis容器服务，所以这里演示方案一

分别进入3台Redis容器内部，执行以下操作
首先，进入Docker容器内部

> 使用 docker exec -ti 容器ID /bin/bash 分别进入三个Redis容器

然后，编写Redis哨兵配置文件

> 使用 cd / 命令，进入根目录

使用`touch sentinel.conf`命令，创建哨兵配置文件

在进行编辑时，需要先安装vim，命令为`apt-get update` ,`apt-get install vim`
使用 vim 命令编辑 sentinel.conf 文件，
添加以下内容 

> sentinel monitor host6379 172.17.0.2 6379 1

最后，启动Redis哨兵

使用`redis-sentinel /sentinel.conf`启动Redis哨兵监控
使用`ps –ef |grep redis`命令，可以看到redis-server和redis-sentinel正在运行

至此，哨兵模式配置完毕。