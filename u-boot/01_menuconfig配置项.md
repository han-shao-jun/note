

# 环境变量

## 一般配置项

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