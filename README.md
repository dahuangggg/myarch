# myarch

[中文](#中文) | [English](#english)

## 中文

个人 Arch Linux 安装与 KDE 桌面配置笔记（Btrfs + UEFI/GRUB）。

### 状态

早期项目，不再维护。内容仅供参考，软件包名和步骤可能已过时，请以 [Arch Wiki](https://wiki.archlinux.org/) 为准。

### 功能

- 从 Live 环境安装基础系统：联网、换源、分区、格式化、`pacstrap`、`genfstab`、`arch-chroot`
- Btrfs 子卷布局（`@`、`@home`，zstd 压缩）+ EFI + Swap
- 主机名、时区、Locale、root 密码、微码、GRUB 引导
- 新建 `wheel` 用户、启用 `multilib` 与 `archlinuxcn` 仓库
- KDE Plasma + SDDM 桌面、中文字体、fcitx5 输入法、蓝牙、代理（v2rayA）
- AMD + NVIDIA 显卡驱动（optimus-manager）、zsh 插件、休眠（`resume`）、常用软件

### 技术栈

Arch Linux · pacman / yay (AUR) · Btrfs · GRUB (UEFI) · systemd · NetworkManager · KDE Plasma · SDDM · zsh

### 使用方法

本仓库只有文档，无需构建或安装依赖。

1. 用 Arch Linux 安装镜像启动目标机器。
2. 按下方[安装步骤](#安装步骤)依次执行命令。
3. 按实际硬件修改设备名（如 `/dev/nvme0n1`）、主机名、用户名等占位内容。

> 注意：分区和格式化命令会清除磁盘数据，执行前请确认目标磁盘。

### 目录结构

```text
myarch/
└── README.md   # 说明文档 + 完整安装步骤
```

## English

Personal notes for installing Arch Linux and setting up a KDE desktop (Btrfs + UEFI/GRUB). The step-by-step guide below is written in Chinese.

### Status

Early project, no longer maintained. For reference only; package names and steps may be outdated, so check the [Arch Wiki](https://wiki.archlinux.org/).

### Features

- Base install from the live ISO: network, mirrors, partitioning, formatting, `pacstrap`, `genfstab`, `arch-chroot`
- Btrfs subvolume layout (`@`, `@home`, zstd compression) + EFI + swap
- Hostname, timezone, locale, root password, microcode, GRUB bootloader
- `wheel` user, `multilib` and `archlinuxcn` repositories
- KDE Plasma + SDDM, Chinese fonts, fcitx5 input method, Bluetooth, proxy (v2rayA)
- AMD + NVIDIA graphics drivers (optimus-manager), zsh plugins, hibernation (`resume`), common apps

### Tech stack

Arch Linux · pacman / yay (AUR) · Btrfs · GRUB (UEFI) · systemd · NetworkManager · KDE Plasma · SDDM · zsh

### Getting started

This repo is documentation only; there is nothing to build or install.

1. Boot the target machine from the Arch Linux installation ISO.
2. Run the commands in the [installation steps](#安装步骤) below in order.
3. Replace placeholders such as device names (`/dev/nvme0n1`), hostname and username to match your machine.

> Warning: the partitioning and formatting commands erase disk data. Double-check the target disk first.

### Directory structure

```text
myarch/
└── README.md   # docs + full installation steps
```

---

## 安装步骤

### 禁用 reflector 服务
```bash
systemctl stop reflector.service
```

### 连接网络
```bash
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect wifi-name
exit
```

### 更新系统时钟
```bash
timedatectl set-ntp true # 将系统时间与网络时间进行同步
timedatectl status # 检查服务状态
```

### 更换国内软件仓库镜像源
```bash
vim /etc/pacman.d/mirrorlist
```

```text
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch # 中国科学技术大学开源镜像站
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch # 清华大学开源软件镜像站
```

### 分区
```bash
lsblk
cfdisk /dev/nvme0n1
...
```

### 格式化
```bash
mkfs.fat -F32 /dev/nvme0n1p1 # 格式化 EFI 分区
mkswap /dev/nvme0n1p2 # 格式化 Swap 分区
mkfs.btrfs -L myArch /dev/nvme0n1p3 # 格式化 Btrfs 分区
mount -t btrfs -o compress=zstd /dev/sdxn /mnt
btrfs subvolume create /mnt/@ # 创建 / 目录子卷
btrfs subvolume create /mnt/@home # 创建 /home 目录子卷
umount /mnt
```

### 挂载
```bash
mount -t btrfs -o subvol=/@,compress=zstd /dev/nvmexn1p3 /mnt # 挂载 / 目录
mkdir /mnt/home # 创建 /home 目录
mount -t btrfs -o subvol=/@home,compress=zstd /dev/nvmexn1p3 /mnt/home # 挂载 /home 目录
mkdir -p /mnt/boot # 创建 /boot 目录
mount /dev/nvmexn1p1 /mnt/boot # 挂载 /boot 目录
swapon /dev/nvmexn1p2 # 挂载交换分区
```

### 安装系统
```bash
pacstrap /mnt base base-devel linux linux-firmware btrfs-progs
pacstrap /mnt networkmanager vim sudo zsh zsh-completions #必要软件
```

### 生成 fstab 文件
```bash
genfstab -U /mnt > /mnt/etc/fstab
```

### change root
```bash
arch-chroot /mnt
```
### 设置主机名与时区
```bash
vim /etc/hostname
```

```text
myarch
```

```bash
vim /etc/hosts
```

```text
127.0.0.1  localhost
::1        localhost
127.0.1.1  myarch.localdomain myarch
```

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 硬件时间设置
```bash
hwclock --systohc
```

### 设置 Locale
```bash
vim /etc/locale.gen
```

```text
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

```bash
locale-gen
```

### 设置root密码
```bash
passwd root
```

### 安装微码
```bash
pacman -S amd-ucode # AMD
```

### 安装引导程序
```bash
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
vim /etc/default/grub
```

```text
 “loglevel=5 nowatchdog”
```

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### 完成安装
```bash
exit # 退回安装环境
umount -R /mnt # 卸载新分区
reboot # 重启
```

```bash
systemctl enable --now NetworkManager # 设置开机自启并立即启动 NetworkManager 服务
nmcli dev wifi list # 显示附近的 Wi-Fi 网络
nmcli dev wifi connect "Wi-Fi名（SSID）" password "网络密码" # 连接指定的无线网络
pacman -S neofetch
neofetch # 完成
```

### 配置 root 账户的默认编辑器
```bash
vim ~/.bash_profile
```

```text
export EDITOR='vim'
```

### 添加用户
```bash
useradd -m -G wheel -s /bin/bash myusername
passwd myusername
EDITOR=vim visudo # 这里需要显式的指定编辑器，因为上面的环境变量还未生效
 #%wheel ALL=(ALL:ALL) ALL
```

### 开启 32 位支持库与 Arch Linux 中文社区仓库
```bash
vim /etc/pacman.conf
```

```text
[multilib]
...

[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch # 中国科学技术大学开源镜像站
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch # 清华大学开源软件镜像站
```

```bash
pacman -Syyu
```

### 安装KDE桌面环境
```bash
pacman -S plasma-meta konsole dolphin
```

### 配置并启动 greeter sddm
```bash
systemctl enable sddm
systemctl start sddm
```

### 必备1
```bash
sudo pacman -S ntfs-3g # 使系统可以识别 NTFS 格式的硬盘
sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei # 几个开源中文字体
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra # 安装谷歌开源字体及表情
sudo pacman -S chromium # 安装 chromium 浏览器
sudo pacman -S ark # 压缩软件。在 dolphin 中可用右键解压压缩包
sudo pacman -S packagekit-qt5 packagekit appstream-qt appstream # 确保 Discover（软件中心）可用，需重启
sudo pacman -S gwenview # 图片查看器
sudo pacman -S fcitx5-im fcitx5-chinese-addons # 输入法
sudo pacman -S archlinuxcn-keyring 
sudo pacman -S yay
```

### 开启蓝牙
```bash
sudo systemctl enable --now bluetooth
```

### 代理
```bash
sudo pacman -S v2ray v2raya
sudo systemctl enable --now v2raya
```

### 配置非 root 账户的默认编辑器
```bash
vim ~/.bashrc
```

```text
  export EDITOR='vim'
```

### 显卡驱动
```bash
sudo pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon
sudo pacman -S nvidia-open nvidia-settings lib32-nvidia-utils
yay -S optimus-manager optimus-manager-qt
reboot
sudo systemctl enable optimus-manager.service
```

### 系统设置
```bash
System Settings->Startup and Shutdown->Desktop Session->Start with an empty session
System Settings->Workspace Behavior->General Behavior->Selects them
```

### zsh
```bash
sudo pacman -S zsh zsh-autosuggestions zsh-syntax-highlighting zsh-completions
sudo pacman -S autojump
chsh -l # 查看安装了哪些 Shell
chsh -s /usr/bin/zsh # 修改当前账户的默认 Shell
vim ~/.zshrc
```

```text
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/autojump/autojump.zsh
```

### 休眠
```bash
lsblk -o name,mountpoint,size,uuid
sudo vim /etc/default/grub
```

```text
  resume=UUID=......
```

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo vim /etc/mkinitcpio.conf
```

```text
  udev resume autodetect
```
```bash
sudo mkinitcpio -P
```

### 软件
```bash
sudo pacman -S okular
sudo pacman -S flameshot
sudo pacman -S telegram-desktop
yay -S linuxqq
yay -S wechat-uos
sudo pacman -S discord
yay -S wemeet-bin
sudo pacman -S libreoffice-still libreoffice-still-zh-cn
sudo pacman -S typora pandoc
# uxplay
yay -S uxplay-git
sudo systemctl start avahi-daemon.service
gst-plugins-good

sudo pacman -S kdeconnect sshfs
sudo pacman -S mpv
yay -S yesplaymusic
sudo pacman -S shotcut
sudo pacman -S gimp
sudo pacman -S blender
yay -S figma-linux
```
