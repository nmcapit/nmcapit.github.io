---
---

# 目录

* [HC32L110(一) HC32L110芯片介绍和Win10下的烧录](https://www.cnblogs.com/milton/p/16586369.html)
* [HC32L110(二) HC32L110在Ubuntu下的烧录](https://www.cnblogs.com/milton/p/16586831.html)
* [HC32L110(三) HC32L110的GCC工具链和VSCode开发环境](https://www.cnblogs.com/milton/p/16634604.html)
* [HC32L110(四) HC32L110的startup启动文件和ld连接脚本](https://www.cnblogs.com/milton/p/16651892.html)
* [HC32L110(五) Ubuntu20.04 VSCode的Debug环境配置](https://www.cnblogs.com/milton/p/16653827.html)

以下介绍项目中的startup和ld文件, 以及HC32L110的启动机制

仓库地址: [https://github.com/IOsetting/hc32l110-template](https://github.com/IOsetting/hc32l110-template)

如果转载, 请注明出处.

# 关于

因为是面向 GCC Arm Embedded 工具链的版本, 所以 startup 代码和 ld 连接描述脚本都依据 GCC Arm 工具链的格式.

# Startup文件说明

startup_hc32l110.c 文件位于 Libraries/CMSIS

```c
// 为下面的 uint32_t 等类型引入定义
#include <stdint.h>

// 将 ptr_func_t 定义为函数指针
typedef void (*ptr_func_t)();

// 下面这三个 __data 开头的变量是一组, 用于载入变量预先定义的值. 这些地址在连接阶段, 根据区域的实际情况被赋值

// __data_start 是载入的目标起始地址
extern uint32_t __data_start;
// __data_end 是载入的目标结束地址
extern uint32_t __data_end;
// 载入值的来源
extern uint32_t __data_load;

// __bss 开头的变量, 代表启动时需要清零的变量, __bss_start 和 __bss_end 分别代表了内存的起始和结束地址, 也是连接阶段会赋值
extern uint32_t __bss_start;
extern uint32_t __bss_end;

extern uint32_t __heap_start;
extern uint32_t __stacktop;

// 初始化, 在进入main函数之前需要执行的方法列表
extern ptr_func_t __init_array_start[];
extern ptr_func_t __init_array_end[];

// 引入外部定义的 SystemInit 和 main 方法
extern int main(void);
extern void SystemInit(void);

// 弱函数别名, 在对应的函数未定义时, 会调用别名对应的函数
#define WEAK_ALIAS(x) __attribute__ ((weak, alias(#x)))

// 下面这些都是中断函数
/* Cortex M3 core interrupt handlers */
void Reset_Handler(void);
void NMI_Handler(void) WEAK_ALIAS(Dummy_Handler);
void HardFault_Handler(void) WEAK_ALIAS(Dummy_Handler);
void SVC_Handler(void) WEAK_ALIAS(Dummy_Handler);
void PendSV_Handler(void) WEAK_ALIAS(Dummy_Handler);
void SysTick_Handler(void) WEAK_ALIAS(Dummy_Handler);

// 直接用中断号作为函数名, 具体的对应关系在 ddl.h 中, 
// 这些是沿用官方DDL驱动代码, 将来会替换为直接调用实际的中断处理函数
void IRQ000_Handler(void) WEAK_ALIAS(Dummy_Handler);
void IRQ001_Handler(void) WEAK_ALIAS(Dummy_Handler);
void IRQ002_Handler(void) WEAK_ALIAS(Dummy_Handler);
// 中间略
void IRQ031_Handler(void) WEAK_ALIAS(Dummy_Handler);

/* 将 __stacktop 初始地址记录到 __stack_init. 
关于 used 的定义: 即是未被使用, 编译后也需要保留
This attribute, attached to a function, means that code must be emitted 
for the function even if it appears that the function is not referenced. 
This is useful, for example, when the function is referenced only in 
inline assembly.
*/
__attribute__((section(".stack"), used)) uint32_t *__stack_init = &__stacktop;

/* Stack top and vector handler table 
中断向量表, 这些函数, 和前面的定义需要一致. 这些函数的实际逻辑在 ddl.h 和 ddl.c中定义.
*/
__attribute__ ((section(".vectors"), used)) void *vector_table[] = {
    Reset_Handler,
    NMI_Handler,
    HardFault_Handler,
    0,
    0,
    0,
    0,
    0,
    0,
    0,
    SVC_Handler,
    0,
    0,
    PendSV_Handler,
    SysTick_Handler,
    IRQ000_Handler,
    IRQ001_Handler,
    IRQ002_Handler,
    IRQ003_Handler,
// 中间略
    IRQ029_Handler,
    IRQ030_Handler,
    IRQ031_Handler};

/*
最重要的, 重启后的初始化方法, 由ld文件中的 ENTRY(Reset_Handler) 指定
*/
__attribute__((used)) void Reset_Handler(void)
{
    uint32_t *src, *dst;

    /* 从 Flash 到 RAM 复制变量值 */
    src = &__data_load;
    dst = &__data_start;
    while (dst < &__data_end) *dst++ = *src++;

    /* 清空 bss section */
    dst = &__bss_start;
    while (dst < &__bss_end) *dst++ = 0;

    // 这里调用前面声明的 SystemInit
    SystemInit();
    // 调用初始化函数列表
    for (const ptr_func_t *f = __init_array_start; f < __init_array_end; f++)
    {
        (*f)();
    }

    // 调用前面声明的main
    main();
}

// 默认的中断处理方法
void Dummy_Handler(void)
{
    while (1);
}
```

# LD文件说明

以hc32l110x4.ld为例
```
/* MEMORY 内存块配置, 格式为

MEMORY
{
  name [(attr)] : ORIGIN = origin, LENGTH = len
  …
}
*/
MEMORY
{
    FLASH  (rx): ORIGIN = 0x00000000, LENGTH = 16K
    RAM   (rwx): ORIGIN = 0x20000000, LENGTH = 2K
}

// 运行一个程序时第一个被执行到的指令称为"入口点", 默认是start, 可以使用"ENTRY"连接脚本命令来设置入口点.参数是一个符号名
ENTRY(Reset_Handler)

/* "SECTIONS"命令是链接脚本中最重要的部分, 段命令格式如下, 会包含多个 secname, 区域必须已经在MEMORY中定义

SECTIONS {
...
secname start BLOCK(align) (NOLOAD) : AT ( ldadr )
  { contents } >region :phdr =fill
...
}
*/
SECTIONS
{
    // 当前地址为FLASH区域起始地址
   . = ORIGIN(FLASH);
   // 
   .text : {
        // KEEP 命令主要作用是防止垃圾收集机制把这几个重要的节排除在外，另外也保证堆栈和向量表在段中的位置处于最顶端
        KEEP(*(.stack))
        // 对应startup里面的 section(".vectors")
        KEEP(*(.vectors))
        KEEP(*(.vectors*))
        // .text: 所有的编译出来的代码段，都放在这里
        KEEP(*(.text))
        // 通过 ALIGN 命令, 将当前地址指针调整到4字节对齐
        . = ALIGN(4);
        *(.text*)
        . = ALIGN(4);
        // 常量数据的代码段
        KEEP(*(.rodata))
        *(.rodata*)
        // 当前指针, 调节到4字节对齐后的地址
        . = ALIGN(4);
    } >FLASH // 这个段放在名为FLASH的内存块

    // 初始化方法指针队列
    .init_array ALIGN(4): {
        __init_array_start = .;
        KEEP(*(.init_array))
        __init_array_end = .;
    } >FLASH

    __stacktop = ORIGIN(RAM) + LENGTH(RAM);
    // LOADADDR(.data) 获取.data段的加载地址(lma)，也就是data段在Flash中存放的起始地址
    __data_load = LOADADDR(.data);
    // 当前地址为 RAM 区域起始地址
    . = ORIGIN(RAM);

    // 数据部分, 可以看下面对 __data_start 和 __data_end 的赋值方式
    .data ALIGN(4) : {
        __data_start = .;
        *(.data)
        *(.data*)
        . = ALIGN(4);
        __data_end = .;
    } >RAM AT >FLASH // 这些变量位于RAM, 值会从FLASH的对应区域载入

    /* 可以看下面对 __bss_start 和 __bss_end 的赋值方式
       关于 NOLOAD: The `(NOLOAD)' directive will mark a section to not be loaded 
       at run time. The linker will process the section normally, but will mark 
       it so that a program loader will not load it into memory
       就是会正常连接, 但是运行时不载入内存
    */
    .bss ALIGN(4) (NOLOAD) : {
        __bss_start = .;
        *(.bss)
        *(.bss*)
        . = ALIGN(4);
        __bss_end = .;
        *(.noinit)
        *(.noinit*)
    } >RAM    // 这些变量位于RAM

    . = ALIGN(4);
    __heap_start = .;
}
```

# 参考

* GNU linker ld (GNU Binutils) version 2.39 [https://sourceware.org/binutils/docs-2.39/ld/index.html](https://sourceware.org/binutils/docs-2.39/ld/index.html)
* GCC链接脚本(.ld)文件详解 [https://blog.csdn.net/a2529280665/article/details/121576020](https://blog.csdn.net/a2529280665/article/details/121576020)