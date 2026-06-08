---
title: "ArchLinux+Windows单硬盘安装以及后续配置 UEFI+GPT"
date: 2024-05-10T10:38:14+08:00
draft: false
description: ""
---
{{< katex >}}


注：此教程针对的是UEFI+GPT分区表的环境下进行安装，目的是安装Windows10+ArchLinux双系统，且是单硬盘安装，本人的本地环境是intel + nvidia。

### 安装前确保

1. 本地已经安装好Windows10，且为arch linux分好一定空间
2. 已经用Rufus等写盘工作制作好arch linux启动u盘

### 用启动盘进入archiso

输入以下命令以增大字号

```bash
setfont ter-132n
```

输入以下命令以检测机器是否正常联网

```bash
ping archlinux.org -c 5
```

输入以下命令以查看计算机上的网络接口

```bash
ip -c a
```

输入以下命令以验证系统是否已在UEFI模式下启动

```bash
ls /sys/firmware/efi/efivars/
```

如果输出一大串，则说明成功以EFI模式启动

### 更新系统时间配置

输入以下命令来查看系统时间信息

```bash
timedatectl status
```

输入`timedatectl list-timezones`来列出所有国家和地区

在上述界面中按`q`以退出

以中国大陆为例，使用以下命令以更改时区设置

```bash
timedatectl set-timezone Asia/Shanghai
```

### 设置键盘布局

键盘默认布局为美式键盘en_US，基本满足需求

如需配置键盘，可执行以下步骤

输入以下命令以列出可用的键盘布局

```bash
ls /usr/share/kbd/keymaps/i386/qwerty
```

输入以下命令以载入键盘布局

```bash
loadkeys /usr/share/kbd/keymaps/i386/qwerty/us.map.gz
```

### 硬盘分区

输入以下命令以列出所有硬盘分区和挂载点

```bash
lsblk
```

sda开头的即为windows下的分区

输入以下命令以显示硬盘具体名称和信息

```bash
hdparm -i /dev/sda
```

输入`fdisk -l`可查看更多信息

输入以下命令来查看硬盘sda的所有分区，并进行创建分区

```bash
cfdisk /dev/sda
```

对于arch linux，我们需要建立三个分区，root和home和swap（efi和esp已经由windows创建）

分区如上图所示，从上到下分别是/，/home和swap分区，注意swap分区要把type改为linux swap

更改完成后选择write，将操作写入磁盘

接下来我们要注意格式化上述分区

输入以下命令来建立文件系统（格式化）：

```bash
mkfs.ext4 /dev/sda5
mkfs.ext4 /dev/sda6
mkswap /dev/sda7
swapon /dev/sda7
```

### 挂载分区（挂载给live usb环境以方便在live usb环境下通过chroot进入到主系统根目录）

输入以下命令以挂载分区

```bash
mount /dev/sda5 /mnt
mkdir /mnt/home
mount /dev/sda6 /mnt/home
```

输入`lsblk`以确认挂载情况

### 自动切换到快速源（可选）

备份mirrorlist

```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```

安装rankmirrors工具

```bash
pacman -Sy
pacman -S pacman-contrib
```

生成最快的10个服务器地址并载入配置文件

```bash
rankmirrors -n 10 /etc/pacman.d/mirrorlist.bak > /etc/pacman.d/mirrorlist
```

这会花几分钟的时间

如果这一步卡住了或者出了问题，可以输入以下命令回滚到初始配置

```bash
cp /etc/pacman.d/mirrorlist.bak /etc/pacman.d/mirrorlist
```

### 安装ArchLinux

安装archlinux到挂载到/mnt的根分区

```bash
pacstrap -i /mnt base base-devel linux linux-lts linux-headers linux-firmware intel-ucode [amd-ucode](amd的cpu) sudo nano vim git neofetch networkmanager dhcpcd pulseaudio [bluez](蓝牙模块) [wpa_supplicant](wlan)
```

此过程需要一定时间，请耐心等待

完成后

#### 生成文件系统表（FSTAB）

目前根目录被挂载到了/mnt, 但是当我们开机从主驱动器启动arch时，我们需要告诉系统将所有这些分区挂载到同一位置

输入以下命令来生成fstab文件

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

现在我们可以看到所有分区及其挂载点都已正确写入

