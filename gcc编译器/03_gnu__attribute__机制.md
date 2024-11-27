## attribute 基本概念
__attribute__是GCC编译器提供的一种扩展语法，用于指定变量、函数、类型等的属性。__attribute__可以用于指定变量的对齐方式、函数的调用约定、类型的大小、编译放置的段等属性。特别是在Linux内核中使用量大，有些代码的实现依赖这个机制，在u-boot中也有使用。一般__attribute_在函数返回参数前，变量名之后，变量类型之前

```c
__attribute__((__weak__)) void reset_misc(void)
{
}

__attribute__((__unused__)) int ret = 0;

int ret __attribute__((__unused__));

```

## 常用__attribute__

### __attribute__((__weak__))
函数弱定义，当有两个同名函数被定义时，被标记的函数会被忽略，”类似c++中的重载“

```c

__attribute__((__weak__)) int fun() 
{

}

int fun() 
{
  printf("in %s\n",__FUNCTION__);
  return 0;
}

```
### __attribute__((unused))
标记函数/变量可能不使用，但编译时不要报函数/变量未使用警告

### __attribute__((__used__))
表示函数/变量 **“被使用了”**，避免函数/变量因被未显式使用编译器将其优化掉(删除)

### __attribute__((__section__(“ram_func”)))
指定函数/变量在被编译时放置到ram_func段中

### __attribute__((__packed__))
指定结构体或联合体的成员按照1字节对齐

### __attribute__((__aligned__(n)))
指定变量的对齐方式，n表示对齐字节数

### __attribute__((__always_inline__))
强制函数内联

### __attribute__((__alias__(“symbol”)))
设置函数的别名为symbol
```c
int __fun() 
{
  printf("in %s\n",__FUNCTION__);
  return 0;
}

//此函数相当于 __fun
int fun() __attribute__((alias("__fun")));


```

## __attribute__机制实战用例

+ u-boot链接器器数组
+ Linux initcal