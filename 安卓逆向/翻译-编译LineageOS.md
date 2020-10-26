## 系统：

Ubuntu18.04

vmware

## 环境搭建

### 安装SDK

https://dl.google.com/android/repository/platform-tools-latest-linux.zip

执行命令：

> unzip platform-tools-latest-linux.zip -d ~

编译~/.profile文件中添加：

```
if [ -d "$HOME/platform-tools" ] ; then
PATH="$HOME/platform-tools:$PATH"
fi
```

再执行`source ~/.profile`

安装必要的依赖包

```
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev openjdk-8-jdk python
```

### 创建目录

```shell
mkdir -p ~/bin
mkdir -p ~/android/lineage
```

### 安装repo

```shell
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

chmod a+x ~/bin/repo
```

将repo设置为环境变量，编辑~/.profile

```
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```

执行`source ~/.profile`

### 同步LIneage OS

```
cd ~/android/lineage

git config --global user.email "you@example.com"
git config --global user.name "Your Name"

repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/lineageOS/LineageOS/android.git -b lineage-17.1 --repo-url=https://gerrit-googlesource.lug.ustc.edu.cn/git-repo 
```

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

```shell
repo sync
```

这将根据网速来判断需要花费多长时间。



## 参考

- https://wiki.lineageos.org/devices/sailfish/build
- https://mirrors.tuna.tsinghua.edu.cn/help/lineageOS/