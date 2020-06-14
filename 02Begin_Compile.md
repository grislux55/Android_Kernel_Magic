# 二、入门编译

## **准备工作**

### 1.下载交叉编译工具链（以下工具链按推荐程度排序）

#### **`Clang`部分**

- 开发者kdrag0n提供的预编译Clang

    > [点此去](https://github.com/kdrag0n/proton-clang/releases)

- 开发者wloot提供的预编译Clang

    > [点此去](https://github.com/wloot/tc-build/releases)

- Android官方提供的Clang编译器：

    > [点此去](https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/)

#### **`GCC`部分**

- 开发者arter97提供的预编译GCC：

    > [64位](https://github.com/arter97/arm64-gcc)
[32位](https://github.com/arter97/arm32-gcc)

- 开发者kdrag0n提供的预编译GCC：

    > [64位](https://github.com/kdrag0n/aarch64-elf-gcc)
[32位](https://github.com/kdrag0n/arm-eabi-gcc)

- ARM提供的GCC：

    > 下载AArch32 target with soft float (arm-linux-gnueabi)和AArch64 GNU/Linux target (aarch64-linux-gnu)
[点此去](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads)


### 2.Git，初见

Git是一个分布式的版本控制软件，用于多人协作开发软件，由Linux之父Linus编写。

有关Git的教程可以去这里学习：

[廖雪峰的Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)

[Git Reference](https://git-scm.com/docs)

[Git Book](https://git-scm.com/book/en/v2)

我们在此次阶段主要要学习的就是Git的清除，因为在内核编译的时候会产生很多乱七八糟的文件，进行清理显得比较有必要。

#### 1.git reset

git reset顾名思义，就是可以将当前整个工作区的状态重新设置的意思，在其后面加上不同的参数可以设置不同的范围。

`--soft`：将git的状态恢复到当前状态，这项操作不会涉及文件。

`--mixed`：将git的状态恢复到当前状态，仅涉及已经提交的文件。

`--hard`：将git的状态恢复到当前状态，同时将所有已经被git跟踪的文件恢复。

#### 2.git clean

git reset会重置已经跟踪的文件，git clean则是删除未被Git跟踪的文件。

`-f`：强制删除文件

`-d`：删除目录

`-x`：同时删除被.gitignore排除的文件

### 3.编写编译脚本

#### 1.Shell脚本语法

详见[此](https://www.runoob.com/linux/linux-shell.html)

#### 2.清理脚本

我们在第一行需要写一个shell bang，用来提示系统在运行这个文件的时候要使用什么程序

1. 新建一个.sh文件，在文件中输入以下内容：

    > ```bash
    > #!/bin/sh
    > ```

2. 使用上文学到的Git命令进行清理

    > ```bash
    > git clean -fdx
    > git reset --hard
    > ```

#### 3.编译配置文件输出脚本

##### Hello Makefile

众所周知，Linux内核是使用GNU make进行构建的，所以对于GNU make我们有一本长到狗看完直接去世的[GNU make manual](https://www.gnu.org/software/make/manual/make.html)。

估计等你看完这个手册安卓已经喜提Zircon内核了......当然不会要你读这么多东西，我们只需要会修改变量就行了，因为需要指定我们的编译器路径。

1. 定义变量

    > Makefile里面定义变量非常地复杂，`=`可以普通地定义一个变量。例如：
    >
    > ```makefile
    > foo = wtf # foo的值是wtf
    > ```
    >
    > `:=`则可以定义一个只展开一次的变量。例如：
    >
    > ```makefile
    > foo = wtf
    > bar = $(foo) wtf
    > test1 = $(bar)  # test1的值为 wtf wtf
    > test2 := $(bar) # test2的值为 $(foo) wtf
    > ```
    >
    > `?=`则是按照条件定义的变量，如果这个变量已经存在了，则不进行定义。例如：
    >
    > ```makefile
    > foo = wtf
    > foo ?= wth # foo的值是wtf
    > bar ?= wth # bar的值是wth
    > ```
    > 
    > `+=`则是扩展之前已经定义过的变量。例如：
    > 
    > ```makefile
    > foo = wtf
    > foo += wth # foo的值是wtf wth
    > ```

2. 使用变量

    > 啊这，上面已经有使用过了，直接`$(变量名)`就行了

3. 变量优先级问题

    > 众所周知，make要读取很多变量，比如我们接下来要设置的`export ARCH=<目标架构类型>`这是环境变量，make命令传递进去的`make ARCH=<目标架构类型>`这是参数变量，还有在makefile里面定义的`CC := $(CROSS_COMPILE)gcc`这是普通变量。那么makefile是怎么使用它们的呢？
    > make上的变量，是以`参数变量 > 普通变量 > 环境变量`的方式来使用这些变量的，那可能就有人要说了——啊你这个那我写makefile的时候岂不是就被参数变量随便搞死了？气抖冷，普通变量什么时候才能站起来。
    > 很简单，makefile也提供了对应的语法，将普通变量的优先级提升到最高，只需要在对应的变量前面加上`override`

- 问题：

    > 啥是配置文件？

- 解答：

    > 一套源码可能被多个设备共有。在编译的时候，make通过不同的配置文件来区分这些设备，这些配置文件在`arch/[架构]/config`下，在make的时候无需指定路径，直接输入配置文件名字即可。

##### 编写脚本

1. 新建一个.sh文件，在文件中输入以下内容：

    > ```bash
    > #!/bin/sh
    > ```

2. 设置环境变量（如果将对应的变量作为参数传入则无需设置）

    > ```bash
    > # Clang
    > export ARCH=<目标架构类型>
    > export SUBARCH=<目标架构类型>
    > export CLANG_PATH=<Clang的所在目录（记得是bin文件夹）>
    > export PATH="$CLANG_PATH:$PATH"
    > export CROSS_COMPILE=<Clang的binutil的前缀>
    > export CROSS_COMPILE_ARM32=<Clang的32位binutil的前缀>
    > ```
    > 
    > 或
    > 
    > ```bash
    > # GCC
    > export ARCH=<目标架构类型>
    > export SUBARCH=<目标架构类型>
    > export GCC_PATH=<GCC的所在目录（记得是bin文件夹）>
    > export GCC32_PATH=<GCC32的所在目录（记得是bin文件夹）>
    > export PATH="$GCC_PATH:$GCC32_PATH:$PATH"
    > export CROSS_COMPILE=<GCC的前缀>
    > export CROSS_COMPILE_ARM32=<GCC的32位前缀>
    > ```

3. make及传入参数

    > ```bash
    > cd <项目文件目录>
    > make CC=<编译器> AR=<前缀>-ar NM=<前缀>-nm OBJCOPY=<前缀>-objcopy OBJDUMP=<前缀>-objdump STRIP=<前缀>-strip [O=<输出路径>] <要编译的配置文件>
    > ```

#### 4.编译脚本

1. 新建一个.sh文件，在文件中输入以下内容：

    > ```bash
    > #!/bin/sh
    > ```

2. 设置环境变量（如果将对应的变量作为参数传入则无需设置）

    > ```bash
    > # Clang
    > export ARCH=<目标架构类型>
    > export SUBARCH=<目标架构类型>
    > export CLANG_PATH=<Clang的所在目录（记得是bin文件夹）>
    > export PATH="$CLANG_PATH:$PATH"
    > export CROSS_COMPILE=<Clang的binutil的前缀>
    > export CROSS_COMPILE_ARM32=<Clang的32位binutil的前缀>
    > ```
    > 
    > 或
    > 
    > ```bash
    > # GCC
    > export ARCH=<目标架构类型>
    > export SUBARCH=<目标架构类型>
    > export GCC_PATH=<GCC的所在目录（记得是bin文件夹）>
    > export GCC32_PATH=<GCC32的所在目录（记得是bin文件夹）>
    > export PATH="$GCC_PATH:$GCC32_PATH:$PATH"
    > export CROSS_COMPILE=<GCC的前缀>
    > export CROSS_COMPILE_ARM32=<GCC的32位前缀>
    > ```

3. make及传入参数

    > ```bash
    > cd <项目文件目录>
    > make CC=<编译器> AR=<前缀>-ar NM=<前缀>-nm OBJCOPY=<前缀>-objcopy OBJDUMP=<前缀>-objdump STRIP=<前缀>-strip [O=<输出路径>] -j$(nproc --all)
    > ```

### 4.开始编译
- 依次运行：项目清理脚本、配置文件输出脚本、内核编译脚本。完成一次完整编译
- 仅运行：内核编译脚本。完成一次脏编译

### 5.内核文件
- 内核文件在[你的项目/输出路径/arch/目标内核架构/boot]下，自己复制出来去做成刷机包就好了。

### 6.简单问题解答

- 问题一：

    > 内核源码在哪里？

- 解答：

    > 对不起，这个教程并不会直接教你从零编写出一个内核（也没法教啊这个），所以内核源码是使用各种官方或非官方的已发布内核源码，这些源码都是一定程度上的针对特定设备来开发的，并不存在任何通用内核源码。

### 7.错误修复

如果您在编译时出现了任何问题，可以对照下面的常见错误进行一些错误修复，下面的错误列表收录了一些常见的错误。如果下面的错误列表中不包含您的错误，请开一个issue来报告，如果您找到了一个新的错误，可以尝试开一个pull request。

- 错误一：

    > `xxx not found`

- 解决：

    > `sudo apt-get install xxx`

- 错误二：

    > ```
    > ********************************************************************************
    > You are using version [你的make版本] of make.
    > Android can only be built by version [应当使用的make版本].
    > see http://source.android.com/source/download.html
    > ********************************************************************************
    > ```

- 解决：

    > ```
    > 1、下载对应版本的 make 压缩包:ftp://ftp.gnu.org/gnu/make/
    > 2、将make压缩包放到Ubuntu任意目录下解压
    > tar -xjvf [压缩包]
    > 3、进入make文件的目录
    > ./configure
    > make
    > sudo make install
    > ```