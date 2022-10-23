---
title: "AIR105 使用OpenOCD写入"
---


在Ubuntu下尝试用 openocd 写入, 没成功, 过程记录一下

## 硬件

* WCH-Link
* Air105开发板

## 步骤

安装 openocd
```bash
sudo apt install openocd
```

为兼容 WCH-Link 增加一个udev规则 99-platformio-udev.rules 
```bash
$ more /etc/udev/rules.d/99-platformio-udev.rules 
# WCH Link (CMSIS-DAP compatible adapter)
ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="8011", MODE="0666", ENV{ID_MM_DEVICE
_IGNORE}="1", ENV{ID_MM_PORT_IGNORE}="1"
```
保存后执行
```
sudo udevadm control --reload-rules
```
然后重新拔插WCHLink让权限生效


准备一个 openocd 配置文件 ocd-stm32.cfg , 内容如下
```
interface cmsis-dap
cmsis_dap_vid_pid 0x1a86 0x8011

transport select swd
#set CPUTAPID 0x2ba01477
source [find target/stm32f4x.cfg]
```
使用WCH-Link其中的`cmsis_dap_vid_pid 0x1a86 0x8011`是必须的, 否则会报`Error: unable to find CMSIS-DAP device`错误

使用 openocd 连接, 此时可以正确识别
```bash
$ openocd -f ./ocd-stm32.cfg
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
adapter speed: 2000 kHz
adapter_nsrst_delay: 100
none separate
cortex_m reset_config sysresetreq
Info : CMSIS-DAP: SWD  Supported
Info : CMSIS-DAP: Interface Initialised (SWD)
Info : CMSIS-DAP: FW Version = 2.0.0
Info : SWCLK/TCK = 1 SWDIO/TMS = 1 TDI = 0 TDO = 0 nTRST = 0 nRESET = 1
Info : CMSIS-DAP: Interface ready
Info : clock speed 2000 kHz
Info : SWD DPIDR 0x2ba01477
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
```

但是写入似乎不能正确识别芯片的ID
```bash
$ openocd -f ./ocd-stm32.cfg -c "reset_config none" -c "program air105_blink.hex verify reset exit" -d2
Open On-Chip Debugger 0.10.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
debug_level: 2
adapter speed: 1000 kHz
adapter_nsrst_delay: 100
none separate
cortex_m reset_config sysresetreq
none separate
Info : CMSIS-DAP: SWD  Supported
Info : CMSIS-DAP: Interface Initialised (SWD)
Info : CMSIS-DAP: FW Version = 2.0.0
Info : SWCLK/TCK = 1 SWDIO/TMS = 1 TDI = 0 TDO = 0 nTRST = 0 nRESET = 1
Info : CMSIS-DAP: Interface ready
Info : clock speed 1000 kHz
Info : SWD DPIDR 0x2ba01477
Info : stm32f1x.cpu: hardware has 6 breakpoints, 4 watchpoints
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x00000328 msp: 0x20010d20
** Programming Started **
auto erase enabled
Info : device id = 0x00000000
Warn : Cannot identify target as a STM32 family.
Error: auto_probe failed
** Programming Failed **
shutdown command invoked
```

openocd没有对应这个cpu的flash算法, 需要单独写或等人提供了




# 参考

* OpenOCD刷写FLASH代码结构浅析(基于RISCV)[https://zhuanlan.zhihu.com/p/507467621](https://zhuanlan.zhihu.com/p/507467621)
* 为国产芯片增加OpenOCD Flash驱动————以AIC8800为例[https://www.bilibili.com/read/cv15348789/](https://www.bilibili.com/read/cv15348789/)