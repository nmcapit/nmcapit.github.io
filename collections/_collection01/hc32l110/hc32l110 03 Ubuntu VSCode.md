---
---

# 目录

* [HC32L110(一) HC32L110芯片介绍和Win10下的烧录](https://www.cnblogs.com/milton/p/16586369.html)
* [HC32L110(二) HC32L110在Ubuntu下的烧录](https://www.cnblogs.com/milton/p/16586831.html)
* [HC32L110(三) HC32L110的GCC工具链和VSCode开发环境](https://www.cnblogs.com/milton/p/16634604.html)
* [HC32L110(四) HC32L110的startup启动文件和ld连接脚本](https://www.cnblogs.com/milton/p/16651892.html)
* [HC32L110(五) Ubuntu20.04 VSCode的Debug环境配置](https://www.cnblogs.com/milton/p/16653827.html)

以下介绍Ubuntu下搭建用于HC32L110系列MCU的GCC工具链和VSCode的开发环境.

仓库地址: [https://github.com/IOsetting/hc32l110-template](https://github.com/IOsetting/hc32l110-template)

如果转载, 请注明出处.

# 硬件准备

## JLink-OB

前一篇中已经介绍, 用于Linux环境下烧录

## 基于HC32L110系列MCU的开发板

以下是 AS06-VTB07H [产品页链接](http://www.ashining.com/product_view.aspx?t=149&cid=253). 这个开发板有新旧两个版本. 4.0使用的是STM8, 5.0使用的是HC32L110, 现在能买到的都是后者, pin脚全部引出, 有预留烧录口, 有一个功能按钮, 两个LED, 自带USB2TTL通信(P01, P02), 非常方便.

![](https://img2022.cnblogs.com/blog/650273/202208/650273-20220829010545144-1872223471.png)

下面的介绍都基于这个开发板. 如果使用其他的板子, GPIO口自己调整一下就可以.

# 软件准备

## 烧录软件 JLink

JLink软件和对应的flash算法文件, 在前一篇中已经介绍

## IDE VSCode

安装并配置好, 在网上有很多教程.

## GCC ARM工具链

在GCC ARM网站下载工具链接[https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads), 然后解压到合适的目录
```bash
tar xvf gcc-arm-11.2-2022.02-x86_64-arm-none-eabi.tar.xz
cd /opt/gcc-arm/
sudo mv ~/Backup/linux/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/ .
sudo chown -R root:root gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/
```
检查版本
```bash
/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gcc --version
arm-none-eabi-gcc (GNU Toolchain for the Arm Architecture 11.2-2022.02 (arm-11.14)) 11.2.1 20220111
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
````

# 示例项目导出和编译

## 导出项目
```bash
git clone https://github.com/IOsetting/hc32l110-template.git
```
根据自己的环境参数修改下Makefile
```
PROJECT         ?= app
# 改成本地的工具链路径
ARM_TOOCHAIN    ?= /opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin
# 改成本地的JLinkExe的路径
JLINKEXE        ?= /opt/SEGGER/JLink/JLinkExe
# MCU型号, 这个是16K的版本, 如果是32K版本要改成 HC32L110x6, 对应下面的ld文件也要修改
DEVICE          ?= HC32L110x4
# The path for generated files
BUILD_DIR       = Build

# Link descript file, hc32l110x4.ld or hc32l110x6.ld
LDSCRIPT        = Libraries/LDScripts/hc32l110x4.ld
# ...
```

## 编译默认项目

目录 User 下有默认的点灯示例代码, 在上一步修改完Makefile后, 就可以编译了
```bash
make clean
make
```
如果想看到详细的命令行, 可以用
```bash
V=1 make
```

## 烧录

编译完成后, 执行下面的命令烧录
```bash
make flash
```
如果使用的是 AS06-VTB07H, 烧录完成后可以看到两个LED每隔一秒交替闪烁.

# 配置VSCode开发环境

作为Linux下的生产力工具, VSCode是要用起来的, 开发C语言项目, 需要配置的就是两个, c_cpp_properties.json 和 tasks.json

首先用VSCode打开项目目录, C/C++插件未安装会有提示, 按提示安装即可.

## 配置C/CPP: c_cpp_properties.json

快捷键`Ctrl + Shift + P`调出菜单, 输入 C/C++, 可以看到 **C/C++ Edit Configurations (JSON)**, 回车, 在创建的模板中按以下内容编辑, 路径换成自己的
```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [],
            "compilerPath": "/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gcc",
            "cStandard": "gnu99",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "gcc-arm",
            "configurationProvider": "ms-vscode.makefile-tools"
        }
    ],
    "version": 4
}
```

## 配置Tasks: tasks.json

快捷键`Ctrl + Shift + P`调出菜单, 输入 Task, 可以看到 **Config Task**, 回车 -> Create tasks.json from template -> Others, 编辑创建的 tasks.json , 根据自己的习惯添加快捷方式
```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "clean, build",
            "type": "shell",
            "command": "make clean;make",
            "problemMatcher": []
        },
        {
            "label": "build, download",
            "type": "shell",
            "command": "make;make flash",
            "problemMatcher": []
        },
        {
            "label": "download",
            "type": "shell",
            "command": "make flash",
            "problemMatcher": []
        },
        {
            "label": "build",
            "type": "shell",
            "command": "make",
            "problemMatcher": []
        },
        {
            "label": "clean",
            "type": "shell",
            "command": "make clean",
            "problemMatcher": []
        },
    ]
}
```
平时使用时, 用`Alt + Shift + F10`就可以调出Task菜单, 快速执行编译和烧录等操作.

# 编译选项

## 项目中新增代码目录和单个C文件

User目录下的代码, Makefile已经覆盖, 会自动添加无需修改 Makefile. 如果新增其他的目录需要包含整个目录, 或包含单个源文件, 可以编辑Makefile中的这部分
```
# C source folders
CDIRS   := User \
        Libraries/CMSIS \
        Libraries/HC32L110_Driver/src
# C source files (if there are any single ones)
CFILES := 
```
其中 CDIRS 用于目录的添加, CFILES 用于单个C源文件的添加.

除此以外, 如果有新增的 include 路径, 需要添加到 INCLUDES 这个变量中
```
# Include paths
INCLUDES    := Libraries/CMSIS \
            Libraries/HC32L110_Driver/inc \
            User
```
如果编译与预期不符, 在make时增加`V=1`查看实际的命令行.

## 调整编译参数

编译的参数都在 rules.mk 文件中,
```
# Global compile flags
CFLAGS      = -Wall -ggdb -ffunction-sections -fdata-sections
ASFLAGS     = -g -Wa,--warn

# Arch and target specified flags
OPT         ?= -Os
CSTD        ?= -std=c99
ARCH_FLAGS  := -fno-common -mcpu=cortex-m0plus -mthumb

# c flags
TGT_CFLAGS  += $(ARCH_FLAGS) $(addprefix -D, $(LIB_FLAGS))
# asm flags
TGT_ASFLAGS += $(ARCH_FLAGS)
# ld flags
TGT_LDFLAGS += --specs=nano.specs -mcpu=cortex-m0plus -mthumb -nostartfiles -Wl,--gc-sections -Wl,-Map=$(BDIR)/$(PROJECT).map -Wl,--print-memory-usage
```
如果要优化编译的结果大小或执行速度, 需要修改OPT参数, `-O0`是无优化, `-O1`是基础优化, `-Os`是尺寸压缩, 具体优化项参考 [https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
```
OPT         ?= -Os
```
其中`-ffunction-sections -fdata-sections`和`-Wl,--gc-sections`是非常重要的参数, 如果不使用这些参数, 编译的结果大概会超出HC32L110的16K字节限制.