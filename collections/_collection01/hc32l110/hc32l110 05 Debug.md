---
---

# 目录

* [HC32L110(一) HC32L110芯片介绍和Win10下的烧录](https://www.cnblogs.com/milton/p/16586369.html)
* [HC32L110(二) HC32L110在Ubuntu下的烧录](https://www.cnblogs.com/milton/p/16586831.html)
* [HC32L110(三) HC32L110的GCC工具链和VSCode开发环境](https://www.cnblogs.com/milton/p/16634604.html)
* [HC32L110(四) HC32L110的startup启动文件和ld连接脚本](https://www.cnblogs.com/milton/p/16651892.html)
* [HC32L110(五) Ubuntu20.04 VSCode的Debug环境配置](https://www.cnblogs.com/milton/p/16653827.html)

本文介绍在Ubuntu20.04下, VSCode中如何设置对 HC32L110 进行 debug

仓库地址: [https://github.com/IOsetting/hc32l110-template](https://github.com/IOsetting/hc32l110-template)

如果转载, 请注明出处.

# 环境说明

本文使用的软硬件环境已经在前面介绍

## 硬件

* 基于 HC32L110 系列MCU的开发板
* JLink OB

## 软件

* Ubuntu20.04
* VSCode

# 配置步骤

## 安装配置 Cortex-Debug

在VSCode的插件中, 搜索安装Cortex-Debug

在VSCode中, 切换到Run And Debug, 点击上方的 Add Configuration, 会在 .vscode 目录下的 launch.json (如果没有会自动创建)中添加配置, 需要增加对应的配置信息
```json
"configurations": [
        {
            "name": "Cortex Debug",
            "cwd": "${workspaceFolder}",
            "executable": "${workspaceFolder}/Build/app.elf",
            "request": "launch",        // 可以是launch或attach, 后者表示运行中接入, 前者会执行前置任务并重启
            "type": "cortex-debug",
            "runToEntryPoint": "main",
            "servertype": "jlink",
            "device": "HC32L110X4",     // 如果是32K的版本, 需要修改为 HC32L110X6
            "interface": "swd",
            "runToMain": true,          // false则从 reset handler 开始停留
            "preLaunchTask": "build",   // 根据 tasks.json 中配置的任务填写, 
            // "preLaunchCommands": ["Build all"], // 如果不使用 preLaunchTask, 可以用这个参数指定命令行命令
            "svdFile": "",              // svd 用于观察外设
            "showDevDebugOutput": "vscode", // 输出的日志级别, parsed:解析后的内容, raw:直接输出, vscode:包含scode调用和raw
            "swoConfig":
            {
                "enabled": true,
                "cpuFrequency": 24000000,
                "swoFrequency":  4000000,
                "source": "probe",
                "decoders":
                [
                    {
                        "label": "ITM port 0 output",
                        "type": "console",
                        "port": 0,
                        "showOnStartup": true,
                        "encoding": "ascii"
                    }
                ]
            }
        }
    ]
```
具体的配置项含义, 可以参考 [Debug Attributes](https://github.com/Marus/cortex-debug/blob/master/debug_attributes.md)

同时在 .vscode/settings.json 中增加以下配置, 如果文件不存在则创建. 路径根据自己的环境修改
```json
{
    "cortex-debug.gdbPath": "/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb",
    "cortex-debug.JLinkGDBServerPath": "/opt/SEGGER/JLink/JLinkGDBServerCLExe",
}
```

## 修改 rules.mk

在rules.mk中开启debug, 涉及到两处, `OPT`要使用`-O0`, 表示不执行任何优化. 使用优化后代码中的部分变量在编译后会被丢弃无法跟踪
```makefile
OPT         ?= -O0
```
在`CFLAGS`中增加gdb输出
```makefile
CFLAGS      += -g -gdwarf-2 # original: -g
```
这样编译后, 尺寸会比原先大不少


## Ubuntu20.04下遇到的问题

以上配置后, 点击 Run And Debug 中的绿色运行图标应该就能启动Debug, 但是可能在看到以下输出后就没有反应了
```
Reading symbols from /opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-objdump --syms -C -h -w /home/milton/Stm32Projects/hc32_workspace/hc32l110-template/Build/app.elf
Reading symbols from /opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-nm --defined-only -S -l -C -p /home/milton/Stm32Projects/hc32_workspace/hc32l110-template/Build/app.elf
Launching GDB: /opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb -q --interpreter=mi2
1-gdb-version
```
这时候到命令行执行一下命令`/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb -q --interpreter=mi2`就能看到问题

### 1. libncursesw.so.5: cannot open shared object file

首先是会有这样的错误输出
```bash
/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb -q --interpreter=mi2
/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb: error while loading shared libraries: libncursesw.so.5: cannot open shared object file: No such file or directory
```
观察ldd可以看到缺少两个so: libncursesw.so.5, libpython3.6m.so.1.0
```bash
ldd /opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb
    linux-vdso.so.1 (0x00007ffc82998000)
    libncursesw.so.5 => not found
    libtinfo.so.5 => /lib/x86_64-linux-gnu/libtinfo.so.5 (0x00007f326ba1e000)
    libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f326ba18000)
    libpython3.6m.so.1.0 => not found
    libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f326b9f5000)
    libutil.so.1 => /lib/x86_64-linux-gnu/libutil.so.1 (0x00007f326b9f0000)
    libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f326b89f000)
    libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007f326b871000)
    libstdc++.so.6 => /lib/x86_64-linux-gnu/libstdc++.so.6 (0x00007f326b68f000)
    libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f326b674000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f326b482000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f326ba6e000)
````

libncursesw.so.5可以通过安装 libncursesw5 可以解决
```bash
sudo apt install libncursesw5
```

### 2. libpython3.6m.so.1.0: cannot open shared object file

这样就剩下另一个动态连接库无法找到
```bash
/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb -q --interpreter=mi2
/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb: error while loading shared libraries: libpython3.6m.so.1.0: cannot open shared object file: No such file or directory
```
但是 Ubuntu20.04 自带的 Python3.8, 无法安装 Python3.6
```bash
~$ sudo apt-get install libpython3.6
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Note, selecting 'libpython3.6-stdlib' for regex 'libpython3.6'
0 upgraded, 0 newly installed, 0 to remove and 29 not upgraded.
```
简单粗暴的解决方法就是直接用3.8的代替
```bash
cd /usr/lib/x86_64-linux-gnu/
$ ll libpython*
lrwxrwxrwx 1 root root      19 Jul  1 20:27 libpython2.7.so.1 -> libpython2.7.so.1.0
-rw-r--r-- 1 root root 3455600 Jul  1 20:27 libpython2.7.so.1.0
lrwxrwxrwx 1 root root      55 Jun 23 04:18 libpython3.8.a -> ../python3.8/config-3.8-x86_64-linux-gnu/libpython3.8.a
lrwxrwxrwx 1 root root      17 Jun 23 04:18 libpython3.8.so -> libpython3.8.so.1
lrwxrwxrwx 1 root root      19 Jun 23 04:18 libpython3.8.so.1 -> libpython3.8.so.1.0
-rw-r--r-- 1 root root 5449112 Jun 23 04:18 libpython3.8.so.1.0
$ sudo ln -s libpython3.8.so.1.0 libpython3.6m.so.1.0
```
这时候再运行就可以启动了
```bash
$ /opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb -q --interpreter=mi2
/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb: Symbol `PyBool_Type' has different size in shared object, consider re-linking
/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb: Symbol `PySlice_Type' has different size in shared object, consider re-linking
/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb: Symbol `PyFloat_Type' has different size in shared object, consider re-linking
=thread-group-added,id="i1"
(gdb)
```

## Ubuntu22.04下的问题

Ubuntu22.04使用的python3版本是python3.10, 使用上面的方法直接链接为3.6m已经无法使用. 需要使用新的arm gcc版本.
从 https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads 下载 [arm-gnu-toolchain-11.3.rel1-x86_64-arm-none-eabi.tar.xz](https://developer.arm.com/-/media/Files/downloads/gnu/11.3.rel1/binrel/arm-gnu-toolchain-11.3.rel1-x86_64-arm-none-eabi.tar.xz). 这个版本使用的是 python3.8.
解压后查看gdb版本, 会提示这样的错误
```bash
/opt/gcc-arm/arm-gnu-toolchain-11.3.rel1-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb --version

Could not find platform independent libraries <prefix>
Could not find platform dependent libraries <exec_prefix>
Consider setting $PYTHONHOME to <prefix>[:<exec_prefix>]
Python path configuration:
  PYTHONHOME = (not set)
  PYTHONPATH = (not set)
  program name = '/usr/local/bld-tools/bld-tools-virtual-env/bin/python'
  isolated = 0
  environment = 1
  user site = 1
  import site = 1
  sys._base_executable = '/usr/local/bld-tools/bld-tools-virtual-env/bin/python'
  sys.base_prefix = '/usr'
  sys.base_exec_prefix = '/usr'
  sys.executable = '/usr/local/bld-tools/bld-tools-virtual-env/bin/python'
  sys.prefix = '/usr'
  sys.exec_prefix = '/usr'
  sys.path = [
    '/usr/lib/python38.zip',
    '/usr/lib/python3.8',
    '/usr/lib/lib-dynload',
  ]
Fatal Python error: init_fs_encoding: failed to get the Python codec of the filesystem encoding
Python runtime state: core initialized
ModuleNotFoundError: No module named 'encodings'

Current thread 0x00007faa8ac87c00 (most recent call first):
<no Python frame>
```

可以通过ppa直接安装python3.8
```bash
# 添加仓库
sudo add-apt-repository ppa:deadsnakes/ppa
# 安装, 不会替代系统的python3.10
sudo apt install python3.8
# 检查版本
python3.8 --version
```
再次查看gdb version就不再提示错误信息
```bash
/opt/gcc-arm/arm-gnu-toolchain-11.3.rel1-x86_64-arm-none-eabi/bin/arm-none-eabi-gdb --version

GNU gdb (Arm GNU Toolchain 11.3.Rel1) 12.1.90.20220802-git
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```



# 使用

点击 Run And Debug 中的绿色运行图标启动Debug, F10 下一步, F11 进入, Shift + F11 跳出. 左侧能观察到变量值和寄存器值. 操作方法和其它IDE基本一致.


# 参考

* Cortex-Debug Wiki [https://github.com/Marus/cortex-debug/wiki](https://github.com/Marus/cortex-debug/wiki)
* Matej Blagšič: Debugging STM32 in VSCODE with stlink and jlink [https://www.youtube.com/watch?v=g2Kf6RbdrIs](https://www.youtube.com/watch?v=g2Kf6RbdrIs)
  * GitHub Project [https://github.com/prtzl/Embedded_videos/tree/master/045_BUILDING-PT4/test-f407vg](https://github.com/prtzl/Embedded_videos/tree/master/045_BUILDING-PT4/test-f407vg)
* Cortex-Debug 参数说明 [https://github.com/Marus/cortex-debug/blob/master/debug_attributes.md](https://github.com/Marus/cortex-debug/blob/master/debug_attributes.md)

* [https://stackoverflow.com/questions/64491112/libncursesw-so-5-is-installed-but-a-program-that-needs-it-says-no-such-file-or](https://stackoverflow.com/questions/64491112/libncursesw-so-5-is-installed-but-a-program-that-needs-it-says-no-such-file-or)
* libpython3.6m.so.1.0: cannot open shared object file: No such file or directory [https://github.com/JacquesLucke/animation_nodes/issues/906](https://github.com/JacquesLucke/animation_nodes/issues/906)
* How to install libpython3.6m.so.1.0 on ubuntu 20.04 [https://askubuntu.com/questions/1404248/how-to-install-libpython3-6m-so-1-0-on-ubuntu-20-04](https://askubuntu.com/questions/1404248/how-to-install-libpython3-6m-so-1-0-on-ubuntu-20-04)
