安装Archlinux
============
一、下载并设置导入u盘
------------------
    先从archlinux官网http://mirrors.163.com/archlinux/iso/2020.06.01/
    下载archlinux-2020.06.01-x86_64.iso文件
    因为iso文件不能直接导入u盘中，所以需要格式化，刻入u盘
    在rufus官网https://rufus.ie/downloads/
    下载rufus-3.10.exe
    
二、利用u盘启动（F2，选中option1为UEFI NET...）
------------------------------------------
### 1、验证启动模式
```
# ls /sys/firmware/efi/efivars
```

### 2、连接网络（先使用有线网络）
#### a.查看网络类型
```
# ip link
```
#### b.连接有线网络
```
# dhcpcd
```
#### c.查看是否连接成功
```
# ping www.baidu.com
```

### 3、更新系统时间
```
# timedatectl set-ntp true
```

### 4、建立硬盘分区
```
# cfdisk
```
sda1    EFI     300MB
sda2    /boot   700MB
sda3    swap    2G
sda4    /       其余的容量
需要写入(write)，再退出

### 5、格式化分区
```
# mkfs.ext4 /dev/sda2
# mkfs.ext4 /dev/sda4
# mkfs.vfat /dev/sda1
# mkswap /dev/sda3
# swapon /dev/sda3
```

### 6、挂载分区
#### a.先挂载根（/）分区
```
# mount /dev/sda4 /mnt
```
#### b.创建/home和/efi文件夹在/mnt中
```
# mkdir /mnt/home
# mkdir /mnt/efi
```
#### c.挂载剩余分区（交换分区除外）
```
# mount /dev/sda1 /mnt/efi
# mount /dev/sda2 /mnt/home
```

### 7、安装所需要的软件包
#### a.编辑/etc/pacman.d/mirrorlist
```
# nano /etc/pacman.d/mirrorlist
```
#### b.添加aliyun镜像在最上面
```
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
```
#### c.安装 base 软件包和 Linux 内核以及常规硬件的固件
```
# pacstrap /mnt base linux linux-firmware
```

### 8、配置系统（新系统）
#### a.将配置好的分区从u盘转移到硬盘
```
# genfstab -U /mnt >> /mnt/etc/fstab
```
#### b.使用Change root 到新安装的系统
```
# arch-chroot /mnt
```
#### c.设置时区
```
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# hwclock --systohc
```
#### d.下载nano编辑
```
# pacman -S nano
```
#### e.下载dhcpcd
```
# pacman -S dhcpcd
```
#### f.设置本地化
```
# nano /etc/locale.gen
# locale-gen
```

### 9、配置网络
#### a.创建 hostname 文件
```
# nano /etc/hostname
--------------------
    gofast
```
#### b.添加对应的信息到 hosts
```
127.0.0.1	localhost
::1		    localhost
127.0.1.1	gofast.localdomain	gofast
```
### 10、设置 Root 密码
```
# passwd
--------
    1234
```

三、安装引导程序（Grub）
-------------------
```
# pacman -S grub efibootmgr
# grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=grub
# grub-mkconfig -o /boot/grub/grub.cfg
```
四、重启
------
```
# exit
```
*手动卸载被挂载的分区（脱离u盘）*
```
# umount -R /mnt
```
```
# reboot
```

五、重启进入archlinux后，配置无线网络
--------------------------------
### a.安装wpa_supplicant、iw包
```
# pacman -S wpa_supplicant
# pacman -S iw
```
### b.配置wifi
```
# ip link
# ip link set wlp2s0 up (down是关闭)
# wpa_supplicant -i Xiaomi_hzg -c < abcd123456789
# wpa_supplicant (-B) -i wlp2s0 -c /etc/wpa_supplicant/wpa_supplicant.conf
# ping www.baidu.com
```