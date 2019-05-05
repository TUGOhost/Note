# 整合FastDFS
docker安装fastdfs镜像

> docker pull qbanxiaoli/fastdfs

> docker run -d --restart=always --privileged=true --net=host --name=fastdfs -e IP=47.100.191.203 -e WEB_PORT=88 -v ${HOME}/fastdfs:/var/local/fdfs qbanxiaoli/fastdfs

测试fastdfs是否搭建成功
> docker exec -it fastdfs /bin/bash 

> echo "Hello FastDFS!">index.html

> fdfs_test /etc/fdfs/client.conf upload index.html

# 整合elasticsearch
拉取docker镜像
> docker pull docker.elastic.co/elasticsearch/elasticsearch:6.6.2

Run
> docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.6.2

# 整合RabbitMQ
拉取docker镜像
> docker pull rabbitmq:3.6.15-management
运行
> docker run -d --hostname localhost --name myrabbit -p 15672:15672 -p 5672:5672 rabbitmq:3.6.15-management