# 准备安装介质
## 下载arch linux

使用清华源下载arch linux
清华源[下载链接](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/)

## 刻录系统镜像

### Windows/macOS系统

下载balenaEtcher[官方下载链接](https://www.balena.io/etcher/)
Windows也可以使用rufus[下载链接](http://rufus.ie)
不建议使用UltraISO

### Linux系统

下载balenaEtcher[官方下载链接](https://www.balena.io/etcher/)
运行以下命令找到你的U盘

```bash
lsblk
```

运行以下命令刻录Linux系统

```bash
dd bs=4m if=/dev/sdx of=/dir/archlinux.iso
```

注：sdx为你的U盘
/dir/archlinux.iso为下载路径，必须为绝对路径，如：

```
/home/username/Downloads/archlinux-2020.10.01-x86_64.iso
```

一定不要使用：

```
~/Downloads/archlinux-2020.10.01-x86_64.iso
```

# 进Bios安装arch linux

## 连接网络

如果电脑支持Wi-Fi，可以运行以下命令（最新版的archlinux好像移除了相关驱动）：

```bash
wifi-menu
```

如果不支持Wi-Fi，可以用网线连接电脑（博主是用安卓手机USB网络共享连接电脑），然后运行以下命令：

```bash
dhcpcd
```

检测网络连接

```bash
ping https://mirrors.tuna.tsinghua.edu.cn
```

## 设置时区

```bash
timedatectl set-ntp true
```

## 磁盘分区

运行以下命令找到你的硬盘

```bash
lsblk
```

机械硬盘一般是/dev/hdx
SATA固态硬盘一般是/dev/sdx
NVME固态硬盘一般是/dev/nvmexn1

```bash
fdisk /dev/mydisk
```

其中mydisk是你的硬盘
fdisk常用命令
| 命令 | 操作 |
|--|--|
| g | 转化为GPT格式，常见于UEFI启动 |
|o|转化为dos格式，常见于LEGACY启动|
|m|帮助|
|n | 新建分区|
|d |删除分区 |
|w |保存数据 |
|q |退出但不保存 |
### 主板支持UEFI引导
|分区|挂载点|建议大小|格式|
|-|-|-|-|
|efi|/boot/efi|300M|vfat|
|boot|/boot|500M|ext4|
|根分区|/|20G|ext4|
|家目录|/home|剩余空间|xfs|
|交换分区|-|10G|[swap]|
### 主板不支持UEFI引导
|分区|挂载点|建议大小|格式|
|-|-|-|-|
|根分区|/|20G|ext4|
|家目录|/home|剩余空间|xfs|
|交换分区|-|10G|[swap]|
### 格式化分区
格式化efi分区
```bash
mkfs.vfat /dev/mydisk1
```
格式化boot分区
```bash
mkfs.ext4 /dev/mydisk2
```
格式化根分区
```bash
mkfs.ext4 /dev/mydisk3
```
格式化home分区
```bash
mkfs.xfs /dev/mydisk4
```
设置swap分区
```bash
mkswap /dev/mydisk5
```
激活swap分区
```bash
swapon /dev/mydisk5
```
注：
 - 交换分区分区在此处激活，后续无需挂载交换分区
 - 如果主板不支持UEFI引导，请忽略boot和efi分区
### 挂载分区
挂载根分区
```bash
mount /dev/mydisk3 /mnt
```
在根分区下新建boot文件夹
```bash
mkdir /mnt/boot
```
挂载boot分区
```bash
mount /dev/mydisk2 /mnt/boot
```
在boot目录下新建efi文件夹
```bash
mkdir /mnt/boot/efi
```
挂载efi分区
```bash
mount /dev/mydisk1 /mnt/boot/efi
```
在根分区下新建home文件夹
```bash
mkdir /mnt/home
```
挂载home分区
```bash
mount /dev/mydisk4 /mnt/home
```
注：
 - 交换分区分区已提前激活，分区时无需挂载交换分区
 - 如果主板不支持UEFI引导，请忽略boot和efi分区
## 更换软件源
最新版archlinux的liveCD有坑，会自动切换你的软件源，所以，请先刷新一下软件源
```bash
pacman -Syy
```
如果更新很慢，可以按Ctrl+C直接终止操作
然后可以开始修改了
删除软件源文件
```bash
rm /etc/pacman.d/mirrorlist
```
重新新建
```bash
vim /etc/pacman.d/mirrorlist
```
输入以下内容
```
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```
## 安装基本系统
```bash
pacstrap -i /mnt base base-devel linux linux-firmware vim nano
```
注：
 - 安装vim和nano是因为需要修改配置文件，然而arch linux不带vim和nano， 需要手动安装。
## 配置fstab
自动配置fstab
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
检查fstab
```bash
cat /mnt/etc/fstab
```
正常情况（此处用archlinux实体机演示）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201010141256970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI1MTc4MA==,size_16,color_FFFFFF,t_70#pic_center)

注：
 - 无论主板是否支持UEFI都可以用这个方法安装
## 切换至新系统
```bash
arch-chroot /mnt /bin/bash
```
# 系统基本配置
## 语言设置
配置本地语言
```bash
vim /etc/locale.gen
```
反注释
```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```
应用配置
```bash
locale-gen
```
运行
```bash
echo LANG=en_US.UTF-8 >> /etc/locale.conf
```
注：
 - 不设置中文的原因是因为tty环境下可能会出现中文乱码，安装桌面时会安装中文字体并修改相关内容
## 时区设置
本地时区配置
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
设置硬件时钟
```bash
hwclock --systohc --utc
```
## 引导系统
### Grub引导Windows或其他Linux
安装此软件包(如果使用archlinux单系统或者是不想用arch linux引导其他系统则可以省略此步骤)
```bash
pacman -S os-prober
```
### 主板支持UEFI引导
下载grub安装时所必需的文件
```bash
pacman -S dosfstools grub efibootmgr
```
`安装grub
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --recheck
```
如果提示no error reported则说明grub安装成功
更新grub
```bash
grub-mkconfig -o /etc/grub/grub.cfg
```
### 主板不支持UEFI引导
下载grub安装时所必需的文件
```bash
pacman -S grub
```
安装grub
```bash
grub-install /dev/sda --recheck
```
如果提示no error reported则说明grub安装成功
更新grub
```bash
grub-mkconfig -o /boot/grub/grub.cfg 
```
更新grub
## 用户设置
设置主机名
```bash
vim /etc/hostname
```
输入主机名，只能输入字母（如果跳过此配置，主机名默认为archlinux）
设置root密码
```bash
passwd
```
添加用户
```bash
useradd -m -g users -s /bin/bash username
```
设置用户密码（可以和root相同）
```bash
passwd username
```
为用户添加sudo权限
```bash
vim /etc/sudoers
```
在
```
root ALL=(ALL) ALL
```
下面添加
```
username ALL=(ALL) ALL
```
如果想在输入密码时显示星号，可以追加
```
Defaults env_reset,pwfeedback
```
注：
 - usename为用户名
 -  输入密码时不显示密码是正常现象
 - 编辑/etc/sudoers时要用:wq!命令执行，加一个感叹号强制执行
## 安装网络驱动
### wifi
安装wifi驱动
```bash
pacman -S netctl iw wpa_supplicant dialog
```
博通网卡请安装这个驱动
```bash
pacman -S broadcom-wl
```
### 有线连接dhcp 
```bash
pacman -S dhcp dhcpcd
```
## 退出chroot并进入新系统
退出chroot
```bash
exit
```
卸载efi分区
```bash
umount /dev/mydisk1
```
卸载boot分区
```bash
umount /dev/mydisk2
```
卸载home分区
```bash
umount /dev/mydisk4
```
卸载根分区
```bash
umount /dev/mydisk3
```
重启电脑
```bash
reboot
```
注：
 - 运行reboot命令后，请拔掉U盘
# 配置系统
## 网络配置
### Wi-Fi
```bash
sudo wifi-menu
```
### dhcp
```bash
sudo systemctl enable --now dhcpcd
```
重新检查网络连接
```bash
ping https://mirrors.tuna.tsinghua.edu.cn
```
## 安装驱动
### 显卡驱动
```bash
lspci | grep VGA
```
按照自己的显卡型号安装相应驱动
|显卡|驱动名称|
|--|--|
|通用|xf86-video-vesa|
|Intel|xf86-video-intel|
|AMD|xf86-video-amdgpu|
|NVIDIA|nvidia nvidia-utils cuda nvidia-settings opencl-nvidia|
|开源MVIDIA(不推荐)|xf86-video-nouveau|
|ati|xf86-video-ati|
|vmware虚拟机|xf86-video-vmware
|
FBI Warning：
 - 千万不要安装nouveau，千万不要安装nouveau，千万不要安装nouveau！重要的事情说三遍。如果你不怕电脑莫名卡死，当我没说。（doge）ps：博主受过nouveau的折磨
### 触摸板驱动
笔记本专用，台式机可以忽略（如果有外置触摸板也可以安装这个驱动）
```bash
sudo pacman -S xf86-input-synaptics
```
## 安装桌面
安装中文字体
```bash
sudo pacman -S ttf-dejavu wqy-microhei
```
将语言改成中文
```bash
sudo vim /etc/locale.conf
```
将英语注释掉，添加以下内容
```
LANG=zh_CN.UTF-8
```
安装x窗口系统
```bash
sudo pacman -S xorg
```
安装桌面环境（以kde为例）
```bash
sudo pacman -S plasma
```
安装kde软件包
```bash
sudo pacman -S kde-applications
```
安装kde网络管理器
```bash
sudo pacman -S plasma-nm
```
启动sddm桌面
```bash
sudo systemctl enable sddm
```
启动网络管理
```bash
sudo systemctl enable NetworkManager
```
重启，Enjoy it！
```bash
sudo reboot
```
# 安装后配置
## 添加archlinuxcn源
编辑pacman.conf文件
```bash
sudo vim /etc/pacman.conf
```
在末尾追加以下内容
```
[archlinuxcn]
SigLevel = TrustAll
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
安装archlinuxcn密钥环
```bash
sudo pacman -S archlinuxcn-keyring
```
## 安装fcitx5中文输入法
新安装的arch linux系统不带中文输入法，直接安装
```bash
sudo pacman -S fcitx5-chinese-addons fcitx5-git fcitx5-gtk fcitx5-qt fcitx5-pinyin-zhwiki kcm-fcitx5
```
编辑配置文件
```bash
vim ~/.pam_environment
```
写入以下内容
```
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=@im=fcitx
```
设置开机默认启动fcitx5
编辑配置文件
```bash
vim ~/.xprofile
```
写入以下内容
```
fcitx5 &
```
重启电脑，则可以输入中文
## 安装常用软件
软件包
|软件名|包名|
|--|--|
|网易云音乐|netease-cloud-music|
|WPS|wps-office|
|WPS中文支持|wps-office-mui-zh|
|火狐|firefox|
|VS Code|visual-studio-code-bin|
|谷歌浏览器|google-chrome|
|OBS Studio|obs-studio|
|火焰截图|flameshot|
|AUR|yaourt yay|
AUR软件包
|软件名|包名|
|-|-|
|octopi|octopi|
|anaconda|anaconda|
|jetbrains工具包|jetbrains-toolbox|

提示：
 - 非原生Arch同样适用（如Manjaro Arco Linux）
## 安装blackarch渗透测试工具包（可选）
添加blackarch软件源
```bash
sudo vim /etc/pacman.conf
```
添加以下内容
```
[blackarch]
SigLevel = Never
Server = https://mirrors.tuna.tsinghua.edu.cn/blackarch/$repo/os/$arch
```
安装blackarch密钥环
```bash
sudo pacman -S blackarch-keyring
```
更改blackarch软件源
```bash
sudo vim /etc/pacman.conf
```
把刚才的blackarch软件源改为
```
[blackarch]
SigLevel = TrustAll
Server = https://mirrors.tuna.tsinghua.edu.cn/blackarch/$repo/os/$arch
```
安装渗透工具
```bash
sudo pacman -S blackarch
```
警告：
 - 非Arch原生不要安装blackarch,不然容易滚挂！
 - Arch原生一定要按照我的提示操作，不然会提示密钥环问题。
# 桌面美化
## 美化效果图
以Manjaro为例，arch同样适用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200509175252686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI1MTc4MA==,size_16,color_FFFFFF,t_70#pic_center)
## 界面美化
### 调整面板
删除自带面板，新建空面板，从左往右依次是：
> 应用程序菜单->全局菜单->面板间距->系统托盘->数字时钟
可根据实际情况添加其他部件
### 安装主题
直接下载全局主题，依次点击
>系统设置->全局主题->获取新的主题（如果无法下载，可以分别在Plasma样式、颜色、应用程序风格等自己选择主题进行下载，也可以下载SDDM桌面、欢迎界面、图标和鼠标指针）
>系统设置->应用程序风格->窗口装饰->标题栏按钮（自己调整窗口标题栏，可以把按键放到左边）
>系统设置->工作空间行为->桌面特效->摆动窗口
>系统设置->工作空间行为->桌面特效->破碎
>系统设置->工作空间行为->桌面特效->飘落
>系统设置->工作空间行为->桌面特效->外观->魔灯
>系统设置->工作空间行为->焦点->滑出
>系统设置->工作空间行为->窗口管理->桌面立方
>系统设置->窗口管理->任务切换器->把微风改为翻转切换
### 安装Latte-Dock
关闭背景大小
绝对大小48
鼠标悬停放大40%
删除模拟时钟
在dock上添加部件
左侧为应用程序面板
右侧为回收站
```bash
sudo pacman -S latte-dock
```
如果右键无反应（此问题常常发生在Manjaro系统上），运行
```bash
sudo pacman -Syyu
```
此时Latte-Dock可以正常使用
### 安装小部件
>右键点击桌面空白部分->点击添加部件->搜索 Simple System Monitor -> 点击安装

安装后点击配置，Background Color选择Cristal，如果没有交换分区，可以自行关闭
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200509175600576.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI1MTc4MA==,size_16,color_FFFFFF,t_70#pic_center)
### 安装panon(可选)
直接添加部件，下载panon，然后运行命令
```bash
sudo pacman -S qt5-websockets python-docopt python-numpy python-pillow python-pyaudio python-cffi python-websockets
```
### 平铺KDE（可选，比较接近i3wm）
2020年9月24日更新
#### 教程来源
[教程来自于以下链接](https://opensuse.bwsl.wang/opensuse/%E5%B9%B3%E9%93%BAKDE.html)，[点击查看](https://space.bilibili.com/268630727?from=search&seid=12844584618959748428)原作者哔哩哔哩主页
#### 详细教程
```bash
git clone https://github.com/lingtjien/Grid-Tiling-Kwin.git ~/Grid-Tiling-Kwin
```
```bash
kpackagetool5 --type KWin/Script -i ~/Grid-Tiling-Kwin
```
```bash
mkdir ~/.local/share/kservices5
```
```bash
cp ~/Grid-Tiling-Kwin/metadata.desktop ~/.local/share/kservices5/kwin-script-grid-tiling.desktop
```
#### 效果图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924193831303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI1MTc4MA==,size_16,color_FFFFFF,t_70#pic_center)

## 终端美化
### 安装zsh
安装zsh
```bash
sudo pacman -S zsh
```
配置zsh
```bash
chsh -s /bin/zsh
```
如果想换回bash，运行
```bash
chsh -s /bin/bash
```
### 安装oh-my-zsh
运行命令（需要在zsh下运行）
如果没安装git需要运行以下命令，否则跳过
```bash
sudo pacman -S git
```
不要带sudo，否则会导致用户界面无法达到预期效果。
终端运行

~~sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"~~ 
如果提示timed out可以复制以下脚本，另存为install.sh
```bash
#!/bin/sh
#
# This script should be run via curl:
#   sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# or wget:
#   sh -c "$(wget -qO- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
#
# As an alternative, you can first download the install script and run it afterwards:
#   wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
#   sh install.sh
#
# You can tweak the install behavior by setting variables when running the script. For
# example, to change the path to the Oh My Zsh repository:
#   ZSH=~/.zsh sh install.sh
#
# Respects the following environment variables:
#   ZSH     - path to the Oh My Zsh repository folder (default: $HOME/.oh-my-zsh)
#   REPO    - name of the GitHub repo to install from (default: ohmyzsh/ohmyzsh)
#   REMOTE  - full remote URL of the git repo to install (default: GitHub via HTTPS)
#   BRANCH  - branch to check out immediately after install (default: master)
#
# Other options:
#   CHSH       - 'no' means the installer will not change the default shell (default: yes)
#   RUNZSH     - 'no' means the installer will not run zsh after the install (default: yes)
#   KEEP_ZSHRC - 'yes' means the installer will not replace an existing .zshrc (default: no)
#
# You can also pass some arguments to the install script to set some these options:
#   --skip-chsh: has the same behavior as setting CHSH to 'no'
#   --unattended: sets both CHSH and RUNZSH to 'no'
#   --keep-zshrc: sets KEEP_ZSHRC to 'yes'
# For example:
#   sh install.sh --unattended
#
set -e

# Default settings
ZSH=${ZSH:-~/.oh-my-zsh}
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
BRANCH=${BRANCH:-master}

# Other options
CHSH=${CHSH:-yes}
RUNZSH=${RUNZSH:-yes}
KEEP_ZSHRC=${KEEP_ZSHRC:-no}


command_exists() {
	command -v "$@" >/dev/null 2>&1
}

error() {
	echo ${RED}"Error: $@"${RESET} >&2
}

setup_color() {
	# Only use colors if connected to a terminal
	if [ -t 1 ]; then
		RED=$(printf '\033[31m')
		GREEN=$(printf '\033[32m')
		YELLOW=$(printf '\033[33m')
		BLUE=$(printf '\033[34m')
		BOLD=$(printf '\033[1m')
		RESET=$(printf '\033[m')
	else
		RED=""
		GREEN=""
		YELLOW=""
		BLUE=""
		BOLD=""
		RESET=""
	fi
}

setup_ohmyzsh() {
	# Prevent the cloned repository from having insecure permissions. Failing to do
	# so causes compinit() calls to fail with "command not found: compdef" errors
	# for users with insecure umasks (e.g., "002", allowing group writability). Note
	# that this will be ignored under Cygwin by default, as Windows ACLs take
	# precedence over umasks except for filesystems mounted with option "noacl".
	umask g-w,o-w

	echo "${BLUE}Cloning Oh My Zsh...${RESET}"

	command_exists git || {
		error "git is not installed"
		exit 1
	}

	if [ "$OSTYPE" = cygwin ] && git --version | grep -q msysgit; then
		error "Windows/MSYS Git is not supported on Cygwin"
		error "Make sure the Cygwin git package is installed and is first on the \$PATH"
		exit 1
	fi

	git clone -c core.eol=lf -c core.autocrlf=false \
		-c fsck.zeroPaddedFilemode=ignore \
		-c fetch.fsck.zeroPaddedFilemode=ignore \
		-c receive.fsck.zeroPaddedFilemode=ignore \
		--depth=1 --branch "$BRANCH" "$REMOTE" "$ZSH" || {
		error "git clone of oh-my-zsh repo failed"
		exit 1
	}

	echo
}

setup_zshrc() {
	# Keep most recent old .zshrc at .zshrc.pre-oh-my-zsh, and older ones
	# with datestamp of installation that moved them aside, so we never actually
	# destroy a user's original zshrc
	echo "${BLUE}Looking for an existing zsh config...${RESET}"

	# Must use this exact name so uninstall.sh can find it
	OLD_ZSHRC=~/.zshrc.pre-oh-my-zsh
	if [ -f ~/.zshrc ] || [ -h ~/.zshrc ]; then
		# Skip this if the user doesn't want to replace an existing .zshrc
		if [ $KEEP_ZSHRC = yes ]; then
			echo "${YELLOW}Found ~/.zshrc.${RESET} ${GREEN}Keeping...${RESET}"
			return
		fi
		if [ -e "$OLD_ZSHRC" ]; then
			OLD_OLD_ZSHRC="${OLD_ZSHRC}-$(date +%Y-%m-%d_%H-%M-%S)"
			if [ -e "$OLD_OLD_ZSHRC" ]; then
				error "$OLD_OLD_ZSHRC exists. Can't back up ${OLD_ZSHRC}"
				error "re-run the installer again in a couple of seconds"
				exit 1
			fi
			mv "$OLD_ZSHRC" "${OLD_OLD_ZSHRC}"

			echo "${YELLOW}Found old ~/.zshrc.pre-oh-my-zsh." \
				"${GREEN}Backing up to ${OLD_OLD_ZSHRC}${RESET}"
		fi
		echo "${YELLOW}Found ~/.zshrc.${RESET} ${GREEN}Backing up to ${OLD_ZSHRC}${RESET}"
		mv ~/.zshrc "$OLD_ZSHRC"
	fi

	echo "${GREEN}Using the Oh My Zsh template file and adding it to ~/.zshrc.${RESET}"

	sed "/^export ZSH=/ c\\
export ZSH=\"$ZSH\"
" "$ZSH/templates/zshrc.zsh-template" > ~/.zshrc-omztemp
	mv -f ~/.zshrc-omztemp ~/.zshrc

	echo
}

setup_shell() {
	# Skip setup if the user wants or stdin is closed (not running interactively).
	if [ $CHSH = no ]; then
		return
	fi

	# If this user's login shell is already "zsh", do not attempt to switch.
	if [ "$(basename "$SHELL")" = "zsh" ]; then
		return
	fi

	# If this platform doesn't provide a "chsh" command, bail out.
	if ! command_exists chsh; then
		cat <<-EOF
			I can't change your shell automatically because this system does not have chsh.
			${BLUE}Please manually change your default shell to zsh${RESET}
		EOF
		return
	fi

	echo "${BLUE}Time to change your default shell to zsh:${RESET}"

	# Prompt for user choice on changing the default login shell
	printf "${YELLOW}Do you want to change your default shell to zsh? [Y/n]${RESET} "
	read opt
	case $opt in
		y*|Y*|"") echo "Changing the shell..." ;;
		n*|N*) echo "Shell change skipped."; return ;;
		*) echo "Invalid choice. Shell change skipped."; return ;;
	esac

	# Check if we're running on Termux
	case "$PREFIX" in
		*com.termux*) termux=true; zsh=zsh ;;
		*) termux=false ;;
	esac

	if [ "$termux" != true ]; then
		# Test for the right location of the "shells" file
		if [ -f /etc/shells ]; then
			shells_file=/etc/shells
		elif [ -f /usr/share/defaults/etc/shells ]; then # Solus OS
			shells_file=/usr/share/defaults/etc/shells
		else
			error "could not find /etc/shells file. Change your default shell manually."
			return
		fi

		# Get the path to the right zsh binary
		# 1. Use the most preceding one based on $PATH, then check that it's in the shells file
		# 2. If that fails, get a zsh path from the shells file, then check it actually exists
		if ! zsh=$(which zsh) || ! grep -qx "$zsh" "$shells_file"; then
			if ! zsh=$(grep '^/.*/zsh$' "$shells_file" | tail -1) || [ ! -f "$zsh" ]; then
				error "no zsh binary found or not present in '$shells_file'"
				error "change your default shell manually."
				return
			fi
		fi
	fi

	# We're going to change the default shell, so back up the current one
	if [ -n "$SHELL" ]; then
		echo $SHELL > ~/.shell.pre-oh-my-zsh
	else
		grep "^$USER:" /etc/passwd | awk -F: '{print $7}' > ~/.shell.pre-oh-my-zsh
	fi

	# Actually change the default shell to zsh
	if ! chsh -s "$zsh"; then
		error "chsh command unsuccessful. Change your default shell manually."
	else
		export SHELL="$zsh"
		echo "${GREEN}Shell successfully changed to '$zsh'.${RESET}"
	fi

	echo
}

