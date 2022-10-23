---
---


# 目录

* [合宙AIR105(一): Keil MDK开发环境, DAP-Link 烧录和调试](https://www.cnblogs.com/milton/p/15858286.html)
* [合宙AIR105(二): 时钟设置和延迟函数](https://www.cnblogs.com/milton/p/16387525.html)

# 关于AIR105

AIR105是合宙LuatOS生态下的一款芯片, QFN88封装. 2022年1月初上市, 开发板与摄像头一起搭售. 与兆讯的 MH1903S 是同一款芯片, MH1903 系列还有 BGA169 等高密度封装

主要配置参数: Cortex-M4F内核, 最高频率204Mhz, 片上内建640KB SRAM和4MB Flash. 尺寸10x10mm, 56个可编程GPIO PIN.

![](https://img2022.cnblogs.com/blog/650273/202201/650273-20220131150110870-682995159.png)

<br>

## 功能特性

* ARM SecurCore™ SC300™核心
  * 32-bit RISC Core(ARMv7-M)
  * MPU 内存保护单元
  * 最高 204MHz 主频(1、2 分频可调)
  * FPU 单元
  * 1 个受控 JTAG-DP/SW-DP 调试端口
* 640KB SRAM 
* 1 个 QSPI 控制器, 支持 XIP
* 系统控制模块(控制所有外设模块时钟及系统相关配置)
* 安全加密算法加速引擎
  * 对称算法: DES、TDES、AES-128/192/256 
  * 非对称算法: RSA-1024/2048、ECC
  * HASH 校验算法: SHA-1/224/256/384/512 
* 2 路 SmartCard 接口(支持 EMV Level-1 协议规范、ISO7816-3 标准), 其中 SCI0 集成 7816 电平转换功能, 可配置输出 3V 和 1.8V 
* 4 路 UART 接口(均支持 4 线)
* 3 路 SPI 接口(其中 SPI0 主从可配, 其他 2 路仅支持 Master) * 1 路高速 SPI Master 接口 SPI5
* 1 路 IIC 接口
* 1 路 KBD(4x5 矩阵键盘)
* 8 个 32 位 TIMER(带有 PWM 功能, 支持单周期输出) 
* 1 路 LCDI 接口, 支持 8080、6800 总线协议
* 1 个真随机数发生器
* 1 个 DMA 控制器(支持 8 通道 DMA 传输)
* 1 个 CRC 模块(支持 16Bit/32Bit、多种常用多项式计算)
* 最多支持 87 个 GPIO
* 最多支持 8 个静态 Tamper 或 4 组动态 Tamper(4 输出, 4 输入), 动/静态可配
* 1 组内部 Sensor(支持高低电压、高低温、Mesh、时钟和 voltage glitch 检测)
* 1 块密钥存储区(支持硬件快速擦除)
* 1 个 USB(OTG-FS)
  * 支持 USB2.0 和 OTG1.0a 
  * 内置 USB PHY 模块
  * 专用 DMA 通道和专有的中断向量, 加快数据通信速度
* 集成内部看门狗
* 1 个 10bit DAC 接口
* 1 个 7 通道 12bit ADC, 最高支持 857KHz 采样率(0 通道固定采集 CHARGE_VBAT 电压, 其余通道采集电压范围 0~1.8V 或 0~3.6V 可配)
* 支持磁条解码功能, 支持 ISO/ABA、AAMVA、IBM 和 JIS II 等标准卡
* 1 个 DCMI 接口
* 芯片集成一个可对外输出 150mA 驱动能力的 LDO 
* 芯片集成 USB 充电管理模块, 支持最大 200mA 的充电电流
* 芯片集成开关机功能
* 支持 27.12M 输出

# 资料下载

* 产品资料页: [https://doc.openluat.com/article/4369](https://doc.openluat.com/article/4369)
* AIR150资料: [https://wiki.luatos.com/chips/air105/index.html](https://wiki.luatos.com/chips/air105/index.html)
  * 芯片资料: [https://wiki.luatos.com/chips/air105/mcu.html](https://wiki.luatos.com/chips/air105/mcu.html)
  * 开发板使用说明: [https://wiki.luatos.com/chips/air105/board.html](https://wiki.luatos.com/chips/air105/board.html)
  * 硬件资料: [https://wiki.luatos.com/chips/air105/hardware.html](https://wiki.luatos.com/chips/air105/hardware.html)
  * 寄存器手册: [Air105芯片数据手册_1.1.pdf](https://cdn.openluat-luatcommunity.openluat.com/attachment/20220114193313925_Air105%E8%8A%AF%E7%89%87%E6%95%B0%E6%8D%AE%E6%89%8B%E5%86%8C_1.1.pdf)
* 烧录说明: [https://wiki.luatos.com/boardGuide/flash.html](https://wiki.luatos.com/boardGuide/flash.html)

# 使用Keil MDK + DAPLINK 烧录和 Debug

合宙提供了这个芯片及开发板的 LuatOS 开发环境和工具链, 但还是有很多开发用户希望能使用纯C语言的环境和工具链. 在合宙用户群群友(736314515@qq, 309670104@qq, 感谢)的帮助下, 把烧录和调试部分跑通了, 以下说明如何在Keil5 MDK中使用 DAP-Link 烧录 AIR105 开发板.

## 准备工作

### 板卡

* AIR105开发板
  需要引出PC3口, 后面会说明
* DPALink转接卡
  可以使用标准的DAP-Link, 也可以使用WCH-Link卡(需要切换到DAP-Link状态), 这两种卡都测试过, 体现在软件中是一样的, 都可以使用

注意: 如果使用WCH-Link, 一定要看清电压, 别误接板上的5V输出.

### 工具部分

* OK线, 用于引出PC3, 如果不知道OK线长什么样的, 可以在某宝搜索. 如果有直径0.2-0.3mm的漆包线也可以使用
* 特尖烙铁头, 搜索型号 900-SI, 900M-T-IS, 900M-T-I 这类特尖头和特尖弯头, 因为PC3对应的电阻为0402, 与开关贴得很近, 普通烙铁头不容易焊接
* 普通焊锡丝
* UV559(或者类似的, 非酸性)助焊油

### 软件

* 预先安装好 Keil5 MDK, 以下假设使用的是默认的安装目录 C:\Keil_v5
* 下载好示例项目并在Keil5 MDK中打开, [下载地址](https://download.csdn.net/download/michaelchain/78709171)
* 下载FLM文件备用, [下载地址](https://gitee.com/openLuat/luatos-soc-air105/blob/master/Air105.FLM)

<br>

## 硬件调整: 接出PC3

使用DAP-Link连接需要4根线: VCC, GND, SWCLK 和 SWDIO, 其中 SWCLK对的是PC4, 这个pin脚是现成的; SWDIO对的是PC3, 这个开发板并未接出PC3, 所以需要自行焊接引出. 

根据开发板线路图, 靠近开关一侧的LED通过一个5K的电阻连接了PC3, 所以只需要从这个电阻的一侧引出接线即可. 位置可以参考下图的红框部分. 

可以使用图中这种固定方式, 焊排针时多留一个排针, 将线一端先固定好, 另一端让线尖正好搭在电阻的一侧. 在线尖处加上助焊剂后, 烙铁尖头搭一点锡后将线焊上. 注意锡不能多, 时间不能太久. 焊完要用万用表检查一下, 量排针到电阻的另一侧, 阻值是否正确.

![](https://img2022.cnblogs.com/blog/650273/202201/650273-20220131150508005-1726551463.jpg)

<br>

## 修改PC3口输出状态

如果开发板还处于出厂状态, PC3用于点亮LED, 会处于输出状态, 所以此时开发板通过DAP-Link连上电脑后, Keil中查看DAP-Link设备会显示 DAP Error, 需要先刷入其他固件, 将PC3口的输出状态修改掉. 

烧录使用 Luatools, 参考[烧录说明](https://wiki.luatos.com/boardGuide/flash.html). 可以自己制作(看最后的说明), 或者使用这个已经制作好的soc [点击下载](https://download.csdn.net/download/michaelchain/78712105)

烧录成功后, PC3口不会再用于输出, 串口会输出大量杂乱内容, 可以忽略.

## 接线

3.3V和GND在Air105开发板上有好几组, 选一组连即可
```
# DAP-Link -> Air105 Dev-Board

3.3V       -> 3.3V 
GND        -> GND
SCK        -> PC4 版上标识为 ADC5
SWD        -> PC3 根据上面的说明额外接出
```

## 配置Keil5 MDK

开发板通过DAP-Link连上电脑后, 在Keil中查看DAP-Link就正常了, 识别 DAP-Link 和 Air105 不需要安装额外的软件

![](https://img2022.cnblogs.com/blog/650273/202201/650273-20220131150938340-1249955527.png)

烧录还需要配置Flash算法文件

### 1. 下载Flash算法文件

在前面准备工作中下载的FLM文件, 放到 C:\Keil_v5\ARM\Flash 目录下, 这个目录下能看到其他的 .FLM 文件.

### 2. 在Keil MDK中的设置

1. 在Keil MDK中, 点击 Project -> Option for Target ... 打开配置对话框, 切换到 Debug 标签页
1. 在Use中选中 CMSIS-DAP Debugger, 点击 Settings 打开对话框, 如果前面都操作正确的话, 这里在SW Device中能看到一个IDCODE为 0x2BA01477的设备
1. 在这个对话框中, 左侧的 CMSIS-DAP - JTAG/SW Adapter 中, 将 Max Clock 修改为 10MHz. 默认的1MHz会导致写入时报 AIR105.FLM 文件无法载入的错误. 

![](https://img2022.cnblogs.com/blog/650273/202201/650273-20220131151006174-2017384750.png)

### 3. 设置FLM和内存大小

1. 在这个对话框, 切换到Flash Download标签页, 可以看到 Programming Algorithm 列表是空白的, 点击 Add, 选中 Air105, 点击 Add 添加到列表
1. 在这个对话框的右上角, RAM for algorithm 部分, 将 Size 修改为 0x00004000. 默认为 0x00001000, 会报内存不足
1. 在这个对话框勾选 Reset and Run, 然后点击 OK 保存

![](https://img2022.cnblogs.com/blog/650273/202201/650273-20220131151027356-8679650.png)

### 4. 烧录

经过上面的配置, 应该就可以直接按F8烧录了
![](https://img2022.cnblogs.com/blog/650273/202201/650273-20220131151153247-1752214218.jpg)

### 5. Debug

Debug 和普通的 STM32F103 系列一样, 通过 Keil MDK 的 Debug 菜单可以直接进入.

## 其他说明

### 制作soc文件

soc文件其实是一个压缩文件, 用7zip之类的工具打开后, 可以将其中的app.bin移除, 换成自己的app.bin. 

### 在keil mdk中生成 bin 格式固件

以 Blink 这个项目为例

1. 打开项目配置文件 Options for Target ...
1. 切换到 User 标签页, 在 After Build/Rebuild 下, 勾选 Run #1, 填入要执行的内容, 填入
```
C:\Keil_v5\ARM\ARMCLANG\bin\fromelf.exe --bin -o  .\Objects\Blink.bin .\Objects\Blink.axf
```
不同的项目, 其相对路径可能不一样, 需要自行调整

### 自建项目, 写入报 No Algorithm found 错误

完整错误为
```
No Algorithm found for: 00000000H - 00001183H
```
这是因为AIR105的 \_\_ROM\_BASE 与标准的 ARMCM4 不一样, 需要修改项目中的 ARMCM4_ac6.sct 文件, 按下面的数值调整
```
#define __ROM_BASE      0x01001000
#define __ROM_SIZE      0x00080000

#define __RAM_BASE      0x20000000
#define __RAM_SIZE      0x000A0000

#define __STACK_SIZE    0x00000200
#define __HEAP_SIZE     0x00000C00
```

# 参考

* MDK项目示例: [https://gitee.com/iosetting/air105_project](https://gitee.com/iosetting/air105_project)
* MDK项目示例(W5500), 上面的项目, 驱动部分代码来源于此 [https://github.com/Wiznet-ShenZhen/MH1903SEVB_Routine](https://github.com/Wiznet-ShenZhen/MH1903SEVB_Routine)
* 合宙项目中的SDK部分 [https://gitee.com/openLuat/luatos-soc-air105/tree/master/bsp/air105](https://gitee.com/openLuat/luatos-soc-air105/tree/master/bsp/air105)  
  * 对比MDK项目示例的 Drivers/Air105_StdPeriph_Driver/inc 部分, 和 luatos-soc-air105/bsp/air105/chip/include 部分, 代码基本是一样的
  * luatos-soc-air105 并没有使用 bsp/air105/chip/include 的代码, 而是另外写了一套位于 bsp/air105/hal 目录