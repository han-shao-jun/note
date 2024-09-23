与驱动框架相关的api都在driver/core文件夹下
##  获取udevice属性
api全部是以dev_开始
- dev_get_driver_data，获取匹配表中的data字段
- dev_read_addr，获取设备树reg属性值

## uclass