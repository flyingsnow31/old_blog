---
title: "Nvmedirect配置文档"
date: 2023-02-04T20:08:04+08:00
tags: ["nvmedirect"]
categories: 软件安装与配置
---

# nvmedirect配置文档

## 环境需求

使用的环境为Ubuntu 14.04/16.04

需要注意的是，直接安装好的Ubuntu 14.04/Ubuntu 16.04的linux内核版本为4.4。

注意：安装Ubuntu虚拟机时，需要将硬盘设置为NVMe而非SCSI

但是nvmedirect需要在4.11的内核版本上进行，所以需要升级一下内核版本。

方法如下

[升级 Ubuntu Linux 内核的几种不同方法 | Linux 中国 (qq.com)](https://mp.weixin.qq.com/s/6sjNdiLNfP2d9B5GGIUnsA?login=from_csdn)

通过多方比较，选用第一种方法，即使用dpkg手动安装内核。

### 升级内核

预编译好的内核文件网址如下，从中找到需要的版本，即4.11

[Index of /~kernel-ppa/mainline (ubuntu.com)](https://kernel.ubuntu.com/~kernel-ppa/mainline/)

根据自己的硬件环境，找到对应的内核版本。

将`linux-headers-...-all.deb`, `linux-headers-...-generic-....deb`, `linux-image-....deb`共三个文件下载，如果存在`linux-module-....deb`也需要下载。

将其放在同一个目录下。

执行如下命令，通过dpkg进行安装

```shell
$ sudo dpkg --install *.deb
```

然后系统自动执行安装过程，等待一小会儿后，安装完成，重启系统

```shell
$ sudo reboot
```

重新进入系统，查看内核版本

```shell
$ uname -r
4.11.0-041100-generic
```

内核升级完成

### 其他需求

需要安装gcc, make等编译相关工具，gcc版本需要在5

#### 更新gcc

由于ubuntu14的apt最高版本为gcc4，因此需要增加源安装

添加源

```shell
$ sudo add-apt-repository ppa:ubuntu-toolchain-r/test
$ sudo apt-get update
```

安装指定版本gcc

```shell
$ sudo apt-get install -y gcc-5
```

设置gcc各个版本的优先级

```shell
$ sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 90
```

切换gcc版本

```shell
$ sudo update-alternatives --config gcc
```

检查

```shell
$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/5/lto-wrapper
...
Thread model: posix
gcc version 5.5.0 20171010 (Ubuntu 5.5.0-12ubuntu1~14.04)
```

## 编译安装

### 下载源码

```shell
$ git clone https://github.com/nvmedirect/nvmedirect.git
```

### 编译内核与共享库

```shell
$ cd nvmedirect
$ make
```

### 安装

```shell
$ sudo make install
```

安装完成

### 加载内核

```shell
$ sudo modprobe nvmed
```

## 编译安装中可能存在的问题

### make失败

内核问题，需要升级到4.11，保证内核版本在4.13以下

### make install失败

ssl出错，报错如下

```shell
At main.c:160:
- SSL error:02001002:system library:fopen:No such file or directory: bss_file.c:169
- SSL error:2006D080:BIO routines:BIO_new_file:no such file: bss_file.c:172
sign-file: certs/signing_key.pem: No such file or directory
```

此为内核签名问题，执行如下操作

[(25条消息) ubuntu16.04驱动安装提示sign-file: certs/signing_key.pem: No such file or directory_小码农9527的博客-CSDN博客](https://blog.csdn.net/u011050845/article/details/125699628)

创建 `x509.genkey` 文件

```
[ req ]
default_bits = 4096
distinguished_name = req_distinguished_name
prompt = no
string_mask = utf8only
x509_extensions = myexts
 
[ req_distinguished_name ]
CN = Modules
 
[ myexts ]
basicConstraints=critical,CA:FALSE
keyUsage=digitalSignature
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid
```

执行如下命令

```shell
$ openssl req -new -nodes -utf8 -sha512 -days 36500 -batch -x509 -config x509.genkey -outform DER -out signing_key.x509 -keyout signing_key.pem
```

复制密钥到系统内核源码的证书目录

```shell
$ sudo cp signing_key.pem signing_key.x509 /usr/src/linux-headers-4.11.0-041100-generic/certs/
$ sudo cp signing_key.pem signing_key.pem /usr/src/linux-headers-4.11.0-041100-generic/certs/
```

### 加载内核失败

[linux - 我做 modprobe 时找不到模块 - SegmentFault 思否](https://segmentfault.com/q/1010000042783853)

报错如下

```shell
$ sudo modprobe nvmed
modprobe: FATAL: Module nvmed not found.
```

首先确认此目录下存在编译好的内核文件

```shell
$ ls /lib/modules/4.11.0-041100-generic/extra
nvmed.ko
```

然后执行如下命令

```shell
$ sudo depmod
$ sudo modprobe nvmed
```

加载成功

