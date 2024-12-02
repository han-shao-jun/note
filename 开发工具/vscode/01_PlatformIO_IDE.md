## 前言
vscode是目前程序员使用最多的文本编辑器，其中PlatformIO_IDE插件是一个用于开发嵌入式程序的插件，其提供了编译、调试、烧录、书写代码等功能，可开发esp32系列、stm32系列、Arduino系列、51单片机，极大程度上解决了有需要使用多种芯片平台嵌入式开发者需求。

## 安装
PlatformIO_IDE是由乌克兰人开发，插件本身是托管在微软vscode插件服务器上的，但是一些项目依赖在国外的服务器上，国内网络环境访问不到。经过查看插件相关的[PlatformIO Core的源码](https://github.com/platformio/platformio-core/blob/develop/platformio/package/download.py)发现是使用Python写的，下载依赖包是通过requests模块实现的，可以设置代理(科学上网)来加速下载，在其他很多帖子中并没有提到这点，导致安装或第一次初始化项目时失败(超时无法下载)。最简单'方法添加`HTTP_PROXY` 和 `HTTPS_PROXY` 两个环境变量，指向代理地址和端口。在打开了科学上网工具后，先用vscode 插件商店下载插件，还是不能下载可以按照[官网](https://docs.platformio.org/en/latest/core/installation/methods/installer-script.html)文档通过命令行下载(还是要依赖代理)。

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/dev_tool/vscode/01/vscode_01_00.png)

把上图中的地址复制粘贴到浏览器地址栏出现以下界面(正常的)，再**全选**复制保存在本地文件get-platformio.py

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/dev_tool/vscode/01/vscode_01_01.png)

打开power shell或者git bash终端，先添加代理，再执行上文保存的Python脚本

```shell
# 根据实际情况修改
export http_proxy='http://localhost:7890'
export https_proxy='http://localhost:7890'
python3 get-platformio.py
```

安装成功需要添加路径到PATH环境变量中，注意路径可能不一样(windos用户名不同)根据实际情况来。

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/dev_tool/vscode/01/vscode_01_02.png)

## 初始化

第一次新建项目时会下载对应开发的依赖包，为了方便加速下载依赖包，先在命令行中新建一个hello项目，后续就可以在vscode中直接新建项目了。添加环境变量后关闭终端，执行以下命令打开图形化界面(浏览器)

```shell
export http_proxy='http://localhost:7890'
export https_proxy='http://localhost:7890'
pio home
```

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/dev_tool/vscode/01/vscode_01_03.png)

浏览器界面，点击新建一个项目

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/dev_tool/vscode/01/vscode_01_04.png)

以安信可esp32s2为例

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/dev_tool/vscode/01/vscode_01_05.png)

等待下载依赖包，**注意没有代理**这一步需要很长时间甚至失败

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/dev_tool/vscode/01/vscode_01_06.png)

完成后制自动打开保存的文件夹，使用vscode打开文件夹，在src文件夹下添加**main.cpp**，注意是**cpp文件**(原因不是本文重点)，再点下方工具栏 **√** 图标开始编译，旁边是烧录按钮。完成这些步骤就可以coding了

![](https://blog-1305120110.cos.ap-shanghai.myqcloud.com/dev_tool/vscode/01/vscode_01_07.png)

