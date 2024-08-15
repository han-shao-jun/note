@[TOC]
## 前言
buildroot 比 busybox 更上一层楼，buildroot 不仅集成了 busybox，而且还集成了各种常见的第三方库和软件。buildroot 和 uboot、 kernel 很类似，我们需要到其官网上下载源码，然后对其进行配置，比如设置交叉编译器、设置目标 CPU 参数等，最主要的就是选择所需要的第三方库或软件。本文记录如何构建BuildRoot根文件系统

***

## 下载源码

进入[官网]([Buildroot - Making Embedded Linux Easy](https://buildroot.org/))下载源码
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_00.png)

选择xz压缩包比较小

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_01.png)

下载完成上传到linux系统中解压

```sh
tar -xvf buildroot-2024.05.tar.xz
```

解压获取以下内容

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_02.png)

## 配置

buildroot 和 uboot、 Linux kernel 一样也支持图形化配置，输入如下命令即可打开图形化配 置界

```sh
make meunconfig
```

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_03.png)

1.配置Target options

| -> Target Architecture         | = ARM (little endian) |
| ------------------------------ | --------------------- |
| -> Target Architecture Variant | = cortex-A7           |
| -> Target ABI                  | = EABIhf              |
| -> Floating point strategy     | = NEON/VFPv4          |
| -> ARM instruction set         | = ARM                 |
完成如下
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_04.png)

2、配置 Toolchain

此配置项用于配置交叉编译工具链，也就是交叉编译器，这里设置为我们自己所使用的交  叉编译器即可。 buildroot 其实是可以自动下载交叉编译器的，但是都是从国外服务器下载的，鉴于国内的网络环境，强烈推荐大家设置成自己所使用的交叉编译器。需要配置的项目和其对应的内容如下。主要编路径与平常使用环境变量不同要填包含bin文件的绝对路径。
例如编译可执行文件路径为以下
/home/super/mp157/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf/bin/arm-none-linux-gnueabihf-gcc
该路径应该为/home/super/mp157/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf，与下图相同
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_05.png)

3、 配置 System configuration  
此选项用于设置一些系统配置，比如系统主机名字、欢迎语、用户名、密码等。需要配置的 项目和其对应的内容如下：

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_06.png)

4、配置 Filesystem images
根文件系统格式选择ext4

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_07.png)

5.配置 Target packages
选择第三方软件
模块加载工具
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_08.png)

ssh
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_09.png)

ftp
![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_10.png)

## 编译

```sh
sudo make FORCE_UNSAFE_CONFIGURE=1
```

在以下路径有编译好的根文件系统，ext4格式的和tar压缩包，rootfs.ext4可以直接使用烧录工具烧录，rootfs.tar可以直接解压用nfs挂载测试

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/linux_driver/03/driver_03_11.png)