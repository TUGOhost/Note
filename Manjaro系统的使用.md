## 前言

自己使用Ubuntu都好几个月了，但是自己瞎弄导致dpkg一大多问题，并且vmware和vituralbox在ubuntu上使用的不是那么好，所以打算换个linux系统。arch也是在自己搞挂ubuntu的桌面中想到的，能不能整个干净点的系统，但是arch在虚拟机中安装的时候，完全是命令行，这有点浪费时间，索性就找下别的，manjaro进入我的视野，搜了下大多数的帖子也说他适合新手呢。

## 大部分工具的安装

``` shell
sudo pacman -Syy
sudo pacman-mirrors -i -c China -m rank # 选择中国源
sudo pacman -Syy
sudo pacman -S vim # 安装vim
sudo vim /etc/pacman.conf

sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring
sudo pacman -S fcitx-im fcitx-configtool fcitx-googlepinyin # 安装google拼音
sudo vim ~/.xprofile
# 输入以下内容：
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
sudo pacman -S visual-studio-code-bin # 安装vscode
sudo pacman -S make
sudo pacman -S screenfetch # 一个显示系统信息的工具，可以不用安装
sudo pacman -S clang
sudo pacman -S virtualbox # 安装 virtualbox，但是不知道为啥装完之后对于27寸 2k屏突然支持不好。
sudo pacman -S python # 安装python3
sudo pacman -S python2 # 安装python2
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py # 下载get-pip.py
sudo python get-pip.py # 获得pip3
# 在idea打开的时候fcitx无法使用中文的问题
vim $idea_path/idea.sh # $idea_path idea存放的路径/bin
# 输入以下内容
export XMODIFIERS=@im=fcitx
export QT_IM_MODULE=fcitx
sudo pacman -S yay git net-tools tree vim # 下载一些必备的工具
sudo yay -S vmware-workstation # 下载vmware Workstation，竟然在5.8上能够打开，惊奇！需要将内核调至5.7版本，要不然5.8版本会导致创建虚拟机或者打开虚拟机都失败的情况。
```

## 滚动更新与系统备份

滚动更新是arch系的一大特色。

关于系统备份：https://blog.lilydjwg.me/2013/12/29/rsync-btrfs-dm-crypt-full-backup.42219.html



系统更新使用命令：

> sudo pacman -Syyu

即可，每天一次防止滚挂。

## vmware 虚拟机没有网络连接

错误信息：

could not connect 'Ethernet0' to virtual network '/dev/vmnet0'

could not connect 'Ethernet0' to virtual network '/dev/vmnet8'

解决方法：

> sudo systemctl start vmware-networks.service