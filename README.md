# 1. 使用 qemu 模拟器搭建 ARM 运行环境

**环境：**

- Vmware 15.5.7
- Ubuntu20.04.3 LTS 64 位
- u-boot-2018.11.tar.bz2

**资源文件链接**

- linux 内核：https://www.kernel.org/
- uboot：https://ftp.denx.de/pub/u-boot/

**整体流程说明如下：**

1. 安装交叉编译工具链
2. 安装 qemu 模拟器
3. 编译 ARM 架构 u-boot
4. 用 u-boot 测试 qemu 能否正常启动

**注意事项：**

1. 实验中需要工具，如 git、 vim（系统自带 vim 是精简版，对键盘识别有问题）等，请自行下载安装
2. 注意执行完指令后终端给出的提示，请确认命令是否正确执行
3. 自学相关 linux 命令，确认输入命令是否正确，请使用 tab 键补全，不要直接复制粘贴

## 1.1. 安装交叉编译工具链，下载 qemu

1. 创建工作目录

   ```sh
   mkdir workdir
   cd workdir
   mkdir qemuLinux
   cd qemuLinux
   ```

2. 安装 ARM 交叉编译工具，以及编译 uboot，内核时用到的依赖

   ```sh
   sudo apt install gcc-arm-linux-gnueabi bison flex build-essential libncurses-dev libssl-dev
   ```

3. 安装 qemu 模拟器，并验证是否安装成功
   ```sh
   sudo apt install qemu qemu-system-arm
   qemu-system-arm --version
   ```
4. 查看 qemu 支持的 ARM 开发板型号
   ```sh
   qemu-system-arm -M help
   ```
   在列出的表单中，可以看出 qemu 支持 vexpress-a9 开发板。

## 1.2. 前期环境准备工作正确完成之后，接下来在正确启动 linux 内核之前，还需要编译制作 uboot

1. 将实验所需的文件 u-boot-2018.11.tar.bz2 拷贝到 qemuLinux 目录中，并解压

   ```sh
   tar xvf u-boot-2018.11.tar.bz2
   ```

2. 进入解压后的 uboot 文件夹 u-boot-2018.11。可以首先来查看一下 uboot 已经默认支持的有哪些 vexpress 开发板

   ```sh
   cd u-boot-2018.11/
   ls configs/vex*
   ```

   configs 目录中是各个厂商的内核配置文件，这个目录非常重要。
   从列出的文件中可以看出，uboot 默认是支持 vexpress_a9 开发板的，并且已经有了配置模板文件。

3. 执行 make 编译，编译成功会生成 .config 文件

   ```sh
   make vexpress_ca9x4_defconfig
   ```

4. 接下来，直接利用 uboot 默认支持的 vexpress_ca9x4_defconfig 文件，修改顶层的 Makefile 文件，配置交叉编译工具
   在 Makefile 文件中，第 249 行输入一下代码

   ```sh
   CROSS_COMPILE ?= arm-linux-gnueabi-
   ```

   在 config.mk 文件中，更改第 23 行的内容

   ```sh
   ARCH := arm
   ```

5. 完成以上步骤之后，保存退出。接下来执行 make -j4 编译，开启 4 个线程来进行编译 uboot

   ```sh
   make -j4
   ```

   编译成功之后，目前我们就得到了 uboot。

6. 紧接着，我们首先就来验证一下 uboot 是否制作成功了，在终端中，输入以下命令

   ```sh
   qemu-system-arm -M vexpress-a9   -kernel u-boot -nographic -m 128M
   ```

7. 如果启动的开始画面和结束画面如下图所示，则表示 uboot 制作成功。
   开始画面：
   ![](./figures/2021-12-07-11-30-31.png)

   结束画面：
   ![](./figures/2021-12-07-11-31-20.png)