main() {
	# Run as unattended if stdin is closed
	if [ ! -t 0 ]; then
		RUNZSH=no
		CHSH=no
	fi

	# Parse arguments
	while [ $# -gt 0 ]; do
		case $1 in
			--unattended) RUNZSH=no; CHSH=no ;;
			--skip-chsh) CHSH=no ;;
			--keep-zshrc) KEEP_ZSHRC=yes ;;
		esac
		shift
	done

	setup_color

	if ! command_exists zsh; then
		echo "${YELLOW}Zsh is not installed.${RESET} Please install zsh first."
		exit 1
	fi

	if [ -d "$ZSH" ]; then
		cat <<-EOF
			${YELLOW}You already have Oh My Zsh installed.${RESET}
			You'll need to remove '$ZSH' if you want to reinstall.
		EOF
		exit 1
	fi

	setup_ohmyzsh
	setup_zshrc
	setup_shell

	printf "$GREEN"
	cat <<-'EOF'
		         __                                     __
		  ____  / /_     ____ ___  __  __   ____  _____/ /_
		 / __ \/ __ \   / __ `__ \/ / / /  /_  / / ___/ __ \
		/ /_/ / / / /  / / / / / / /_/ /    / /_(__  ) / / /
		\____/_/ /_/  /_/ /_/ /_/\__, /    /___/____/_/ /_/
		                        /____/                       ....is now installed!
		Please look over the ~/.zshrc file to select plugins, themes, and options.
		p.s. Follow us on https://twitter.com/ohmyzsh
		p.p.s. Get stickers, shirts, and coffee mugs at https://shop.planetargon.com/collections/oh-my-zsh
	EOF
	printf "$RESET"

	if [ $RUNZSH = no ]; then
		echo "${YELLOW}Run zsh to try it out.${RESET}"
		exit
	fi

	exec zsh -l
}

main "$@"
```
然后执行
```bash
sh install.sh
```
注意：不要以sudo用户执行，执行前需要安装git，确保安装的有zsh终端
```bash
sudo pacman -S git
```
### 安装PowerLevel10k主题
依次运行以下命令
安装字体
```bash
sudo pacman -S nerd-fonts-complete
```
安装powerlevel10k主题
```bash
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ~/powerlevel10k
```
```bash
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
配置powerlevel10k
```bash
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>! ~/.zshrc
```
在~/.zshrc写入
```
ZSH_THEME="powerlevel10k/powerlevel10k" 
```
使配置立即生效
```bash
source ~/.zshrc
```
安装后重新打开终端，并更改终端字体为hack
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200509203317556.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI1MTc4MA==,size_16,color_FFFFFF,t_70#pic_center)
如需重新配置powerlevel10k主题，可随时运行
```bash
p10k configure
```
### 安装自动补全插件
```bash
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```
运行以下命令更改配置
```bash
vim ~/.zshrc
```
添加
```bash
plugins=(zsh-autosuggestions git)
```
使配置立即生效
```bash
source ~/.zshrc
```
### 安装语法高亮插件
运行以下命令
```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```
配置插件
```bash
echo "source $ZSH_CUSTOM/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
```
使配置立即生效
```bash
source ~/.zshrc
```
### vimplus(可选)
#### 本插件非原创，感谢原作者chxuan
如果想查看原作者文章，请[点击此处](https://www.cnblogs.com/highway-9/p/5984285.html)
从gitee克隆
```bash
# 不使用github是因为国外github连接不稳定
git clone https://gitee.com/chxuan/vimplus.git ~/.vimplus
```
安装vimplus（可能需要五分钟左右）
```bash
cd ~/.vimplus
sh ./install.sh # 不要加sudo
```
安装到root用户
```bash
cd ~/.vimplus
sh ./install_to_user.sh username root
```
更新vimplus
```bash
cd ~/.vimplus
sh ./update.sh
```
 - [ ] # 视频教程部分
 - [ ] ## ArchLinux安装
 - [ ] ## 安装后配置
 - [ ] ## 桌面美化
 - [ ] ## 使用体验
教程正在火速更新，部分过程将用虚拟机演示
