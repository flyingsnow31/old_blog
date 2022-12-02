---
title: "Lab1 Utilities"
date: 2022-11-26T14:30:58+08:00
description: "6.S081 Lab1"
tags: ['操作系统', '6.S081', 'Utilities']
categories: '实验'
---

> 本实验用于熟悉xv6和系统调用

## Boot xv6(easy)

启动xv6系统

首先下载mit提供的针对此次实验设计的xv6版本，而非整体的xv6

```shell
$ git clone git://g.csail.mit.edu/xv6-labs-2020
Cloning into 'xv6-labs-2020'...
...
$ cd xv6-labs-2020
$ git checkout util
Branch 'util' set up to track remote branch 'util' from 'origin'.
Switched to a new branch 'util'
```

在这一步可以将其远程库切换成自己的。

使用 `make qemu` 指令编译并通过qemu启动xv6

```shell
make qemu
riscv64-unknown-elf-gcc    -c -o kernel/entry.o kernel/entry.S
riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -DSOL_UTIL -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o kernel/start.o kernel/start.c
riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -DSOL_UTIL -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o kernel/console.o kernel/console.c
...
nmeta 46 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 1) blocks 954 total 1000
balloc: first 591 blocks have been allocated
balloc: write bitmap block at sector 45
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$
```

输入 `ls` 命令，可以查看当前目录下的所有信息

```sh
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2059
xargstest.sh   2 3 93
cat            2 4 23968
echo           2 5 22800
forktest       2 6 13168
grep           2 7 27320
init           2 8 23896
kill           2 9 22768
ln             2 10 22720
ls             2 11 26208
mkdir          2 12 22872
rm             2 13 22856
sh             2 14 41752
stressfs       2 15 23872
usertests      2 16 147512
grind          2 17 37984
wc             2 18 25104
zombie         2 19 22272
console        3 20 0
```

通过执行 `Ctrl` + `p` 可以实现类似 `ps` 的操作，查看现在执行的所有进程

```sh
$
1 sleep  init
2 sleep  sh
```

要退出qemu，执行 `Ctrl` - `a` `x`

**注意** 先按 `Ctrl` - `a`  松开后按 `x`

## sleep(easy)

实现xv6的Unix程序睡眠；您的睡眠应该暂停一个用户指定的节拍数。计时是由xv6内核定义的时间概念，即来自计时器芯片的两个中断之间的时间。您的解决方案应该在文件USER/sleep.c中。

- 首先应该阅读 https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf xv6 book的第一章节

- 将文件实现放在 `user/` 目录下，可以参考其中的 echo.c, grep.c, rm.c 等

- 处理命令行参数个数问题

- 使用 `atoi()` 转换字符串到整型

- 使用系统调用sleep

- 查看 `kernel/sysproc.c` 中，xv6内核关于sleep系统调用的实现，以及 `user/usys.S` 关于汇编代码中跳转的部分

- 确保main函数执行完成后调用 exit()退出程序

- 将sleep加入到Makefile中的UPROGS变量中，以便在 `make qemu` 时，对其进行编译并能在xv6的shell中调用


实现如下

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[]) 
{
    if(argc < 2) {
        fprintf(2, "Usage: sleep time...\n");
        exit(1);
    }
    int n;
    n = atoi(argv[1]);
    sleep(n);
    exit(0);
}
```

在makefile中添加

```makefile
UPROGS=\
	$U/_cat\
	$U/_echo\
	$U/_forktest\
	$U/_grep\
	$U/_init\
	$U/_kill\
	$U/_ln\
	$U/_ls\
	$U/_mkdir\
	$U/_rm\
	$U/_sh\
	$U/_stressfs\
	$U/_usertests\
	$U/_grind\
	$U/_wc\
	$U/_zombie\
	$U/_sleep\
```

编译测试

```sh
make qemu
riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -DSOL_UTIL -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o user/sleep.o user/sleep.c
riscv64-unknown-elf-ld -z max-page-size=4096 -N -e main -Ttext 0 -o user/_sleep user/sleep.o user/ulib.o user/usys.o user/printf.o user/umalloc.o
riscv64-unknown-elf-objdump -S user/_sleep > user/sleep.asm
riscv64-unknown-elf-objdump -t user/_sleep | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$/d' > user/sleep.sym
mkfs/mkfs fs.img README  user/xargstest.sh user/_cat user/_echo user/_forktest user/_grep user/_init user/_kill user/_ln user/_ls user/_mkdir user/_rm user/_sh user/_stressfs user/_usertests user/_grind user/_wc user/_zombie user/_sleep
nmeta 46 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 1) blocks 954 total 1000
balloc: first 615 blocks have been allocated
balloc: write bitmap block at sector 45
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 1 starting
hart 2 starting
init: starting sh
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2059
xargstest.sh   2 3 93
cat            2 4 23968
echo           2 5 22800
forktest       2 6 13168
grep           2 7 27320
init           2 8 23896
kill           2 9 22768
ln             2 10 22720
ls             2 11 26208
mkdir          2 12 22872
rm             2 13 22856
sh             2 14 41752
stressfs       2 15 23872
usertests      2 16 147512
grind          2 17 37984
wc             2 18 25104
zombie         2 19 22272
sleep          2 20 22744
console        3 21 0
$ sleep 10
$
```

执行成功

## pingpong(easy)

实现一个程序，fork一个子进程，在父进程和子进程之间，使用pipe管道进行信息的传递与发送，最后按照格式输出相应的内容和进程号。

需要查看xv6 book 1.2和1.3章节的相关内容

具体实现如下。

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[]) 
{
    int p1[2], p2[2];
    pipe(p1);
    pipe(p2);
    if(fork() == 0) {
        char buf[20];
        int fd1 = dup(p1[0]);
        read(fd1, buf, sizeof(buf));
        if(strcmp("ping", buf) == 0) {
            printf("%d:received ping\n", getpid());
            write(p2[1], "pong\0", 5);
        }
        close(p1[0]);
        close(p1[1]);
        close(p2[0]);
        close(p2[1]);
        exit(0);
    }
    else{
        write(p1[1], "ping\0", 5);
        char buf[20];
        int fd2 = dup(p2[0]);
        read(fd2, buf, sizeof(buf));
        if(strcmp("pong", buf) == 0) {
            printf("%d:received pong\n", getpid());
        }
        close(p1[0]);
        close(p1[1]);
        close(p2[0]);
        close(p2[1]);
    }
    exit(0);
}
```

将其加入到makefile后，编译执行

```shell
$ pingpong
4:received ping
3:received pong
$
```

执行成功