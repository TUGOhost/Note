# 编译AOSP

## 前言

因为自己本机的ubuntu无法进行编译，出现各种bug，而且网上的解决方案也是参差不齐、五花八门的，所以想通过docker来搭建一个编译环境，让整个过程变得简单一点。





# 参考

- [docker-aosp](https://github.com/kylemanna/docker-aosp)

- [史上最简单Android源码编译环境搭建方法](http://weishu.me/2016/12/30/simple-way-to-compile-android-source/)





# 编译LineageOS

## 环境

docker ubuntu18.04

> docker pull ubunut:18.04
>
> docker run -it -v ulineageos:/root ubuntu:18.04 /bin/bash

ulineageos是本地的一个目录名，挂在到docker容器中的/root目录

docker容器内的命令：

> apt-get update
>
> apt-get install wget
>
> cd ~/
>
> wget https://dl.google.com/android/repository/platform-tools-latest-linux.zip
>
> apt install p7zip p7zip-rar p7zip-full -y
>
> 7z x platform-tools-latest-linux.zip
>
> apt install vim
>
> vim ~/.profile

`~/.profile`中添加

```
if [ -d "$HOME/platform-tools" ] ; then
PATH="$HOME/platform-tools:$PATH"
fi
```

> source ~/.profile
>
> apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev openjdk-8-jdk python
>
> mkdir -p ~/bin
>
> mkdir -p ~/android/lineage
>
> curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
>
> chmod a+x ~/bin/repo

将repo设置为环境变量，编辑~/.profile

```
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

> source ~/.profile

## 同步Lineage OS


> cd ~/android/lineage
>
> git config --global user.email "you@example.com"
> git config --global user.name "Your Name"
> 
> repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/lineageOS/LineageOS/android.git -b lineage-15.1 --repo-url=https://gerrit-googlesource.lug.ustc.edu.cn/git-repo 

修改`.repo/manifests/default.xml`，将

```xml
  <remote  name="github"
           fetch=".."
           review="review.lineageos.org" />
```

改成

```xml
  <remote  name="github"
           fetch="https://github.com/" />

  <remote  name="lineage"
           fetch="https://mirrors.tuna.tsinghua.edu.cn/git/lineageOS/"
           review="review.lineageos.org" />
```

将

```xml
  <remote  name="aosp"
           fetch="https://android.googlesource.com"
```

改成

```xml
  <remote  name="aosp"
           fetch="https://mirrors.tuna.tsinghua.edu.cn/git/AOSP"
```

将

```xml
  <default revision="..."
           remote="github"
```

改成

```xml
  <default revision="..."
           remote="lineage"
```

同步源码树（以后只需执行这条命令来同步）：

> #防止文件太大同步过程中导致EOF
> git config --global http.postBuffer 524288000
> repo sync

这将根据网速来判断需要花费多长时间。

## 编译

> source build/envsetup.sh
>
> breakfast sailfish



## 参考

- https://www.cnblogs.com/luoyesiqiu/p/10701419.html
- https://wiki.lineageos.org/devices/sailfish/build
- https://mirrors.tuna.tsinghua.edu.cn/help/lineageOS/
- https://github.com/stucki/docker-lineageos





一个隐藏root、xposed、frida的系统，基于lineageos