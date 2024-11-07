# 通用配置项

![通用配置项](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_00.png)

# boot配置项

![boot配置项](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_01.png)

在自动配置项中有自动boot倒计时时长和提示语

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_02.png)

# 命令行接口

![命令行接口](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_03.png)

## boot命令重要配置项

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_04.png)

## 环境变量命令重要配置项

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_05.png)

## 内存命令重要配置项

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_06.png)

## 设备访问命令配置项

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_07.png)


## 网络命令

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_08.png)

## 文件系统访问命令

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_09.png)

# 设备树

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_10.png)

# 环境变量

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_11.png)

## 常用保存位置

### 保存到ext4文件系统中

#### 配置项
例如：保存/uboot.env文件到 **mmc0**设备有et4文件系统的分区4中，在有mmc设备驱动且配置了ext4文件系统可写情况下还需要配置一下项

```c
CONFIG_ENV_IS_IN_EXT4=y  
CONFIG_ENV_EXT4_DEVICE_AND_PART="0:1"  
CONFIG_ENV_EXT4_FILE="/uboot.env"  
CONFIG_ENV_EXT4_INTERFACE="mmc"
```

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/u-boot/01/u-boot_01_12.png)
#### u-boot加载环境变量报错

```shell
Saving Environment to EXT4... Unsupported feature metadata_csum found, not writing.

Unsupported feature metadata_csum found, not writing.
** Error ext4fs_write() **
** Unable to write file u-boot-dtb.imx **
```

#### 解决思路
需要重新格式化 ext4 的分区，默认使用 mkfs.ext4 格式化的分区，会报这个错误，解决方法：在 mkfs.ext4 命令后面加人参数 -O ^metadata_csum，如下
```shell
$ sudo mkfs.ext4 -O ^metadata_csum /dev/sdb4
```

### 保存到fat文件系统中


### 保存到flash中


### 保存到mmc中