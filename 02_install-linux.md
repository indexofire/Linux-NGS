# 安装Linux

这里以小米pro 15.6为例介绍安装archlinux的过程。这不是广告啊，小米新出的游戏本并不合适，小米 pro 15.6性价比比较高，外形也有其独特的风格。对于新款小米13和15.6的选择，散热+噪音比便携性更重要，因此选择了前者。

## 硬件与软件

硬件配置：

- CPU  i5 8250
- 内存  16G
- 硬盘  256G SSD
- 屏幕  15.6 1920x1080 IPS 72% 色域

日常软件配置：

- 操作系统 ArchLinux
- 窗口管理 i3-gaps
- 文件管理 ranger
- 音乐播放 mpd
- 视频播放 mpv
- 文本编辑 vim
- 图片查看 w3m
- 文档处理 wps
- 屏幕录制视频 ffmpeg

## 安装ArchLinux

小米的BIOS界面功能真是简单，看了一下拿到手的是0502版BIOS。在windows10里跑了ssd测试。BIOS 必须设置 supervisor password 才能将修改 secure boot 为 disabled。开机按F2进入BIOS，按F12选择引导设备。

开机按F12，选择U盘启动，进入Archlinux界面。插上一个大容量U盘，用来将原硬盘内容进行备份，以备将来需要恢复时使用。查看一下当前磁盘情况：

```bash
root@archiso~ # lsblk -f
NAME		FSTYPE		LABEL			UUID					MOUNTPOINT
loop0		squashfs											/run/archiso/sfs/airrootfs
sda
-sda1		vfat		BANQ			C873-E5BF
sdb			iso9660		ARCH_201803		2018-03-01-15-13-16-00
-sdb1		iso9660		ARCH_201803		2018-03-01-15-13-16-00	/run/archiso/bootmnt
-sdb2		vfat		ARCHISO_EFI		8688-0060
nvme0n1
-nvme0n1p1	vfat		SYSTEM			E479-B82E
-nvme0n1p2
-nvme0n1p3	ntfs		Windows			3ACC7A8CCC7A4265
-nvme0n1p4	ntfs		Recovery image	C2DE7AF4DE7ADFD9
```

当前版本的linux内核已经包含了驱动，不会出现nvme ssd硬盘找不到的情况。小米pro 256G SSD硬盘上有4个分区，分别时系统引导EFI分区，保留分区，win10分区和系统恢复分区。我想保留win10的recovery区，找一个U盘来备份分区到文件。

```bash
root@archiso~ # mkdir -p /backup
root@archiso~ # mount /dev/sda1 /backup
root@archiso~ # dd if=/dev/nvme0n4 of=/backup/mmipro_recovery.bin bs=64k conv=notrunc,noerror,sync status=progress
15680+0 records in
15680+0 records out
10276604480 bytes (1.0 GB, 908 MiB) copied, 0.697636 s, 1.5 GB/s
```

连接wifi网络，可以直接用wifi-menu，也可以用wpa_supplicant命令连接。连接成功后运行dhcpcd，就可以访问互联网络了。

```bash
# 用wifi-menu
root@archiso~ # wifi-menu
# 用wpa_supplicant
root@archiso~ # wpa_supplicant -i wlp2s0 -c <(wpa_passphrase "YourSSID" "YourKey") -B
# 连接网络
root@archiso~ # dhcpcd
root@archiso~ # ping -c 3 www.archlinux.org
```

考虑了很久要不要加密安装，由于是和孩子一起使用的，有加密狗的话可以限制小孩。但是考虑到可能存在的一些意外情况，安装就采用普通模式吧。对小孩的使用限制就从其他方面着手了。

运行 blkdiscard 清除硬盘，SSD上所有数据都会被清楚，请确认准备完成再执行。分区很简单，用fdisk或者parted都可以，喜欢图形化界面的可以用cgdisk。

```bash
# 擦除SSD数据
root@archiso~ # blkdiscard /dev/nvme0n1
# fdisk分区
root@archiso~ # fdisk /dev/nvme0n1

# 创建gpt分区表
Command (m for help): g
# 创建分区
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-500118158, default 2048): 2048
Last sector, +sector or +size{K,M,G,T,P} (2048-500118158, default 500118158): +512M

Command (m for help): n
Partition number (2-128, default 2): 2
First sector (...)

# 写入分区表
Command (m for help): w

# mkfs.fat -F32 /dev/sdb1
# mkfs.ext4 /dev/sdb2

# parted 分区
root@archiso~ # parted /dev/nvme0n1
```

挂载`/`和`/boot`分区到/mnt

```bash
# 添加挂载boot的路径
root@archiso~ # mkdir /mnt/boot
root@archiso~ # mount /dev/nvme0n1p2 /mnt
root@archiso~ # mount /dev/nvme0n1p1 /mnt/boot
# 修改服务器镜像，并安装基础系统
root@archiso~ # vim /etc/pacman.d/mirrorlist
root@archiso  # wifi-menu
# dhcpcd

root@archiso~ # pacstrap /mnt base base-devel
# 生成fstab
root@archiso~ # genfstab -U /mnt >> /mnt/etc/fstab

# chroot到新安装的系统中进行操作
root@archiso~ # arch-chroot /mnt /bin/bash
[root@archiso /] # ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
[root@archiso /] # hwclock --systohc --utc

[root@archiso /] # nano /etc/locale.gen
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8

[root@archiso /] # locale-gen
[root@archiso /] # echo LANG=en_US.UTF-8 > /etc/locale.conf

[root@archiso /] # echo myhostname > /etc/hostname
[root@archiso /] # nano /etc/hosts
127.0.0.1	localhost.localdomain	localhost
::1		localhost.localdomain	localhost
127.0.0.1	myhostname.localdomain	myhostname

[root@archiso /] # mkinitcpio -p linux
```

```bash
# 安装新系统所必须的无线网络相关软件
[root@archiso /]# pacman -S vim wpa_supplicant networkmanager
```

创建管理员密码，新建用户mark，并建立主用户目录和密码。将用户mark设置为wheel组，获得管理员权限。

```bash
[root@archiso /]# passwd root
[root@archiso /]# useradd -G wheel -m mark
[root@archiso /]# passwd mark
[root@archiso /]# visudo

%wheel ALL=(ALL) ALL
```

以前版本的grub不支持nvme，不确定新版本或者grub-git是否可以完美支持，这里我们就不用grub引导了，直接用systemd-boot来引导系统。

```bash
# 安装efi
[root@archiso /]# bootctl install
[root@archiso /]# vim /boot/efi/entities/arch.conf
[root@archiso /]# vim /boot/efi/load.conf
```

分区时我们没有建立swap分区，因为在SSD中，更喜欢用文件方式使用swap，更为灵活。

```
# mkdir /swap
# fallocate -l 2G /swap/swapfile
# chmod 600 /swap/swapfile
# mkswap /swap/swapfile
# swapon /swap/swapfile
# echo "/swap/swapfile none swap defaults 0 0" >> /etc/fstab
```

添加用户

```
# useradd mark -g wheel -m
# passwd mark
New password:
Retype new password:
```
