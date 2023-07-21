# myarch

## 禁用 reflector 服务
systemctl stop reflector.service

## 连接网络
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect wifi-name
exit

## 更新系统时钟
```bash```
timedatectl set-ntp true # 将系统时间与网络时间进行同步
timedatectl status # 检查服务状态

## 更换国内软件仓库镜像源
vim /etc/pacman.d/mirrorlist
  Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch # 中国科学技术大学开源镜像站
  Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch # 清华大学开源软件镜像站

## 分区
lsblk
cfdisk /dev/nvme0n1
...

## 格式化
mkfs.fat -F32 /dev/nvme0n1p1 # 格式化 EFI 分区
mkswap /dev/nvme0n1p2 # 格式化 Swap 分区
mkfs.btrfs -L myArch /dev/nvme0n1p3 # 格式化 Btrfs 分区
btrfs subvolume create /mnt/@ # 创建 / 目录子卷
btrfs subvolume create /mnt/@home # 创建 /home 目录子卷
umount /mnt

## 挂载
mount -t btrfs -o subvol=/@,compress=zstd /dev/nvmexn1p3 /mnt # 挂载 / 目录
mkdir /mnt/home # 创建 /home 目录
mount -t btrfs -o subvol=/@home,compress=zstd /dev/nvmexn1p3 /mnt/home # 挂载 /home 目录
mkdir -p /mnt/boot # 创建 /boot 目录
mount /dev/nvmexn1p1 /mnt/boot # 挂载 /boot 目录
swapon /dev/nvmexn1p2 # 挂载交换分区

## 安装系统
pacstrap /mnt base base-devel linux linux-firmware btrfs-progs
pacstrap /mnt networkmanager vim sudo zsh zsh-completions #必要软件

## 生成 fstab 文件
genfstab -U /mnt > /mnt/etc/fstab

## change root
arch-chroot /mnt

## 设置主机名与时区
vim /etc/hostname
  myarch

vim /etc/hosts
  127.0.0.1  localhost
  ::1        localhost
  127.0.1.1  myarch.localdomain myarch

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

## 硬件时间设置
hwclock --systohc

## 设置 Locale
vim /etc/locale.gen
  en_US.UTF-8 UTF-8
  zh_CN.UTF-8 UTF-8
locale-gen

## 设置root密码
passwd root

## 安装微码
pacman -S amd-ucode # AMD

## 安装引导程序
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
vim /etc/default/grub
 “loglevel=5 nowatchdog”
grub-mkconfig -o /boot/grub/grub.cfg

 ## 完成安装
exit # 退回安装环境
umount -R /mnt # 卸载新分区
reboot # 重启

systemctl enable --now NetworkManager # 设置开机自启并立即启动 NetworkManager 服务
nmcli dev wifi list # 显示附近的 Wi-Fi 网络
nmcli dev wifi connect "Wi-Fi名（SSID）" password "网络密码" # 连接指定的无线网络
pacman -S neofetch
neofetch # 完成

## 配置 root 账户的默认编辑器
vim ~/.bash_profile
  export EDITOR='vim'

## 添加用户
useradd -m -G wheel -s /bin/bash myusername
passwd myusername

## 开启 32 位支持库与 Arch Linux 中文社区仓库
vim /etc/pacman.conf
  [multilib]
  ...

  [archlinuxcn]
  Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch # 中国科学技术大学开源镜像站
  Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch # 清华大学开源软件镜像站

pacman -Syyu

## 安装KDE桌面环境
pacman -S plasma-meta konsole dolphin

## 配置并启动 greeter sddm
systemctl enable sddm
systemctl start sddm

## 开启蓝牙
sudo systemctl enable --now bluetooth

## 必备1
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

## 代理
sudo pacman -S v2ray v2raya
sudo systemctl enable --now v2raya

## 配置非 root 账户的默认编辑器
vim ~/.bashrc
  export EDITOR='vim'

## 显卡驱动
sudo pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon
sudo pacman -S nvidia-open nvidia-settings lib32-nvidia-utils
yay -S optimus-manager optimus-manager-qt
reboot
sudo systemctl enable optimus-manager.service

## 系统设置
System Settings->Startup and Shutdown->Desktop Session->Start with an empty session
System Settings->Workspace Behavior->General Behavior->Selects them

## zsh
sudo pacman -S zsh zsh-autosuggestions zsh-syntax-highlighting zsh-completions
sudo pacman -S autojump
chsh -l # 查看安装了哪些 Shell
chsh -s /usr/bin/zsh # 修改当前账户的默认 Shell
vim ~/.zshrc
  source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
  source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
  source /usr/share/autojump/autojump.zsh

## 休眠
lsblk -o name,mountpoint,size,uuid
sudo vim /etc/default/grub
  resume=UUID=......
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo vim /etc/mkinitcpio.conf
  udev resume autodetect
sudo mkinitcpio -P
