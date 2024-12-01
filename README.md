这个项目需要开发一个火车售票系统。
# 1. 调试

**实验环境准备：**

1. vmware 虚拟机
2. ubuntu 20.04 LTS
3. vim
4. gcc

**常用调试技巧**

1. 代码检查-试运行-出错法
   修改程序然后重新尝试运行，观察输出结果
2. 取样法
   在程序中增加一些语句以获得更多关于程序内部的运行情况信息
3. 受控执行法
   直接检查程序的执行情况
4. 内存调试法

## 1.1. 代码检查-试运行-出错法

**例 1：给定一个有漏洞的示例程序 `debug1.c`，对其进行调试。**

**实验步骤：**

1. 编译程序，查看终端编译信息

   ```sh
   gcc -o debug1 debug1.c
   ```
    观察编译过程，并执行`./debug1`

2. 在 `main` 函数中添加一些代码以打印输出结果
   ```c
   for(int i = 0; i < 5; i++) {
       printf("array[%d] = { %d, %s }\n", i, array[i].key, array[i].val);
    }
   ```
3. 重新编译程序，查看终端打印的输出信息
4. 显然数据的结果并不是正确的结果，**请对程序作适当修改,并重编译运行，使之输出正确结果。**
## 1.2. 取样法

在程序中增加一些语句以获得更多关于程序内部的运行情况

1. 向文件`debug2.c`中添加打印 `x` 输出信息代码块:

```c
#ifdef DEBUG
printf("variable x has value = %d\n:", x );
#endif
```

2. 编译运行：

```sh
gcc -o debug2 -DDEBUG debug2.c
./debug2
```

3. 试运行 `./debug2`，查看输出结果。

## 1.3. 受控执行法

**使用 `gdb` 进行调试，需要在编译时加入 `-g` 命令**

```sh
gcc  -g debug1.c -o debug1
gdb debug1
```

**查看变量，则需要提前打断点**

```sh
(gdb) b 行号
(gdb) b 30
```

**使用 `run` 命令开始运行程序**

```sh
(gdb) run
```

观察输出结果，如果发现错误，则 `gdb` 会报告失败的原因和位置，可以根据信息找到错误代码。

**使用 `c` 命令继续运行程序**

```sh
(gdb) c
```

**查看变量**

```sh
(gdb) print 变量名
```

**其他的 `gdb` 命令可以使用 `help` 命令**

```sh
(gdb) help
```

## 1.4. 内存调试

**使用 `Valgrind` 工具**

1. `sudo apt install valgrind`
2. `gcc -g egence.c -o egence`
3. 运行可执行文件egence,观察现象
4. 使用 `Valgrind`,对该程序进行分析: `valgrind --leak-check=full -s ./egence`
5. 修改egence.c,使之正确

# 2. 代码分析

试编译运行下面三个文件，并根据中断打印的信息，分析错误的原因。

修改相关错误,使之可以正确运行

```
memory1.c
memory2.c
memory3.c
```

# 3.makefile编写

**要求**:编写一个makefile文件,执行make all指令,可以完成对debug1.c,egence.c,memory2.c三个文件的编译,生成对应的三个可执行文件;执行make clean指令,完成对三个可执行文件的清理。(提供三个可执行文件的运行结果截图)

