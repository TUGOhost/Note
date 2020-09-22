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
sudo yay -S vmware-workstation # 下载vmware Workstation，竟然在5.8上能够打开，惊奇！

```

```text
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```