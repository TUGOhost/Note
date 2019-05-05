# tomcat+Java+Maven在centos7中安装

### tomcat安装

我这里安装tomcat7，8或者其他版本同。

https://tomcat.apache.org/download-70.cgi?Preferred=http%3A%2F%2Fapache.claz.org%2F

```shell
mkdir - p ~/java
cd ~/java
wget http://apache.claz.org/tomcat/tomcat-7/v7.0.90/bin/apache-tomcat-7.0.90.tar.gz
tar apache-tomcat-7.0.90.tar.gz
```

### JDK安装

看这篇[博文](https://blog.csdn.net/fenglailea/article/details/26006647)

```shell
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.rpm
然后命令直接安装即可（yum执行是当前目录下的 *.rpm）
sudo yum install jdk-8u181-linux-x64.rpm
```

#### JAVA JDK 安装位置查找

```shell
which java
```

![](image/109.png)

继续,查看该软连接，链接位置

```shell
ls -ltr /usr/bin/java
```

![](image/110.png)

从上面看，怎么看都不像JDK目录，那么可能还是软连接，继续查找

```shell
ls -lst /etc/alternatives/java
```

![](image/111.png)

此目录 位置就是我们要要的JDK  JAVA 安装目录为(每个JDK的位置不一样的)

```shell
usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/bin/java
```

#### 查看环境变量是否设置

```shell
echo $JAVA_HOME
```

#### 设置

```shell
vim ~/.bashrc
```

在最后一行添加

```shell
export JAVA_HOME=/usr/java/jdk1.8.0_181-amd64
export JRE_HOME=$JAVA_HOME/jre
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

然后输入`javac`看看是否出现如下

![](image/112.png)

### Maven安装

编译后代码安装

下载地址：http://maven.apache.org/download.cgi

选择  Binary tar.gz archive

```shell
wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar zxvf apache-maven-3.3.9-bin.tar.gz
sudo mv apache-maven-3.3.9 /usr/java
```

设置环境变量文件

```shell
vim /etc/profile.d/maven.sh
```

设置环境变量

```shell
#!bin/bash
export MAVEN_HOME=/usr/java/apache-maven-3.3.9
export M2_HOME=$MAVEN_HOME
export PATH=$MAVEN_HOME/bin:$PATH
```

应用生效

```shell
. /ect/profile
. /etc/bashrc
```

![](image/113.png)

#### Maven更改国内镜像设置

```shell
vim /usr/java/apache-maven-3.3.9/conf/settings.xml
```

找到mirrors，修改或增加如下

```xml
 <mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  </mirrors>
```

然后参考这个[博客](https://blog.0x233.cn/2017/02/03/%E9%98%BF%E9%87%8C%E4%BA%91Maven%E9%95%9C%E5%83%8F%E4%BD%BF%E7%94%A8%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%A7%BF%E5%8A%BF/)来更改maven设置。

### mysql安装
由于作为测试服务器使用，所以使用yum快速安装。如下：

yum install mariadb mariadb-server
systemctl start mariadb   #启动mariadb
systemctl enable mariadb  #设置开机自启动
mysql_secure_installation #设置root密码等相关
mysql -uroot -p           #测试登录