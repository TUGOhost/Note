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
> docker run -it --privileged=true -v ulineageos:/root ubuntu:18.04 /bin/bash

docker容器中需要用到`mount`命令，所以需要将`--privileged=true`。

ulineageos是本地的一个目录名，挂在到docker容器中的/root目录

docker容器内的命令：

> apt-get update
>
> apt-get upgrade -y
>
> apt install -y wget vim p7zip p7zip-rar p7zip-full bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev openjdk-8-jdk python file nano screen sudo tig python3-pip python-protobuf
>
> pip3 install six google protobuf
>
> cd ~/
>
> wget https://dl.google.com/android/repository/platform-tools-latest-linux.zip
>
> 7z x platform-tools-latest-linux.zip
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
> repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/lineageOS/LineageOS/android.git -b lineage-17.1 --repo-url=https://gerrit-googlesource.lug.ustc.edu.cn/git-repo 

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

自己实践了下，虽然网速够快，但是清华源的一些东西都不是很全，所以如果能够进外网还是使用官方的教程来进行。

## 编译

> source build/envsetup.sh
>
> breakfast sailfish

然后就是提取blobs了，我这里使用基于OTA的方式来提取，其他的也可以参考官网。

> wget https://mirrorbits.lineageos.org/full/sailfish/20201212/lineage-17.1-20201212-nightly-sailfish-signed.zip
>
> sha256sum 20201212/lineage-17.1-20201212-nightly-sailfish-signed.zip
>
> unzip 20201212/lineage-17.1-20201212-nightly-sailfish-signed.zip payload.bin
>
> git clone https://github.com/LineageOS/scripts
>
> python scripts/update-payload-extractor/extract.py payload.bin --output_dir ./
>
> mkdir system/
> mount system.img system/
> mount vendor.img system/vendor/
> mount product.img system/product/

docker容器里执行`mount system.img system/` 会提示  `mount: system/: mount failed: Operation not permitted.`

待解决，感觉像是docker容器权限的问题。`--privileged=true`即可解决。

回到源代码的根目录中执行`extract-files.sh`

> ./extract-files.sh ~/android/system_dump/
>
> ```
> sudo umount -R ~/android/system_dump/system/
> rm -rf ~/android/system_dump/
> ```

**[100% 139/139] out/soong/.bootstrap/bin/soong_build out/soong/build.ninja** 
FAILED: out/soong/build.ninja 
out/soong/.bootstrap/bin/soong_build -t -l out/.module_paths/Android.bp.list -b out/soong -n out -d out/soong/build.ninja.d -globFile out/soong/.bootstrap
/build-globs.ninja -o out/soong/build.ninja Android.bp 
error: vendor/google/sailfish/Android.bp:323:1: module "TimeService" variant "android_common": module source path "vendor/google/sailfish/proprietary/vend
or/app/TimeService/TimeService.apk" does not exist 
error: vendor/google/sailfish/Android.bp:301:1: module "RCSBootstraputil" variant "android_common": module source path "vendor/google/sailfish/proprietary
/vendor/app/RCSBootstraputil/RCSBootstraputil.apk" does not exist 
error: vendor/google/sailfish/Android.bp:312:1: module "SecureExtAuthService" variant "android_common": module source path "vendor/google/sailfish/proprie
tary/vendor/app/SecureExtAuthService/SecureExtAuthService.apk" does not exist 
error: vendor/google/sailfish/Android.bp:411:1: module "shutdownlistener" variant "android_common": module source path "vendor/google/sailfish/proprietary
/vendor/app/shutdownlistener/shutdownlistener.apk" does not exist 
error: vendor/google/sailfish/Android.bp:243:1: module "WfcActivation" variant "android_common": module source path "vendor/google/sailfish/proprietary/pr
oduct/priv-app/WfcActivation/WfcActivation.apk" does not exist 
FAILED: [W][2020-12-13T12:08:14+0000][75458] void cmdline::logParams(nsjconf_t *)():250 Process will be UID/EUID=0 in the global user namespace, and will 
have user root-level access to files 
[W][2020-12-13T12:08:14+0000][75458] void cmdline::logParams(nsjconf_t *)():260 Process will be GID/EGID=0 in the global user namespace, and will have gro
up root-level access to files 
12:08:43 soong bootstrap failed with: exit status 1 

\#### failed to build some targets (32 seconds) ####

> ```
> croot
> brunch sailfish
> ```

## 参考

- https://www.cnblogs.com/luoyesiqiu/p/10701419.html
- https://wiki.lineageos.org/devices/sailfish/build
- https://mirrors.tuna.tsinghua.edu.cn/help/lineageOS/
- https://github.com/stucki/docker-lineageos
- https://wiki.lineageos.org/extracting_blobs_from_zips.html





一个隐藏root、xposed、frida的系统，基于lineageos