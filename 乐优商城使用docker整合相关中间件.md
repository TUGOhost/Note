# 整合FastDFS
docker安装fastdfs镜像
> docker pull morunchang/fastdfs

运行tracker
> docker run -d --name tracker --net=host morunchang/fastdfs sh tracker.sh

运行storage,注意把xxx.xxx.xxx.xxx换成自己的EOS服务器IP
> docker run -d --name storage --net=host -e TRACKER_IP=xxx.xxx.xxx.xxx:22122 -e GROUP_NAME= morunchang/fastdfs sh storage.sh

进入storage容器内部
> docker exec -it storage  /bin/bash

修改nginx配置文件
> vim /data/nginx/conf/nginx.conf

添加如下内容
```conf
location /group1/M00 {
     proxy_next_upstream http_502 http_504 error timeout invalid_header;
     proxy_cache http-cache;
     proxy_cache_valid  200 304 12h;
     proxy_cache_key $uri$is_args$args;
     proxy_pass http://fdfs_group1;
     expires 30d;
 }
```
退出
> exit

重启storage
> docker restart storage

# 整合elasticsearch
拉取docker镜像
> docker pull docker.elastic.co/elasticsearch/elasticsearch:6.6.2

Run
> docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.6.2