### 系统配置

#### 进入安装好的arch linux的根目录

```bash
arch-chroot /mnt
```

#### 设置root密码

```bash
passwd
```

#### 建立一般用户

```bash
useradd -m light
passwd light
```

为一般用户加root权限

```bash
usermod -aG wheel,storage,power light
```

通过sudo执行root权限

```bash
visudo
```

更改后：

```bash
%wheel ALL=(ALL) ALL
Defaults timestamp_timeout=0
```

#### 设置系统语言

```bash
vim /etc/locale.gen
```

把需要的语言解除注释

生成语言locale

```bash
locale-gen
```

键入以下命令以生成区域设置

```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

键入以下命令以导出系统语言

```bash
export LANG=en_US.UTF-8
```

#### 设置主机名（host name）

```bash
echo ArchLinux > /etc/hostname
```

修改hosts文件内容

```bash
vim /etc/hosts
```

增加的新内容为：
```
127.0.0.1	localhost
::1			localhost
127.0.0.1	ArchLinux.localdomain	localhost
```

#### 设置时区或地区并与本地时间链接

```bash
ln -sf /usr/share/zoneinfo/
```

按tab tab找到所在地区Asia/Shanghai

补全命令

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

同步时钟

```bash
hwclock --systohc
```

### 安装grub

sda1是efi分区，grub将会被安装到这里

建立efi文件夹并挂载

```bash
mkdir /boot/efi
mount /dev/sda1 /boot/efi/
```

安装ntfs-3g以防后续引导不了windows

```bash
pacman -S ntfs-3g
```

安装grub及引导相关软件包

```bash
pacman -S grub efibootmgr dosfstools mtools
```

修改grub配置

```bash
vim /etc/default/grub
```

如图所示，将最后一行取消注释

安装os-prober

```bash
pacman -S os-prober
```

使用一些参数安装grub

```bash
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```

生成grub的config文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

如果没有绿色这一行的话可以等会进arch后再安装ntfs-3g并重新生成grub config来修复

### 启动网络服务

```bash
systemctl enable dhcpcd.service
systemctl enable NetworkManager.service
```

### 回到archiso环境

```bash
exit
```

卸载所有分区

```bash
umount -lR /mnt
```

重启并取出u盘

```bash
reboot
```

### 修复windows引导

```bash
sudo pacman -S ntfs-3g
sudo pacman -S nvidia-lts
sudo mount /dev/sda1 /boot/efi
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

至此windows+arch双系统制作完成

### 安装GUI（KDE plasma）

更新pacman数据库

```bash
sudo pacman -Sy
```

安装xorg和plasma和sddm

```bash
sudo pacman -S xorg xorg-xinit xterm plasma plasma-desktop [plasma-wayland-session] kde-applications kdeplasma-addons sddm
```

时间会比较长，请耐心等待

配置.xinitrc

```bash
# .xinitrc
exec startkde
```

启用sddm

```bash
sudo systemctl enable sddm.service
```

重启

```bash
reboot
```

安装firefox等其他软件包

```bash
pacman -S firefox gimp htop bpytop
```

### 其他重要配置

#### 换源并安装yay（aur包管理器）

```bash
# /etc/pacman.d/mirrorlist
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/\(repo/os/\)arch
```
```bash
sudo pacman -Syyu
```

```bash
# /etc/pacman.conf
[archlinuxcn]
# The Chinese Arch Linux communities packages.
# SigLevel = Optional TrustedOnly
SigLevel = Optional TrustAll
# 官方源
Server   = http://repo.archlinuxcn.org/$arch
# 163源
Server = http://mirrors.163.com/archlinux-cn/$arch
# 清华大学
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

注意以上源只能添加一个

```bash
sudo pacman -Sy
sudo pacman -S archlinuxcn-keyring
sudo pacamn -S yay
```

#### 安装clash

```bash
yay -S clash-premium-bin clash-verge
```

安装qq

```bash
yay -S linuxqq
```

安装nvidia驱动

```bash
sudo pacman -S nvidia [nvidia-lts]
```

安装alsamixer更好地使用耳机

```bash
sudo pacman -S alsa-utils
# 解除耳机禁音后
alsactl --file ~/.config/asound.state store
# resound.sh
#! /bin/bash
alsactl --file ~/.config/asound.state restore
```

安装剪贴板（i3）

```bash
sudo pacman -S xclip
```

安装中文字体

```bash
sudo pacman -S wqy-zenhei
```

安装中文输入法

```bash
sudo pacman -S fcitx5 fcitx5-im fcitx5-chinese-addons
```

设置环境变量

```bash
# /etc/environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
INPUT_METHOD=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

