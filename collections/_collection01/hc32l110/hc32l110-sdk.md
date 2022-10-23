---
---

# Log

```bash
$ PATH=$PATH:/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin make

arm-none-eabi-gcc 
	-MT build/HC32L110_DDL_Rev1.1.4/mcu/common/system_hc32l110.o 
	-MMD -MP -MF build/HC32L110_DDL_Rev1.1.4/mcu/common/system_hc32l110.d 
	-fno-common -mcpu=cortex-m0plus -mthumb -Wall -ggdb -Og 
	-I- -Iddl_workarounds -I. -I HC32L110_DDL_Rev1.1.4/mcu/common/ -I HC32L110_DDL_Rev1.1.4/driver/inc 
	-o build/HC32L110_DDL_Rev1.1.4/mcu/common/system_hc32l110.o 
	-c HC32L110_DDL_Rev1.1.4/mcu/common/system_hc32l110.c

cc1: note: obsolete option '-I-' used, please use '-iquote' instead
arm-none-eabi-gcc -MT build/HC32L110_DDL_Rev1.1.4/driver/src/adc.o -MMD -MP -MF build/HC32L110_DDL_Rev1.1.4/driver/src/adc.d -fno-common -mcpu=cortex-m0plus -mthumb -Wall -ggdb -Og -I- -Iddl_workarounds -I. -I HC32L110_DDL_Rev1.1.4/mcu/common/ -I HC32L110_DDL_Rev1.1.4/driver/inc  -o build/HC32L110_DDL_Rev1.1.4/driver/src/adc.o -c HC32L110_DDL_Rev1.1.4/driver/src/adc.c
cc1: note: obsolete option '-I-' used, please use '-iquote' instead
arm-none-eabi-gcc -MT build/HC32L110_DDL_Rev1.1.4/driver/src/gpio.o -MMD -MP -MF build/HC32L110_DDL_Rev1.1.4/driver/src/gpio.d -fno-common -mcpu=cortex-m0plus -mthumb -Wall -ggdb -Og -I- -Iddl_workarounds -I. -I HC32L110_DDL_Rev1.1.4/mcu/common/ -I HC32L110_DDL_Rev1.1.4/driver/inc  -o build/HC32L110_DDL_Rev1.1.4/driver/src/gpio.o -c HC32L110_DDL_Rev1.1.4/driver/src/gpio.c
cc1: note: obsolete option '-I-' used, please use '-iquote' instead
arm-none-eabi-gcc -MT build/HC32L110_DDL_Rev1.1.4/driver/src/interrupts_hc32l110.o -MMD -MP -MF build/HC32L110_DDL_Rev1.1.4/driver/src/interrupts_hc32l110.d -fno-common -mcpu=cortex-m0plus -mthumb -Wall -ggdb -Og -I- -Iddl_workarounds -I. -I HC32L110_DDL_Rev1.1.4/mcu/common/ -I HC32L110_DDL_Rev1.1.4/driver/inc  -o build/HC32L110_DDL_Rev1.1.4/driver/src/interrupts_hc32l110.o -c HC32L110_DDL_Rev1.1.4/driver/src/interrupts_hc32l110.c
cc1: note: obsolete option '-I-' used, please use '-iquote' instead
arm-none-eabi-gcc -MT build/HC32L110_DDL_Rev1.1.4/driver/src/clk.o -MMD -MP -MF build/HC32L110_DDL_Rev1.1.4/driver/src/clk.d -fno-common -mcpu=cortex-m0plus -mthumb -Wall -ggdb -Og -I- -Iddl_workarounds -I. -I HC32L110_DDL_Rev1.1.4/mcu/common/ -I HC32L110_DDL_Rev1.1.4/driver/inc  -o build/HC32L110_DDL_Rev1.1.4/driver/src/clk.o -c HC32L110_DDL_Rev1.1.4/driver/src/clk.c
cc1: note: obsolete option '-I-' used, please use '-iquote' instead
arm-none-eabi-gcc -MT build/HC32L110_DDL_Rev1.1.4/driver/src/ddl.o -MMD -MP -MF build/HC32L110_DDL_Rev1.1.4/driver/src/ddl.d -fno-common -mcpu=cortex-m0plus -mthumb -Wall -ggdb -Og -I- -Iddl_workarounds -I. -I HC32L110_DDL_Rev1.1.4/mcu/common/ -I HC32L110_DDL_Rev1.1.4/driver/inc  -o build/HC32L110_DDL_Rev1.1.4/driver/src/ddl.o -c HC32L110_DDL_Rev1.1.4/driver/src/ddl.c
cc1: note: obsolete option '-I-' used, please use '-iquote' instead
arm-none-eabi-gcc -MT build/startup.o -MMD -MP -MF build/startup.d -fno-common -mcpu=cortex-m0plus -mthumb -Wall -ggdb -Og -I- -Iddl_workarounds -I. -I HC32L110_DDL_Rev1.1.4/mcu/common/ -I HC32L110_DDL_Rev1.1.4/driver/inc  -o build/startup.o -c startup.c
cc1: note: obsolete option '-I-' used, please use '-iquote' instead
arm-none-eabi-gcc -MT build/main.o -MMD -MP -MF build/main.d -fno-common -mcpu=cortex-m0plus -mthumb -Wall -ggdb -Og -I- -Iddl_workarounds -I. -I HC32L110_DDL_Rev1.1.4/mcu/common/ -I HC32L110_DDL_Rev1.1.4/driver/inc  -o build/main.o -c main.c
cc1: note: obsolete option '-I-' used, please use '-iquote' instead

arm-none-eabi-gcc 
	build/HC32L110_DDL_Rev1.1.4/mcu/common/system_hc32l110.o 
	build/HC32L110_DDL_Rev1.1.4/driver/src/adc.o 
	build/HC32L110_DDL_Rev1.1.4/driver/src/gpio.o 
	build/HC32L110_DDL_Rev1.1.4/driver/src/interrupts_hc32l110.o 
	build/HC32L110_DDL_Rev1.1.4/driver/src/clk.o 
	build/HC32L110_DDL_Rev1.1.4/driver/src/ddl.o 
	build/startup.o build/main.o 
	-lgcc -ggdb -mcpu=cortex-m0plus -mthumb -nostartfiles -Wl,-Map=blink.map -Thc32l110.ld -nostdlib 
	-o blink.elf

/opt/gcc-arm/gcc-arm-11.2-2022.02-x86_64-arm-none-eabi/bin/arm-none-eabi-objcopy 
	-I elf32-littlearm 
	-O binary 
	blink.elf 
	blink.bin
```
