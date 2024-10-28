
#### 1、make defconfig

　　首先通过make xxx_defconfig，生成最开始的.config，相当于把 XXX_defconfig 文件复制为 .config 文件，其中 defconfig 是最小的 config 项，kernel编译会根据 .config 文件去编译驱动情况，加载过改指令后，后面的 make  menuconfig 就会基于现在的 .config 去配置 config ；

#### 2、make  menuconfig

　　make  menuconfig 的作用类似于 make  config ，就是基于界面去配置 config 文件，make  config 的作用是加载 “ .config ” 作为默认的配置，配置它就是相当于用图形化界面配置 .config 文件；

#### 3、make  savedefconfig

　　你完成自定义配置后，可以使用`make savedefconfig`用于将当前内核`.config`文件配置精简保存为一个“最小的默认配置”，在当前路径下生成一个`defconfig`文件。这个文件只包含与默认配置不同的选项，从而使其更为紧凑。这个文件可以方便地用作项目或设备的默认配置

#### 4、make  olddefconfig

　　通过make oldconfig将刚增加的config项的.config做依赖检查重新生成新的.config文件，且新生成的.config和以前的不同是，将旧的.config重命名为.config.old文件