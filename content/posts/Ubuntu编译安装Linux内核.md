---
title: "Ubuntu编译安装Linux内核"
date: 2023-02-04T20:08:22+08:00
draft: true
tags: ["Ubuntu", "内核"]
categories: 软件安装与配置
---

# Ubuntu编译安装Linux内核

> 以安装Linux4.10版本为例

## 总览

```shell
$ sudo apt update
$ sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
$ wget http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/v4.x/linux-4.10.tar.xz
$ tar xfJ linux-4.10.tar.xz
$ cd linux-4.10/
$ cp /boot/config-$(uname -r) .config
$ make menuconfig
$ make -j$(nproc)
$ sudo make modules_install
$ sudo make install
$ sudo update-initramfs -c -k 4.10.0
$ sudo update-grub
```

## 下载需要的内核版本

官网为https://www.kernel.org/

可以通过镜像站，找到更加详细直观的版本

http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/

```shell
$ wget http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/v4.x/linux-4.10.tar.xz
$ tar xfJ linux-4.10.tar.xz
$ cd linux-4.10/
```

## 环境配置

安装以下软件用于编译安装，如有需要，换源且`sudo apt update`

```shell
$ sudo apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
```

## 配置内核

对内核进行配置是为了得到内核配置文件.config。通过对内核进行配置，可以使未来编译成功的内核增加或减少对一些内核特性的支持。对内核进行配置有多种方法，有基于文本的配置方式也有基于图形的用户界面。下面采用使用比较广泛的make menuconfig方式

在正式编译内核之前，我们首先必须配置需要包含哪些模块。实际上，有一些非常简单的方式来配置。使用一个命令，你能拷贝当前内核的配置文件，然后使用可靠的 menuconfig 命令来做任何必要的更改。

```shell
$ cp /boot/config-$(uname -r) .config
```


上述命令的意思就是拷贝对应当前内核配置到当前目录下并重命名为.config(文件名前加.意思为隐藏文件)。
之后在解压后得内核目录里执行命令

```shell
$ make menuconfig
```


在这里可以选择启用或者禁用一些模块。如果你不知道该如何选择的话，默认退出即可。

我们这里对内核按照默认的配置方式进行编译，因此当配置菜单启动后直接退出并保存即可。此时就在内核源码根目录下生成了.config文件。

## 编译

编译内核包含两部分的工作，其一是编译内核，即编译配置选项中标记为Y的那部分，这部分内核最终形成bzIamge镜像文件；其二是编译内核模块，即编译配置选项中标记为M的那部分内核，这部分形成以.ko结尾的内核模块目标文件。

上述两部分编译工作可以依次通过make bzImage和make modules完成，也可以通过一条make命令直接完成。编译内核的整个过程比较漫长，因此可以对make加-j参数来提高编译的效率。在make时使用该选项会为编译过程分配n个并发任务，这样可以缩短编译时间。

```shell
$ make -j$(nproc)
```

## 安装

安装过程分为两部分，首先对内核模块进行安装，这个过程会将刚刚编译内核模块时生成的内核模块复制到/lib/modules/4.10.0/目录下，其中4.10.0为对应的内核版本。使用的命令如下：

```shell
$ sudo make modules_install
$ sudo make install
```

## 启用内核作为引导

```shell
$ sudo update-initramfs -c -k 4.10.0
```

数字改为自己的版本号

## 更新grub

使用如下命令自动更新启动菜单

```shell
$ sudo update-grub
```



