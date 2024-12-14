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






# 1. 使用 qemu 模拟器搭建 ARM 运行环境

**环境：**

- Vmware 15.5.7
- Ubuntu20.04.3 LTS 64 位
- u-boot-2018.11.tar.bz2

**资源文件链接**

- linux 内核：https://www.kernel.org/
- uboot：https://ftp.denx.de/pub/u-boot/
- busybox：https://busybox.net/downloads/

**整体流程说明如下：**

1. 编译 ARM 架构 Linux 内核
2. 制作文件系统
3. 使用 qemu 启动系统

**注意事项：**

1. 实验中需要工具，如 git、 vim（系统自带 vim 是精简版，对键盘识别有问题）等，请自行下载安装
2. 注意执行完指令后终端给出的提示，请确认命令是否正确执行
3. 自学相关 linux 命令，确认输入命令是否正确，请使用 tab 键补全，不要直接复制粘贴

## 1.1. 编译 Linux 内核

1. 将实验使用的 Linux 内核文件拷贝到 qemuLinux 目录下，安装交叉编译链工具与qemu

   ```sh
   cd workdir/qemuLinux/
   sudo apt-get install gcc-arm-linux-gnueabi
   sudo apt-get install qemu
   ```

2. 解压内核文件
   ```sh
   tar xvf linux-4.9.291.tar.xz
   ```
3. 进入到解压后的目录文件，然后修改顶层目录下的 Makefile 文件
   ```sh
   cd linux-4.9.291/
   vim Makefile
   ```
   修改第 257，258 行的代码：
   ```sh
   ARCH            ?= arm
   CROSS_COMPILE   ?= arm-linux-gnueabi-
   ```
4. 查看一下内核配置文件，可以看到已经有了 vexpress_defconfig 配置文件
   ```sh
   ls arch/arm/configs/
   ```
5. 紧接着，我们就使用 make 命令来执行内核配置

   ```sh
   make CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm vexpress_defconfig
   ```

   生成了.config 配置文件，至此我们就将内核编译时需要的配置文件写入内核了。

6. 下面对内核进行详细的配置，输入 make menuconfig 命令并保存.config配置与.defconfig配置

   ```sh
   make CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm menuconfig
   make savedefconfig
   ```

   ```sh
   System Type -->
      [ ] Enable the L2x0 outer cache controller
      取消该选项，否则QEMU运行不起来
   Kernel Features -->
      [*] Use the ARM EABI to compile the kernel
      确保该选项被选择
   ```

   如果运行 menuconfig 显示缺失 ncurses 包，则运行以下命令安装即可

   ```sh
   sudo apt-get install libncurses5-dev
   ```

7. 接下来可以编译 Linux 内核了，输入以下命令
   ```sh
   sudo apt install u-boot-tools
   make CROSS_COMPILE=arm-linux-gnueabi- ARCH=arm
   make dtbs
   ```
   编译可能需要一点时间，耐心等待一下；
   编译成功后，arch/arm/boot 目录下生成内核镜像文件 zImage
   可以将 zImage 和 dtb 复制到单独的文件夹中方便使用
   ```sh
   cp arch/arm/boot/zImage ../
   cp arch/arm/boot/dts/vexpress-v2p-ca9.dtb ../
   ```
8. 进行到这里，我们可以运行以下命令进行测试，看编译出的内核是否能成功运行，以及 QEMU 对 vexpress-a9 单板支持是否够友好
   ```sh
   qemu-system-arm -M vexpress-a9 -m 512M -kernel zImage -dtb vexpress-v2p-ca9.dtb -nographic -append "console=ttyAMA0"
   ```
   如果看到内核启动过程的打印，说明目前的搭建是成功的
   启动画面：
   ![](./figures/2023-10-09-14-42-52.png)
   结束画面：
   ![image-20231009124659624](./figures/2023-10-09-14-43-14.png)

注：这里简单介绍一下 QEMU 命令的参数：

```sh
-M vexpress-a9 模拟vexpress-a9单板
-m 512M 单板运行物理内存512M
-kernel 指定内核镜像路径
-dtb 指定dtb文件路径
-nographic 不使用图形界面，只使用串口
-append “console=ttyAMA0” 内核启动参数，这里是告诉内核运行的串口设备是什么
```

## 制作根文件系统

在上面的测试中，我们发现内核会报 Kernel panic 错误，这是在提示我们缺少根文件系统。
Busybox 是一个集成了常用 Linux 命令和工具的软件。

1.  将实验使用的 busybox 拷贝到 qemuLinux 目录下，我们将使用 busybox 来制作根文件系统，并解压
    ```sh
    tar xvf busybox-1.33.0.tar.bz2
    ```
2.  对 busybox 进行如下操作

    ```sh
    cd busybox-1.33.0/
    make defconfig
    make CROSS_COMPILE=arm-linux-gnueabi-
    make install CROSS_COMPILE=arm-linux-gnueabi-
    ```

    完成之后，在 busybox 顶层目录的\_install 下面，就生成了 busybox 根文件系统。

3.  接下来就开始制作根文件系统
    创建根目录 rootfs，并将上一步骤生成的文件拷贝到此目录下
    **注意：rootfs 与 busybox 在同级目录 qemuLinux 下**

    ```sh
    cd ..
    mkdir rootfs
    sudo cp -r busybox-1.33.0/_install/* rootfs/
    ```

    从工具链中复制运行库到 lib 目录下

    ```sh
    sudo mkdir rootfs/lib
    sudo cp -P /usr/arm-linux-gnueabi/lib/* rootfs/lib
    ```

    创建 4 个 tty 终端设备（c 代表字符设备，4 是主设备号，1~4 分别是次设备号）

    ```sh
    sudo mkdir -p rootfs/dev
    sudo mknod rootfs/dev/tty1 c 4 1
    sudo mknod rootfs/dev/tty2 c 4 2
    sudo mknod rootfs/dev/tty3 c 4 3
    sudo mknod rootfs/dev/tty4 c 4 4
    ```

    生成镜像，并格式化成 ext3 文件系统

    ```sh
    dd if=/dev/zero of=a9rootfs.ext3 bs=1024M count=32
    mkfs.ext3 a9rootfs.ext3
    ```

    将文件复制到镜像 a9rootfs.ext3 中

    ```sh
    sudo mkdir tmpfs
    sudo mount -t ext3 a9rootfs.ext3 tmpfs/ -o loop
    sudo cp -r rootfs/* tmpfs/
    sudo umount tmpfs
    ```

## 使用 qemu，启动 linux 系统

下面我们就可以启动 qemu 来模拟 vexpress-a9 单板并在上面跑一个 Linux kernel 了

启动方式：

```sh
qemu-system-arm -M vexpress-a9 -m 512M -kernel zImage -dtb vexpress-v2p-ca9.dtb -nographic -append "root=/dev/mmcblk0 console=ttyAMA0 rw init=/linuxrc" -sd a9rootfs.ext3
```

![image-20231009141549712](./figures/2023-10-09-15-14-34.png)

   ![](./figures/2021-12-07-11-31-20.png)
