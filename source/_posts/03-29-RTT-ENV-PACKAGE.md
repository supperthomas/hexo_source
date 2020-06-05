title: '03-29[RTT][ENV][PACKAGE]如何制作软件包'
author: Thomas Li
abbrlink: 3349906663
tags:
  - RT-THREAD
  - ENV
  - PACKAGE
categories:
  - RTOS
date: 2020-03-29 10:39:00
---
## 1.官方资料

首先看下官方文档：

>> https://www.rt-thread.org/document/site/development-guide/package/package/

2. 再看下官方视频：

>>https://www.bilibili.com/video/av79943543?p=1

-----
看完这个，基本上差不多会知道如何做一个软件包了。

![upload successful](/image/2020_03_29_01_0.png)

pkgs --wizard
新版本会让你输入github地址
操作完成之后。


会生成一个文件夹，两个文件


![filename already exists, renamed](/image/2020_03_29_01_1.png)

#### Kconfig 
这个主要和平时操作menuconfig中的选项相关

#### package.json
这个主要跟你这个软件包的git地址相关。

简单来说软件包这就算做好了。
至于里面内容需要分几个东西来讲：
分别是
`pkgs`
`menuconfig`
`scons`


### 1.pkgs
这个工具是专门管理软件包的。
有以下几个常用命令：
#### `pkgs --update`

这个负责升级，其实本质上就是git clone，从git上下载代码；
可以看下package.json文件：


![filename already exists, renamed](/image/2020_03_29_01_2.png)
这张图应该很清晰了。
就是你的每个版本对应的git 地址填对了
你的`pkgs --update` 的时候，更新就会从这个git地址下载你的package包的文件（如果某个package被选中的话）。

#### `pkgs --upgrade`
这个用的比较少，

![filename already exists, renamed](/image/2020_03_29_01_3.png)


从两个地址，

`package:
https://github.com/RT-Thread/packages
`

这个下载到哪里了呢？
我们打开env目录：

`env_released_1.2.0\env\packages\packages`

就是这里，其实本质上，就是更新packages的目录



`env:
https://github.com/RT-Thread/env`

本质上也是从git上下载，只是路径不一样。

这个稍微找一会也能找到：

`env_released_1.2.0\env\tools\scripts`

大家可以打开github和文件夹目录比较下，
应该都是相同的。


![filename already exists, renamed](/image/2020_03_29_01_4.png)



![filename already exists, renamed](/image/2020_03_29_01_5.png)




### 2. menuconfig


这个是linux里面常用的配置界面，嵌入式RTOS能用上menuconfig的,我只见过RTT这样用过，其他的还么见过：



![filename already exists, renamed](/image/2020_03_29_01_7.png)

不知道有没有人稍微了解下这几行命令是如何来显示的。

其实就是目录下面的`Kconfig` 文件


```
mainmenu "RT-Thread Configuration"

config BSP_DIR
    string
    option env="BSP_ROOT"
    default "."

config RTT_DIR
    string
    option env="RTT_ROOT"
    default "../../.."

config PKGS_DIR
    string
    option env="PKGS_ROOT"
    default "packages"
 
source "$RTT_DIR/Kconfig"
source "$PKGS_DIR/Kconfig"
source "../libraries/Kconfig"
source "board/Kconfig"
```

其实这个里面内容比较多，就不展开了，我们现在要做的是软件包，可以选择在某个package中menuconfig。
比如你可以进到目录`env_released_1.2.0\env\packages\packages\peripherals\sensors\dht11`

menuconfig一下：

![filename already exists, renamed](/image/2020_03_29_01_8.png)

看到这个应该能明白大概意思了。
bool类型，只能选y(yes) 和n（no）
后面跟的是显示的内容


menuconfig代表是一个menu配置 后面跟的是`PKG_USING_DHT11`
这个是什么呢？这个其实和下一讲的scons相关，
你可以暂且认为这个就代表这个选项的一个变量，就是敲`y`之后，这个变量值PKG_USING_DHT11就变成y了（可以理解成bool类型中的true）。

子菜单：

![filename already exists, renamed](/image/2020_03_29_01_9.png)

仔细想下刚才的操作，是不是只能点y之后才能进入到子菜单。
其实就是前面有个

`if PKG_USING_DHT11  `
判断
后面
Kconfig语法可以看：

>> [kconfig语法整理](https://www.jianshu.com/p/aba588d380c2)

基本上会Kconfig语法，你就知道menuconfig该怎么改了，唯一要记住的是刚才那个变量。

配置完之后，系统都会保存到.config文件：
这个是最重要的，前面无论你怎么配置，最后都影响的是.config文件。
你也可以直接改.config文件。



![filename already exists, renamed](/image/2020_03_29_01_10.png)

这里你可以看到配置改了，所以有时候，编译linux内核的时候，只要.config文件就可以编译成功了，就是这个道理。
保存之后，
我们用命令pkgs --update，就会看到package中会多出来一个文件，就是你的软件包。

//小测问：
github 我们国内clone的时候，都很慢，为啥env下载的时候很快？

### 3.scons

在做软件包前面基本已经差不多了。接下来讲的scons，其实是和编译相关的，你可以理解为，scons就是make， scons里面的`SConscript` `SConstruct`就是里面的makefile文件，你软件包需要写好写对Scons的脚本，才能编译进去，或者生成的project生成进去。

这个时候就看官方文档：

[Scons 构建工具](https://www.rt-thread.org/document/site/programming-manual/scons/scons/)

官方文档已经很详细很详细了。学完，你就会用你自己的软件包。


理解这些，基本上你应该知道自己接下来该怎么做了：
基本思路是这样的：


1. 先做一个软件包，把名字起好，对应到相应的目录，比如tools。
2. github上搞一个仓库（gitee也可以，只要路径填对即可），将你的软件包中要放的.c .h等代码放入到git上。
3. 将github地址放到你的package.json中
4. 把这个软件包放到env中package对应的目录中，（暂时）
5. 这个时候你在你的bsp里面敲menuconfig，会发现，你的软件包并没有出现在目录中。
6. 根据Kconfig语法，知道是在你的tools目录下面也有个Kconfig，需要把目录加进去。
7. menuconfig里面配置选好，pkgs --update
8. 这个时候，检查你的包里面的东西的完整性就好。
9. 根据scons语法和Kconfig语法，写你的软件包中的scons语法，如果是配置的话，要放到env中的package   Kconfig修改。
10. 基本上把你的目录提交到官方的package收入即可。

`