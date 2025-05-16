---
title: MakeFile基本语法学习
date: 2022-12-23 21:04:56
category: 学习笔记
tags:
  - Android
  - 编程
---

工作中经常和 Makefile 打交道，所以花了一些时间快速学习了一下它的常见用法以及使用场景。

## 基础语法

Makefile是由一系列的单一规则指令组合起来：

```makefile
目标XX1:依赖文件
<TAB>命令1
<TAB>命令2
 
目标XX2:依赖文件
<TAB>命令1
<TAB>命令2
 
... ...
 
指令1：
    命令1
    命令2
指令2：
    命令1
    命令2
... ...
例子：
Tag：a.o b.o c.o
    gcc -o Tag a.o b.o c.o
a.o: a.c
    gcc -c a.c
b.o: b.c
    gcc -c b.c
c.o: c.c
 
clean:
    rm .o
    rm Tag
```

## 变量

被 `=` 赋值的变量，其值取决于最后一次赋值。

指令 `print` 中 `echo` 前加上 `@` 省略回显。

```makefile
buff = xiaoming
name = $(xiaoming)
buff = lisa
print:
 echo name:$(name)
```

被 `:=` 赋值的变量，值保持不变；

被 `?=` 赋值的变量，如果之前已经有值则保持不变，如果没有则赋当前值；

被 `+=` 赋值的变量，在后面追加新的值；

## 模式规则

```makefile
a.o : a.c
    gcc -c a.c
b.o : b.c
    gcc -c b.c
# 运行模式规则“%”：当目标中重现“%”时，目标中“%”所代表的值决定了依赖文件中的“%”的值
%.o : %.c
    gcc -c $<
```

## 伪目标

伪目标主要是为了避免 `Makefile` 中定义的执行指令和工作目录下的实际文件出现名字冲突。

举例说明：当前目录下如果有一个名为 `clean` 的文件，执行 `make clean` 指令，

因为没有依赖文件，所以后续的 `rm` 指令不会被执行。

解决方法为在 `Makefile` 中将指令声明为伪目标即可 `.PHONY`

```makefile
.PHONY

clean：
    rm *.o
```

## 函数

1、函数`subst`：完成字符串替换；

```makefile
$(subst <from>, <to>, <text>)
 
$(subst aaa, AAA, 3a transform 3A aaa)
```

将字符串 `3a transform 3A aaa` 中的 `aaa` 替换为 `AAA` 即：`3a transform 3A AAA`

2、函数 `patsubst`：完成模式字符串替换；

```makefile
$(patsubst <pattern>, <replacement>, <text>)
 
$(patsubst %.c, %.o, a.c b.c c.c)
```

将字符串 `a.c b.c c.c` 替换为 `a.o b.o c.o`

如果 `text = a.c b.c c.c`

那么，`$(text: .c = .o)` 等同于 `$(patsubst %.c, %.o, $(text))`

3、函数 `dir` ：获取路径当中的目录；

```makefile
$(dir <name...>)
 
$(dir </src/a.c>)
```

提取文件`/src/a.c`的目录部分`/src`

4、函数`notdir`：提取文件名；

```makefile
$(notdir <name...>)
 
$(notdir <src/a.c>)
```

提取文件 `/src/a.c` 的非目录部分 `a.c`

5、函数 `foreach` ：循环；

6、函数 `wildcard` ：`wildcard` 用来明确表示通配符；

因为在 `Makefile` 里，变量实质上就是 C/C++ 中的宏，

也就是说，如果一个表达式如 `objs = *.o` ，则 `objs` 的值就是 `*.o` ，而不是表示所有的`.o` 文件。

若果要使用通配符，那么就要使用 `wildcard` 来声明 `*` 这个符号，使 `*`  符号具有通配符的功能。

```makefile
$(foreach <var>, <list>, <text>)
 
SRCDIRS := dira dirb dirc 
$(foreach dir, $(SRCDIRS), $(wildcard $(dir) / *.c))
```

循环将 `SRCDIRS` 中的各个目录放进 `dir` 变量中，调用 `wildcard` 函数提取 `dir` 目录下所有 `.c` 文件

## 自动化变量

| 自动化变量 | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| `$@`       | 规则中的目标集合，在模式规则中，如果有多个目标的话，`$@`表示匹配模式中定义的目标集合 |
| `$%`       | 当目标是函数库的时候表示规则中的目标成员名，如目标不是函数库文件，那么其值为空 |
| `$<`       | 依赖文件集合中的第一个文件，如果依赖文件是以模式`%`定义的，那么`$<`就是符合模式的一系列的文件的集合 |
| `$?`       | 所有比目标新的依赖目标的集合，以空格分开                     |
| `$^`       | 所有依赖文件的集合，使用空格分割，如果在依赖文件中有多个重复的文件，`%^`会去除重复的依赖文件，值保留一份 |
| `$+`       | 和`$^`类似，但是当依赖文件存在重复的话，不会去除重复的依赖文件 |
| `$*`       | 这个变量表示目标模式中`%`以及其之前的部分，如果目标是`test/a.test.c`目标模式为`a.%.c`，那么`$*`就是`test/a.test` |

## 模板示例

1、原始模板