配置完成后重启生效，然后通过fcitx5-configtool添加pinyin即可

安装kitty

```bash
sudo pacman -S kitty
```

注：kitty可能调用不了中文输入法，可以先设置上面的环境变量或者修改kitty的配置文件，让它开启时调用输入法？（我有点想不起来了，具体看archlinux wiki里对kitty的描述）

安装paru

```bash
sudo pacman -S paru
```

安装nerd font

```bash
sudo pacman -S ttf-meslo-nerd
# 可以用pacman -Ss看仓库里都有些啥
```

##### konsole可以用glassy主题

安装网易云音乐

```bash
yay -S netease-cloud-music go-musicfox 
```

安装qq音乐

```bash
yay -S qqmusic
```

开启i386支持（好像没啥用把）

```bash
sudo dpkg --add-architecture i386
```

通过wine安装网易云

```bash
sudo pacman -S wine
# 然后找网易云音乐的exe安装包安装
# wine [exe文件名]
```

这个我用的应该是deepin的wine（deepin-wine）我先安装了32位版的微信，然后安装的网易云音乐，莫名其妙就不报错了，暂时我还不清楚是怎么回事

知道了，用的是wine-for-wechat

一般可以通过winecfg加入atl100 mlang msls31 riched20 usp10 msvcp60 riched32 等函数来解决报错问题

wine装网易云前需要把需要的字体全部装上

```bash
pacman -S adobe-source-han-serif-cn-fonts noto-fonts-cjk adobe-source-han-sans-cn-fonts powerline-fonts ttf-font-awesome wqy-bitmapfont wqy-microhei wqy-microhei-lite wqy-zenhei adobe-source-code-pro-fonts [ttf-apple-emoji]
```

顺便提一嘴，网上说的各种解决wine乱码的办法都不如这个安装字体来的简单且有效

安装ranger

```bash
sudo pacman -S ranger
```

安装cava（cava依赖pulseaudio）

```bash
sudo pacman -S cava
```

安装chrome

```bash
yay -S google-chrome
```

安装微软字体

```bash
yay -S ttf-ms-fonts
```

安装百度网盘

```bash
yay -S baidunetdisk-bin
```

kitty配置

```bash
# ~/.config/kitty/kitty.conf

background_opacity	0.7
font_family			MesloLGL Nerd Font
bold_font			auto
italic_font			auto
bold_italic_font	auto
```

cava配置

```bash
# ~/.config/cava/config

gradient = 1
gradient_count = 2
gradient_color_1 = '#2864FF'
gradient_color_2 = '#C620FF'
```

安装gparted

```bash
sudo pacman -S gparted
```

安装并配置neovim

```bash
sudo pacman -S neovim
git clone https://github.com/lightmon233/nvim.git ~/.config/nvim
nvim
```

截图工具

```bash
yay -S ksnip shotgun
```

KDE全局主题推荐

```bash
Plasma-Overdose
```

zsh及oh-my-zsh安装与配置

zsh

```bash
sudo pacman -S zsh
```

oh-my-zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

powerlevel10k(主题)

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \({ZSH_CUSTOM:-\)HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

Set `ZSH_THEME="powerlevel10k/powerlevel10k"` in `~/.zshrc`.

插件：

自动补全

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

```bash
# ~/.zshrc
plugins=( 
    # other plugins...
    zsh-autosuggestions
)
```

语法高亮

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

```bash
# ~/.zshrc
plugins=( [plugins...] zsh-syntax-highlighting)
```

linux和windows双系统时间不同步问题解决
```bash
timedatectl set-local-rtc 1 --adjust-system-clock
```
解决chrome及其他应用程序的emoji字体显示为方块的问题
```bash
sudo pacman -S noto-fonts-emoji
```

v2raya
```
yay -S v2raya
```
```
sudo systemctl enable --now v2raya.service
```
