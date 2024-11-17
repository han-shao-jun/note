@[toc]
## 基本原理

debootstrap是debian/ubuntu下的一个工具，用来构建一套基本的系统(根文件系统)。生成的目录符合Linux文件系统标准(FHS)，即包含了/boot、/etc、/bin、/usr等等目录，
但它官方比发行版本体积小很多，当然功能也没那么强大，因此只能说是“基本的系统，[官方介绍](https://wiki.debian.org/zh_CN/Debootstrap))，再通过apt工具安装需要的软件即可，
不选择ubuntu的原因是ubuntu太占磁盘(eMMC\SD卡)空间了，Debian相对要小一些。

## 安装依赖

* 安装软件包

```shell
sudo apt install debootstrap qemu-user-static binfmt-support
```

* 获取Debian11-keyring，否者报错

> Cannot check Release signature; keyring file not available /usr/share/keyrings/debian-archive-keyring.gpg

```shell
wget https://ftp-master.debian.org/keys/release-11.asc -qO- | gpg --import --no-default-keyring --keyring ./debian-release-11.gpg
```

## 拉取根文件系统

* 使用debootstrap直接获取基本根文件系统

```shell

# 基本令格式
sudo debootstrap --arch [平台] [发行版本代号] [目录] [源]

sudo debootstrap --arch=armhf --foreign --keyring=./debian-release-11.gpg bullseye ./bullseye_rootfs  https://mirrors.bfsu.edu.cn/debian/ #arm32
sudo debootstrap --arch=arm64 --foreign --keyring=./debian-release-11.gpg bullseye ./bullseye_rootfs  https://mirrors.bfsu.edu.cn/debian/ #arm64

# qemu-arm-static是其中的关键，能在 x86_64 主机系统下 chroot 到 arm 文件系统

sudo cp /usr/bin/qemu-arm-static ./bullseye_rootfs/usr/bin/  #arm32
sudo cp /usr/bin/qemu-aarch64-static ./bullseye_rootfs/usr/bin/  #arm64
```

### 命令解释

* –arch：指定制作的文件系统是什么架构的，armhf为ARMv7-a架构，arm64为ARMv8-a架构
* –foreign：在与主机架构不相同时需要指定此参数，仅做初始化的解包
* --keyring:：使用指定的key
* bullseye：Debian 11的发行版本代号，[Debian 发行版本代号](https://www.debian.org/releases/)
* <https://mirrors.bfsu.edu.cn/debian/> : 中国镜像服务器地址

## 进入文件系统

* 通过chroot进入到制作好的文件系统。 [chroot wiki](https://zh.wikipedia.org/wiki/Chroot)，直接使用提供的脚本进入

> 注意：windos上编辑的文本换行符和linux换行符不相同需要更换成linux的换行符否则会报错，可以使用vscode直接修改换行符，要加上可执行权限
> /bin/sh^M:bad interpreter: No such file or directory

```shell
#!/bin/bash

function mnt() {
    echo "MOUNTING"
    if [[ "${2}" != */ ]]; then
        path="${2}/"
    else 
        path="${2}"
    fi
    sudo mount -t proc /proc ${path}proc
    sudo mount -t sysfs /sys ${path}sys
    sudo mount -o bind /dev ${path}dev
    sudo mount -o bind /dev/pts ${path}dev/pts  
    sudo chroot ${path}
}

function umnt() {
    echo "UNMOUNTING"
    if [[ "${2}" != */ ]]; then
        path="${2}/"
    else 
        path="${2}"
    fi
    sudo umount ${path}proc
    sudo umount ${path}sys
    sudo umount ${path}dev/pts
    sudo umount ${path}dev

}

if [ "$1" == "-m" ] && [ -n "$2" ] ;
then
    mnt $1 $2
elif [ "$1" == "-u" ] && [ -n "$2" ];
then
    umnt $1 $2
else
    echo ""
    echo "Either 1'st, 2'nd or both parameters were missing"
    echo ""
    echo "1'st parameter can be one of these: -m(mount) OR -u(umount)"
    echo "2'nd parameter is the full path of rootfs directory(with trailing '/')"
    echo ""
    echo "For example: mount -m /media/sdcard/"
    echo ""
    echo 1st parameter : ${1}
    echo 2nd parameter : ${2}
fi

```

* 使用方法

```shell
./mount.sh -m ./bullseye_rootfs/  #挂载
./mount.sh -u ./bullseye_rootfs/  #卸载
```

* 执行脚本后，没有报错会进入文件系统，终端显示 `I have no name` ，这是正常的，这是因为根文件系统不完整，执行debootstrap二阶段，成功会自动删除这个文件夹

```shell
debootstrap/debootstrap --second-stage
```

* 如果出现了报错(systemd配置失败)，Failure trying to run:  dpkg --force-overwrite --force-confold --skip-same-version，重新安装systemd

```shell
apt install systemd
```

* 上一步还是不行可以退出删除根文件系统重新再来一次，进入第二步骤之前把镜像源换一下

## 定制文件系统

* 配置locales，语言服务，在交互界面中选择en_US.UTF-8和zh_CN.UTF-8后，点击下方接下来的默认locale配置中，选择en_US.UTF-8（此处根据需要配置）如果是ssh链接linux在终端中选择编号，在桌面环境中是直接列表选择158 en_US.UTF-8 487 
* 
* zh_CN.UTF-8，建议直接使用vscode的终端图形效果好一些

```shell
apt-get install locales -y
dpkg-reconfigure locales
```

* 配置root密码

```shell
passwd
```

* 配置时区

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

* 图形化配置时区

```shell
dpkg-reconfigure tzdata
```

* 安装软件

```shell
apt update &apt upgrade
apt install openssh-server net-tools iputils-ping sudo usbutils fdisk gdisk kmod vim ntpdate neofetch -y
```

* 网卡配置，注释只能单独成行，否则无法识别配置文件

```shell
export HOST=debian
echo $HOST > /etc/hostname
echo "127.0.0.1    localhost.localdomain localhost" > /etc/hosts
echo "127.0.0.1    $HOST" > /etc/hosts

cat >>/etc/network/interfaces <<EOF
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
 address 192.168.5.100
 gateway 192.168.5.1
iface eth0 inet6 auto 
EOF

cat >> /etc/resolv.conf <<EOF
nameserver 114.114.114.114
nameserver 8.8.8.8
EOF
```

* 编译内核要添加ipv6支持，否则用不了ipv6，ping 默认使用ipv6，未开启ipv6报警告：ping: socket: Address family not supported by protocol
* 添加普通用户ping 权限

```shell
chmod u+s /bin/ping
```

* 创建普通用户

```shell
USER=super
useradd -G sudo -m -s /bin/bash $USER 
passwd $USER
```

* 修改环境变量,修改文件/etc/environment添加PATH值，作为系统引导加载PATH，environment在启动根文件系统时加载，删除/etc/profile前边设置的PATH值,profile在用户(包含root)登录时加载

```shell
echo "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" > /etc/environment
```

* 安装终端工具xterm调整窗口终端尺寸，否则在串口终端中显示长字符会重叠,在配置文件中

```shell
apt install xterm
echo 'resize' >> /etc/profile #登录自动重置
```

* 允许通过ssh登录root账号，编辑/etc/ssh/sshd_config配置文件

```shell
vim /etc/ssh/sshd_config

#PermitRootLogin prohibit-password
PermitRootLogin yes
```

直接使用命令一次性修改

```shell
sed -i 's/^#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
```

* 串口自动登录root账号，修改 /lib/systemd/system/serial-getty@.service内容

>将ExecStart=-/sbin/agetty -o '-p -- \\u' --keep-baud 115200,38400,9600 %I $TERM
>替换为ExecStart=-/sbin/agetty -a root --keep-baud 115200,38400,9600 %I $TERM

命令一次性修改

```shell
sed -i 's/^ExecStart=-\/sbin\/agetty.*/#&/' /lib/systemd/system/serial-getty@.service
sed -i '/^#ExecStart=-\/sbin\/agetty.*/a\ExecStart=-\/sbin\/agetty -a root --keep-baud 115200,38400,9600 %I \$TERM/' /lib/systemd/system/serial-getty@.service
```

* 在用户目录下已经root用户目录下.bashrc文件中启用 ll 等命令的别名，可直接使用ubuntu的文件
* 创建模块存放目录 (可以在挂载之后进行，确认内核版本号)lib/modules/5.4.31，5.4.31是内核版本 挂载根文件之后uname -r查看，还要创建必要的模块文件(根据错误来创建)
### 使用脚本定制文件系统

```shell
#!/bin/bash

#add language package
apt install locales -y
dpkg-reconfigure locales

#add root password
passwd root

#add localtime
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

#instaall app
apt update & apt upgrade
apt install ntpdate openssh-server net-tools iputils-ping sudo wget curl usbutils fdisk gdisk parted kmod vim xterm neofetch dosfstools -y


export HOST=debian
echo $HOST > /etc/hostname
echo "127.0.0.1    localhost.localdomain localhost" >> /etc/hosts
echo "127.0.0.1    $HOST" >> /etc/hosts

#add network interface
cat >>/etc/network/interfaces <<EOF
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
 address 192.168.5.100
 gateway 192.168.5.1
iface eth0 inet6 auto 
EOF

#add dns
cat >>/etc/resolv.conf <<EOF
nameserver 114.114.114.114
nameserver 8.8.8.8
EOF

chmod u+s /bin/ping

#add user
USER=super
useradd -G sudo -m -s /bin/bash $USER 
passwd $USER

# change PATH
echo "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" > /etc/environment
sed -i '4d' /etc/profile
sed -i '4d' /etc/profile
sed -i '4d' /etc/profile
sed -i '4d' /etc/profile
sed -i '4d' /etc/profile

#auto serial login
sed -i 's/^ExecStart=-\/sbin\/agetty.*/#&/' /lib/systemd/system/serial-getty@.service
sed -i '/^#ExecStart=-\/sbin\/agetty.*/a\ExecStart=-\/sbin\/agetty -a root --keep-baud 115200,38400,9600 %I \$TERM/' /lib/systemd/system/serial-getty@.service

#allow ssh root login
sed -i 's/^#PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config

echo 'resize' >> /etc/profile
```

## 打包烧录

### 打包

```shell
exit #退出回模拟环境到主机系统
./mount.sh -u bullseye_rootfs/ #卸载设备，必须执行，否则生成磁盘文件会失败
sudo rm ./bullseye_rootfs/usr/bin/qemu-arm-static #arm32
sudo rm ./bullseye_rootfs/usr/bin/qemu-aarch64-static #arm64
sudo dd if=/dev/zero of=rootfs.ext4 bs=1M count=2048 #2048可以改成1024，上板卡后再调整根文件系统大小
sudo mkfs.ext4 -L rootfs -d ./bullseye_rootfs rootfs.ext4 #生成磁盘文件
```

### eMMC/SD分区

参考[[05_磁盘]] ，注意不同芯片支持的分区表不同，以下是MBR分区表两个分区，boot分区fat32格式，rootfs分区ext4格式，分区完成拷贝制作文件到rootfs分区中
```shell
#! /bin/sh

usage ()
{
  echo "
Usage:
  --device              SD block device node (e.g /dev/sda or /dev/sdb)
"
	exit 1
}

unmount_all_partitions ()
{
  for i in `ls -1 $device*`; do
    echo "unmounting device '$i'"
    umount $i 2>/dev/null
  done
}

# Main script starts from here
if [ `whoami` != "root" ]; then
	echo "This script should be called with sudo!!"
	usage
	exit 1
fi

while [ $# -gt 0 ]; do
  case $1 in
    --help | -h)
      usage
      ;;
    --device) shift; device=$1; shift; ;;
    *) usage ;;
  esac
done

if [ ! -b $device ]; then
   echo "ERROR: $device is not a block device file"
	 usage
   exit 1;
fi

echo "************************************************************"
echo "*         THIS WILL DELETE ALL THE DATA ON $device         *"
echo "*                                                          *"
echo "*         WARNING! Make sure your computer does not go     *"
echo "*                  in to idle mode while this script is    *"
echo "*                  running. The script will complete,      *"
echo "*                  but your eMMC may be corrupted.         *"
echo "*                                                          *"
echo "*         Press <ENTER> to confirm....                     *"
echo "************************************************************"
read junk

unmount_all_partitions
sync

cat << END | fdisk $device
n
p
1

+62M
n
p
2


t
1
c
a
1
w
END

unmount_all_partitions
sleep 3

PARTITION1=${device}1
PARTITION2=${device}2

# make partitions.
echo "Formatting ${device} ..."
if [ -b ${PARTITION1} ]; then
	mkfs.vfat -F 32 -n "boot" ${PARTITION1}
else
	echo "Cant find boot partition in /dev"
fi

if [ -b ${PARITION2} ]; then
	mkfs.ext4 -L "rootfs" ${PARTITION2}
else
	echo "Cant find rootfs partition in /dev"
fi

echo "Partitioning and formatting completed!"
```

