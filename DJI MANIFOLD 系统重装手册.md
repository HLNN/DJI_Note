# DJI MANIFOLD 系统重装手册

Time:2018-11-10

### 写在最前

内容主要源于[Manifold用户手册]和[MANIFOLD官网教学视频]。本文主要结合网上的博文对官方资料做了一定整理，记录自己的实际安装过程，方便以后查阅。

## 参考资料

  - [MANIFOLD官网教学视频]：MANIFOLD的官方教学视频，MANIFOLD是基于嵌入式的CUP和GPU，不熟悉嵌入式的话安装会比较麻烦
  - [MANIFOLD官网页面]：MANIFOLD相关的所有的信息和资料都可以在这里找到
  - [Manifold用户手册]：用户手册下载地址，用户手册会告诉你如何制作安装镜像，安装相关软件
  - [Manifold系统镜像包]：Manifold系统镜像下载地址


## 连接MANIFOLD

在重装系统之前首先要进入恢复模式。在恢复模式下可以进行制作镜像，恢复出厂设置，制作镜像等操作，进入恢复模式的方法大致如下：

  1. 首先你需要一台运行Linux的计算机且磁盘剩余空间大于16GB
  2. 用USB线连接Manifold和电脑，并连接Manifold电源线
  3. 按住Manifold的Recovery键，在短按电源键上电
  4. 松开Recovery键进入恢复模式
  5. 在Linux计算机的终端输入`lsusb`，如果出现`NVIDIA Corp`则说明进入恢复模式成果

上面这个是在DJI官方的视频中所用的方法，Manifold进入恢复模式的时候是没有图形界面的，所以之后的操作都是在Linux的计算机的终端中完成。


## 下载解压安装包

  - 访问[Manifold系统镜像包]下载Manifold镜像文件`manifold_image_v1.0.tar.gz`，镜像文件的大小为938MB
  - 执行以下命令解压安装包：

```sh
mkdir ~/manifold
cd ~/manifold
sudo tar -xvpzf <your path>/manifold_image_v1.0.tar.gz
```


## 制作系统镜像

制作系统镜像这一部分有的文件夹或者文件名可能会和官方手册上的不一样。实际安装是一定要注意，一步步跟着手册来做，但文件路径和文件名以实际的解压文件为准。

  - 执行如下命令进入`bootloader`文件夹：

```sh
cd ~/manifold/Linux_for_Tegra/bootloader
```

这个地方官方手册上写的文件路径是`~/manifold/Linux_for_Tegra/bootloader`,但`~/manifold`文件夹中只解压出来了一个叫`Linux_for_Tegra`的文件夹，所以把命令中的文件夹名字替换一下就好。

如果在这个文件夹里以及有叫做`system.img`的镜像，把他重命名备份一下就行，当然你想直接删了也行。

  - 执行如下命令开始制作系统镜像：

```sh
sudo ./nvflash --read APP system.img --bl arbdeg/fastboot.bin --go
```

输入密码时候就会开始制作系统镜像，官方说制作大概需要二十五分钟，但我i3笔记本做了整整42分钟。

还有一个需要注意的就是在上面这一行命令里面所有的参数前面都是`--`，这是**两个**减号。在一些Markdown中这两个减号可能连在一起，看起来就像一杠一样了。我在看网上一些早些年的博文的时候，发现DJI的官方手册之前好像也错写成`-`。另外如果是手输命令的话，这里的文件名都是可以`<Tab>`自动补全的，方便很多也能免去了输错文件名的风险。

完成之后输入`ls`并回车，如出现`system.img`则镜像制作成功。

## 恢复系统镜像

恢复镜像可选择恢复至默认镜像，及恢复出厂设置或者恢复到之前制作的镜像。

   - 恢复之前先确保自己在镜像解压文件夹的主路径下：

```sh
pwd
# ~/manifold/Linux_for_Tegra
```

   - 如不在则切换至该目录：

```sh
cd ~/manifold/Linux_for_Tegra
```

   - 若想恢复出厂默认镜像，运行一下命令将重新制作文件系统镜像并烧录：

```sh
sudo ./flash.sh jetson-tk1 mmcblk0p1
```

   - 若想恢复到之前制作的镜像，运行一下命令录：

```sh
sudo ./flash.sh -r jetson-tk1 mmcblk0p1
```

如果结果中出现`flashed successfully`则烧录成功，否则重新烧录。

理论上恢复出厂默认镜像大约需要三分钟，恢复自己做的镜像大约二十五分钟。

我在第一次烧录的时候电脑还卡死了，等了半个多小时也没结果。关掉当前进程并在新的终端里重新尝试的时候，报错显示没有找到USB设备，但我执行`lsusb`的时候是有`NVIDIA Corp`的。只好重启了一次Manifold，之后烧录正常，用时 分钟。


## 装载镜像

在视频里有这一个步骤，但在用户手册里是没有的。而且在执行的时候基本上马上就完成了，不像之前制作烧录镜像至少需要二三十分钟，所以我觉得这一步应该是把前面制作的镜像挂在到笔记本的`/mnt`目录下，这样在笔记本上就能进行固件的开发，对于我们没有固件开发的需求 的时候这一步就可以省略了，如有需要，执行一下命令。

   - 进入`bootloader`文件夹：

```sh
cd ~/manifold/Linux_for_Tegra/bootloader
```

   - 查看文件夹中的文件，确定存在`system.img`文件：

```sh
ls
# 输出中有"system.img"即可
```

   - 若存在，输入一下命令将镜像装载在`mnt`文件夹下：

```sh
sudo mount -t ext4 -o loop system.img /mnt
```


[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)


  
   [MANIFOLD官网教学视频]: <https://www.dji.com/cn/manifold/info#video>
   [MANIFOLD官网页面]: <https://www.dji.com/cn/manifold>
   [Manifold用户手册]: <https://dl.djicdn.com/downloads/manifold/20170918/Manifold_User_Manual_v1.2_CH.pdf>
   [Manifold系统镜像包]: <https://dl.djicdn.com/downloads/manifold/manifold_image_v1.0.tar.gz>
   [Onboard-SDK]: <https://github.com/dji-sdk/Onboard-SDK>
   [Onboard-SDK-ROS]: <https://github.com/dji-sdk/Onboard-SDK-ROS>
   [DJI Assistant 2]: <https://www.dji.com/cn/manifold/info#download>
   [Vision Map]: <https://developer.dji.com/onboard-sdk/documentation/appendix/versioning.html>
   
   