```makefile
main.bin:a.o b.o c.o
 arm-linux-gnueabihf-ld  -Txxx.lds -o main.elf a.o b.o c.o
 arm-linux-gnueabihf-objcopy -o binary -s -g main.elf main.bin
 arm-linux-gnueabihf-objdump -D main.elf > main.dis
 
a.o : a.c
    arm-linux-gnueabihf-gcc -c a.c -o a.o
b.o : b.c
    arm-linux-gnueabihf-gcc -c b.c -o b.o
c.o : c.s
    arm-linux-gnueabihf-gcc -c c.s -o c.o
 
clean:
 rm -rf *.o main.bin main.elf main.dis
```

2、替换为自动变量和规则模式

```xml
objs := a.o b.o c.o
 
main.bin:$(objs)
 arm-linux-gnueabihf-ld  -Txxx.lds  -o main.elf $^               /*(1)*/
 arm-linux-gnueabihf-objcopy -o binary -s -g main.elf $@         /*(2)*/
 arm-linux-gnueabihf-objdump -D main.elf > main.dis              
%.o : %.c
    arm-linux-gnueabihf-gcc -c $< -o $@                             /*(3)*/
%.o : %.s
    arm-linux-gnueabihf-gcc -c $< -o $@ 
clean:
 rm -rf *.o main.bin main.elf main.dis
```

（1）`$^：a.o b.o c.o`

（2）`$@：main.bin`

（3）`$<：%.c ； $@：%.o`

3、替换为变量

```makefile
CROSS_COMPILE ?= arm-linux-gnueabihf-
NAME          ?= main
 
CC            := $(CROSS_COMPILE)gcc
LD            := $(CROSS_COMPILE)ld
OBJCOPY       := $(CROSS_COMPILE)objcopy
OBJDUMP       := $(CROSS_COMPILE)objdump
 
OBJS := a.o b.o c.o
 
$(NAME).bin = $(OBJS)
    $(LD) -Txxx.lds -o $(NAME).elf $^
    $(OBJCOPY) -o binary -s -g $(NAME).elf $@
    $(OBJDUMP) -D $(NAME).elf > $(NAME).dis
 
%.o : %.c
    $(CC) -c $< -o $@
%.o : %.s
    $(CC) -c $< -o $@
 
clean:
    rm -rf *.o $(NAME).bin $(NAME).elf $(NAME).dis
```

4、多文件工程

```makefile
CROSS_COMPILE ?= arm-linux-gnueabihf-
TARGET        ?= main
 
CC            := $(CROSS_COMPILE)gcc
LD            := $(CROSS_COMPILE)ld
OBJCOPY       := $(CROSS_COMPILE)objcopy
OBJDUMP       := $(CROSS_COMPILE)objdump
 
INCDIRS       := dira \
                 dirb \
                 dirc \
 
SRCDIRS       := dira dirb dirc   
 
INCLUDE       := $(patsubst %, -I %, $(INCDIRS))                              /*（1）*/
 
SFILES        := $(foreach dir, $(SRCDIRS), $(wildcard $(dir) / *.s))         
CFILES        := $(foreach dir, $(SRCDIRS), $(wildcard $(dir) / *.c))         /*（2）*/
 
SFILENDIR     := $(notdir $(SFILES))                                         
CFILENDIR     := $(notdir $(CFILES))                                          /*（3）*/
 
SOBJS         := $(patsubst %, obj/%, $(SFILENDIR:.s=.o))                     
COBJS         := $(patsubst %, obj/%, $(CFILENDIR:.c=.o))                     /*（4）*/
OBJS          := $(SOBJS) $(COBJS)                                            /*（5）*/
 
VPATH         := $(SRCDIRS)                                                   /*（6）*/
 
.PHONY: clean
 
$(TARGET).bin : $(OBJS)
    $(LD) -Txxx.lds -o $(TARGET).elf $^
    $(OBJCOPY) -o binary -s %(TARGET) $@
    $(OBJDUMP) -D -m arm $(TARGET).elf > $(TARGET).dis
 
$(SOBJS) : obj/%.o ： %.s
    $(CC) -Wall -nostdlib -c -o2 $(INCLUDE) -o $@ $<
$(COBJS) : obj/%.o ： %.c
    $(CC) -Wall -nostdlib -c -o2 $(INCLUDE) -o $@ $<   
 
clean:
    rm -rf $(TARGET).elf $(TARGET).dis $(TARGET).bin $(COBJS) $(SOBJS)
```

（1）：`INCLUDE := -I dira -I dirb -I dirc`

将字符串目录前加 `-I` ，`Makefile` 语法要求头文件目录需加 `-I`

（2）：`CFILES := dira/a.c dirb/b.c`

将 `SRCDIRS` 各个目录下的 `c` 文件提取出来

（3）：`CFILENDIR := a.c b.c`

提取 `CFILES` 中的文件名，省略路径

（4）：`COBJS := obj/a.o obj/b.o`

将原目录下各个 `c` 文件 `s` 文件编译为 `.o` 文件，并将其放置 `obj` 目录下。

（5）：`OBJS = obj/a.o obj/b.o obj/c.o`

整合 `SOBJS` 和 `COBJS` 。

（6）：指定编译时查询目录
