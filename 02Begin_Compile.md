# 二、入门编译
## **准备工作**
### 1.下载交叉编译工具链
#### **`Clang`部分**

- 下载Android官方提供的Clang编译器：
> Prebuilt Versions
文件：`linux-x86-master-clang-[版本].tar.gz`
[点此去](https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/)

- 下载完整的GCC工具链：
> AArch32 target with soft float (arm-linux-gnueabi)
文件：`gcc-arm-8.3-[日期]-arm-linux-gnueabi.tar.xz`
AArch64 GNU/Linux target (aarch64-linux-gnu)
文件：`gcc-arm-8.3-[日期]-x86_64-aarch64-linux-gnu.tar.xz`
[点此去](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads)

- 说明：
> 我们在编译时使用Clang编译器与GCC的其他工具组合而成的工具链，因为这样可以减少由于Clang造成的异常。工作流程如下：

![工作流程](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/compile_flow.png)

- 解压：
> 使用tar -Jxf [文件名]来解压.tar.xz文件（会解压到新文件夹内）
使用tar -zxf [文件名]来解压.tar.gz文件（会解压到当前文件夹内）

#### **`GCC`部分**

- 下载完整的GCC工具链：
> AArch32 target with soft float (arm-linux-gnueabi)
文件：`gcc-arm-8.3-[日期]-arm-linux-gnueabi.tar.xz`
AArch64 GNU/Linux target (aarch64-linux-gnu)
文件：`gcc-arm-8.3-[日期]-x86_64-aarch64-linux-gnu.tar.xz`
[点此去](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads)

- 解压：
> 使用tar -Jxf [文件名]来解压.tar.xz文件（会解压到新文件夹内）

### 2.配置交叉编译工具链脚本
- 新建一个.sh文件
- 在文件中输入以下内容：
  - Clang:
  > export PATH="[你的Clang编译器路径]/bin:[你的arm-linux-gnueabi套件路径]/bin:[你的aarch64-linux-gnu套件路径]/bin:${PATH}"

  - GCC
  > export PATH="[你的arm-linux-gnueabi套件路径]/bin:[你的aarch64-linux-gnu套件路径]/bin:PATH"
3. 保存
### 3.编写编译脚本

- 问题：
> 为啥要写编译脚本？

- 解答：
> 因为方便啊，make命令要加好多参数，好麻烦......
PS：不要说用环境变量，有些内核在makefile里面用`=`定义了变量，如果使用环境变量就不能覆写makefile内部的变量了。变量优先级：参数变量>makefile里定义的变量>环境变量

1. 项目清理脚本
  - 新建一个.sh文件
  - 在文件中输入以下内容：
```
Clang:
sh [配置交叉编译工具链时建立的脚本]
cd [你的项目]
make ARCH=[目标内核架构] CC=clang HOSTCC=clang CROSS_COMPILE=aarch64-linux-gnu- CROSS_COMPILE_ARM32=arm-linux-gnueabi- mrproper
GCC:
sh [配置交叉编译工具链时建立的脚本]
cd [你的项目]
make ARCH=[目标内核架构] CC=gcc HOSTCC=gcc CROSS_COMPILE=aarch64-linux-gnu- CROSS_COMPILE_ARM32=arm-linux-gnueabi- mrproper
```
  
2. 配置文件输出脚本
  - 问题：
  > 啥是配置文件？

  - 解答：
  > 一套源码可能被多个设备共有。在编译的时候，make通过不同的配置文件来区分这些设备，这些配置文件在`arch/[架构]/config`下，在make的时候无需指定路径，直接输入配置文件名字即可。

  - 新建一个.sh文件
  - 在文件中输入以下内容：
```
Clang:
sh [配置交叉编译工具链时建立的脚本]
cd [你的项目]
make O=[输出路径（在内核项目文件夹里面的相对路径）] ARCH=[目标内核架构（请与项目清理脚本中的目标内核架构保持一致）] CC=clang HOSTCC=clang CROSS_COMPILE=aarch64-linux-gnu- CROSS_COMPILE_ARM32=arm-linux-gnueabi- [你要编译的配置文件]
GCC:
sh [配置交叉编译工具链时建立的脚本]
cd [你的项目]
make O=[输出路径（在内核项目文件夹里面的相对路径）] ARCH=[目标内核架构（请与项目清理脚本中的目标内核架构保持一致）] CC=gcc HOSTCC=gcc CROSS_COMPILE=aarch64-linux-gnu- CROSS_COMPILE_ARM32=arm-linux-gnueabi- [你要编译的配置文件]
```

3. 内核编译脚本

  - 新建一个.sh文件
  - 在文件中输入以下内容：
```
Clang:
sh [配置交叉编译工具链时建立的脚本]
cd [你的项目]
make -j$(nproc --all) O=[输出路径（请与配置文件输出脚本中的输出路径保持一致）] ARCH=[目标内核架构（请与配置文件输出脚本中的目标内核架构保持一致）] CC=clang HOSTCC=clang CROSS_COMPILE=aarch64-linux-gnu- CROSS_COMPILE_ARM32=arm-linux-gnueabi-
GCC:
sh [配置交叉编译工具链时建立的脚本]
cd [你的项目]
make -j$(nproc --all) O=[输出路径（请与配置文件输出脚本中的输出路径保持一致）] ARCH=[目标内核架构（请与配置文件输出脚本中的目标内核架构保持一致）] CC=gcc HOSTCC=gcc CROSS_COMPILE=aarch64-linux-gnu- CROSS_COMPILE_ARM32=arm-linux-gnueabi-
```

### 3.开始编译
- 依次运行：项目清理脚本、配置文件输出脚本、内核编译脚本。完成一次完整编译
- （如果有运行过配置文件输出脚本且没有使用项目清理脚本）运行：内核编译脚本。完成一次脏编译

### 4.内核文件
- 内核文件在[你的项目/输出路径/arch/目标内核架构/boot]下，自己复制出来去做成刷机包就好了。
