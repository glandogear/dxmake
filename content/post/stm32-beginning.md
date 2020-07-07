---
title: "STM32初步，第一段代码、编译流程、烧录"
date: 2020-07-07T15:23:30+08:00
draft: false
tags: ["STM32","ARM","GCC","C"]
categories: ["嵌入式"]
---

第一次学习使用STM32的芯片，有太多要学习的知识，有更多的各种指南和手册，这给我的感觉就是无从下手。现在终于走过了一个完整的开发流程，做一个记录，给诸位提供一些思路。

<!--more-->

## 我的软硬件环境

### 硬件

选板子我主要有几个关注点：TYPE-C接口，可以通过USB下载固件，简洁。所有我没有看那些开发板，最终找到了下面这个设计和做工看起来都不错的板子。

这块板子的芯片是STM32F401CEU6，有复位和BOOT0按键，一个用户按键，一个用户灯，并且预留了FLASH芯片的焊盘。

![STM32板子](/stm32-beginning/01-1.STM32F401_board.png)

### 编译工具

使用gcc-arm-none-eabi工具链，配合makefile管理整个工程的编译工作。

[ARM GNU 工具链](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain)

[GNU make](https://www.gnu.org/software/make/)

### 库

使用ST公司提供的CMSIS库（微控制器软件接口标准，Cortex Microcontroller Software Interface Standard）。

[STM32标准外设软件库](https://www.st.com/zh/embedded-software/stm32-standard-peripheral-libraries.html/)

## 参考资料

- 板子的原理图。
- STM32F401CE的数据手册和参考手册。[产品页面](https://www.st.com/content/st_com/zh/products/microcontrollers-microprocessors/stm32-32-bit-arm-cortex-mcus/stm32-high-performance-mcus/stm32f4-series/stm32f401/stm32f401ce.html)
- AN2606:STM32微控制器系统存储器自举模式。[PDF](https://www.st.com/content/ccc/resource/technical/document/application_note/b9/9b/16/3a/12/1e/40/0c/CD00167594.pdf/files/CD00167594.pdf/jcr:content/translations/zh.CD00167594.pdf)
- CMSIS文档，包含在CMSIS库中。
- [GCC文档](https://gcc.gnu.org/onlinedocs/gcc/)。

## 关于芯片

1. ARM是一家成立于英国剑桥的公司，本身不生产芯片，而是出售芯片设计技术的授权。
2. ARM公司推出了多个系列的产品，从ARM1到ARM11，ARM11之后的产品改用Cortex命名，分为M、R、A三类。随着系列的演进，芯片的体系架构也在进化。
3. 半导体生成厂商们从ARM公司购买ARM处理器核，根据各自应用领域，加入外围电路，形成自己的ARM微处理器芯片。
4. STM32F401CEU6是由ST公司生产的ARM Cortex-M4 MCU，包含512KB Flash和96KB RAM、DSP、FPU、ART加速器，最高主频为84MHz，核心架构为ARMv7E-M。

## 工程建立

首先看下工程的目录结构：

```plaintext
TOP
|-- makefile
|-- 000-libraries
|   `-- STM32F4xx_DSP_StdPeriph_Lib_V1.8.0
|       `-- Libraries
|           |-- CMSIS
|           `-- STM32F4xx_StdPeriph_Driver
`-- 001-GPIOToggle
    |-- makefile
    |-- src
    |   `-- main.c
    |-- lib
    |   |-- startup_stm32f401xx.s
    |   |-- stm32f4xx.h
    |   |-- system_stm32f4xx.c
    |   `-- system_stm32f4xx.h
    |-- scripts
    |   `-- STM32F401VCTx_FLASH.ld
    `-- build
```

顶层目录下的makefile文件为一些基本的变量设置。000-libraries文件夹下存放库文件，其中就包含STM32F4xx的CMSIS库。001-GPIOToggle是我的第一个工程，使用按键控制LED的亮灭，其中lib文件夹下是从CMSIS中拷贝来的可能根据工程而修改的代码。

## 代码编写

作为一个例子，我想要实现按键按下灯点亮，按键松开灯熄灭的效果，本质一点就是芯片端口的输入和输出设置。

首先，查看板子原理图，LED连接到PC13口和3.3V，KEY连接到PA0口和GND。

![LED原理图](/stm32-beginning/05-1.LED.png)

![KEY原理图](/stm32-beginning/05-2.KEY.png)

接着翻看器件的参考手册，在第8章，General-purpose I/Os (GPIO)中讲到：

> Each general-purpose I/O port has four 32-bit configuration registers (GPIOx_MODER,
> GPIOx_OTYPER, GPIOx_OSPEEDR and GPIOx_PUPDR), two 32-bit data registers
> (GPIOx_IDR and GPIOx_ODR), a 32-bit set/reset register (GPIOx_BSRR), a 32-bit locking
> register (GPIOx_LCKR) and two 32-bit alternate function selection register (GPIOx_AFRH
> and GPIOx_AFRL).

大意就是可以通过GPIOx的寄存器来配置端口和控制输入输出。所谓GPIOx，就是PC13口由GPIOC控制，PA0口由GPIOA控制。

每个寄存器的具体内容，在8.4节，GPIO registers中都有说明，包括每个寄存器的复位值、对应端口的bit范围、是否可读写、以及设定值的意义。例如GPIOC_MODER[27:26]用来控制PC13的功能，00表示输入，01表示输出。除了输入输出，还可以设置输出方式（推挽和开漏）、端口速度、上拉下拉。这里我将PC13设置为推挽低速输出，PA0设置为上拉低速输入。

那么该如何在C代码中设置这些寄存器呢？根据CMSIS库文档中CMSIS-CORE/Peripheral Access一节，可以使用`GPIOC->MODER`的形式控制寄存器，或者查看CMSIS头文件`stm32f4xx.h`的源代码，得知GPIO和GPIO_TypeDef的定义：

```c
#define GPIOC      ((GPIO_TypeDef *) GPIOC_BASE)
```

```c
typedef struct
{
  __IO uint32_t MODER;    /*!< GPIO port mode register,               Address offset: 0x00      */
  __IO uint32_t OTYPER;   /*!< GPIO port output type register,        Address offset: 0x04      */
  __IO uint32_t OSPEEDR;  /*!< GPIO port output speed register,       Address offset: 0x08      */
  __IO uint32_t PUPDR;    /*!< GPIO port pull-up/pull-down register,  Address offset: 0x0C      */
  __IO uint32_t IDR;      /*!< GPIO port input data register,         Address offset: 0x10      */
  __IO uint32_t ODR;      /*!< GPIO port output data register,        Address offset: 0x14      */
  __IO uint16_t BSRRL;    /*!< GPIO port bit set/reset low register,  Address offset: 0x18      */
  __IO uint16_t BSRRH;    /*!< GPIO port bit set/reset high register, Address offset: 0x1A      */
  __IO uint32_t LCKR;     /*!< GPIO port configuration lock register, Address offset: 0x1C      */
  __IO uint32_t AFR[2];   /*!< GPIO alternate function registers,     Address offset: 0x20-0x24 */
} GPIO_TypeDef;
```

那么，main.c的代码如下：

```c
// main.c
#include "stm32f4xx.h"
#include <stdint.h>

#define BTN_PIN ((uint16_t)0x0001)
#define LED_PIN ((uint16_t)0x2000)

int main() {
	GPIOA->MODER 	= 0x0C000000;
	GPIOA->OTYPER 	= 0x00000000;
	GPIOA->OSPEEDR 	= 0x0C000000;
	GPIOA->PUPDR 	= 0x64000001;

	GPIOC->MODER 	= 0x04000000;
	GPIOC->OTYPER 	= 0x00000000;
	GPIOC->OSPEEDR 	= 0x00000000;
	GPIOC->PUPDR 	= 0x00000000;

	while(1) {
		if(GPIOA->IDR & BTN_PIN) {		// BTN not pushed
			GPIOC->BSRRL = LED_PIN;		// set bit, LED off
		} else {
			GPIOC->BSRRH = LED_PIN;		// reset bit, LED on
		}
	}
}
```

## 编译流程

顶层目录下的makefile中是工具设置：

```makefile
PREFIX = arm-none-eabi-
ifdef GCC_PATH
	CC = $(GCC_PATH)/$(PREFIX)gcc
	AS = $(GCC_PATH)/$(PREFIX)gcc -x assembler-with-cpp
	CP = $(GCC_PATH)/$(PREFIX)objcopy
	SZ = $(GCC_PATH)/$(PREFIX)size
else
	CC = $(PREFIX)gcc
	AS = $(PREFIX)gcc -x assembler-with-cpp
	CP = $(PREFIX)objcopy
	SZ = $(PREFIX)size
endif
HEX = $(CP) -O ihex
BIN = $(CP) -O binary -S
```

编译的基本流程是，首先将所有的源文件包括.c和.s文件编译为目标文件.obj，然后将所有的.obj链接成为.elf文件，再通过.elf文件转换得到.hex和.bin文件。编译和链接选项为`C_FLAGS`、`AS_FLAGS`、`LD_FLAGS`，具体涵义可以参考[GCC文档](https://gcc.gnu.org/onlinedocs/gcc/)。其中一部分选项：

- -mcpu=cortex-m4，设置ARM处理器
- -mthumb，设置指令集
- -W -Wall，打开警告
- -fdata-sections -ffunction-sections，将每个函数或符号创建为一个sections，配合-Wl,–gc-sections指示链接器去掉不用的section，能够减少最终的可执行程序的大小。

链接脚本STM32F401VCTx_FLASH.ld是从官方的例子中拷贝来的，主要需要修改的地方如下：

```c
/* Entry Point */
ENTRY(Reset_Handler)

/* Highest address of the user mode stack */
_estack = 0x20018000;    /* end of RAM */
/* Generate a link error if heap and stack don't fit into RAM */
_Min_Heap_Size = 0x200;      /* required amount of heap  */
_Min_Stack_Size = 0x400; /* required amount of stack */

/* Specify the memory areas */
MEMORY
{
FLASH (rx)      : ORIGIN = 0x08000000, LENGTH = 512K
RAM (xrw)      : ORIGIN = 0x20000000, LENGTH = 96K
}
```

工程makefile：

```makefile
######################################
# build tools
######################################
TOP_DIR = $(abspath ..)
include $(TOP_DIR)/makefile

######################################
# build settings
######################################
TARGET = GPIOToggle
DEBUG = 0
OPT = -Og

######################################
# pahts
######################################
BUILD_DIR = ./build
LIBRARY_DIR = $(TOP_DIR)/000-libraries/STM32F4xx_DSP_StdPeriph_Lib_V1.8.0/Libraries

######################################
# sources
######################################
C_SOURCES = ./src/main.c \
            ./lib/system_stm32f4xx.c
ASM_SOURCES = ./lib/startup_stm32f401xx.s

######################################
# C_FLAGS
######################################
CPU = -mcpu=cortex-m4
FPU = -mfpu=fpv4-sp-d16
FLOAT-ABI = -mfloat-abi=hard
MCU = $(CPU) -mthumb $(FPU) $(FLOAT-ABI)

C_DEFS = -DSTM32F401xx
C_INCLUDES = -I./lib \
             -I$(LIBRARY_DIR)/CMSIS/Include

C_FLAGS = $(MCU) $(C_DEFS) $(C_INCLUDES) $(OPT) \
          -std=gnu11 -W -Wall -fdata-sections -ffunction-sections

ifeq ($(DEBUG), 1)
	C_FLAGS += -g -gdwarf-2
endif
C_FLAGS += -MMD -MP -MF"$(addprefix $(BUILD_DIR)/, $(notdir $(@:%.o=%.d)))"

######################################
# AS_FLAGS
######################################
AS_DEFS = 
AS_INCLUDES = 

AS_FLAGS = $(MCU) $(AS_DEFS) $(AS_INCLUDES) $(OPT) \
           -W -Wall -fdata-sections -ffunction-sections

######################################
# LD_FLAGS
######################################
LD_SCRIPT = ./scripts/STM32F401VCTx_FLASH.ld

LIBS = -lc -lm -lnosys
LIBDIR = 
LD_FLAGS = $(MCU) -specs=nano.specs -specs=nosys.specs \
           -T$(LD_SCRIPT) $(LIBDIR) $(LIBS) \
           -Wl,-Map=$(BUILD_DIR)/$(TARGET).map,--cref \
           -Wl,--no-wchar-size-warning -Wl,--gc-sections

######################################
# build the application
######################################
OBJECTS = $(addprefix $(BUILD_DIR)/,$(notdir $(C_SOURCES:.c=.o)))
OBJECTS += $(addprefix $(BUILD_DIR)/,$(notdir $(ASM_SOURCES:.s=.o)))

vpath %.c $(sort $(dir $(C_SOURCES)))
vpath %.s $(sort $(dir $(ASM_SOURCES)))

.PHONY: all hex bin elf obj clean
all: $(BUILD_DIR) obj elf hex bin
hex: $(BUILD_DIR)/$(TARGET).hex
bin: $(BUILD_DIR)/$(TARGET).bin
elf: $(BUILD_DIR)/$(TARGET).elf
obj: $(OBJECTS)

$(BUILD_DIR)/$(TARGET).hex: $(BUILD_DIR)/$(TARGET).elf
	@ echo "HEX   $<"
	@ $(HEX) $< $@

$(BUILD_DIR)/$(TARGET).bin: $(BUILD_DIR)/$(TARGET).elf
	@ echo "BIN   $<"
	@ $(BIN) $< $@

$(BUILD_DIR)/$(TARGET).elf: $(OBJECTS) makefile
	@ echo "LD    $(OBJECTS)"
	@ $(CC) $(OBJECTS) $(LD_FLAGS) -o $@
	@ $(SZ) $@

$(BUILD_DIR)/%.o: %.c makefile | $(BUILD_DIR)
	@ echo "CC    $<"
	@ $(CC) -c $(C_FLAGS) $< -o $@

$(BUILD_DIR)/%.o: %.s makefile | $(BUILD_DIR)
	@ echo "AS    $<"
	@ $(AS) -c $(AS_FLAGS) $< -o $@

$(BUILD_DIR):
	@ mkdir $@

clean:
	@ echo "CLEAN"
	@ rm -rf $(BUILD_DIR)
```

## 烧录

根据资料AN2606：

> 自举程序存储在 STM32 器件的内部自举 ROM 存储器（系统存储器）中。在芯片生产期间由 ST 编程。其主要任务是通过一种可用的串行外设（USART、CAN、USB、I2C 等）将应用程序下载到内部 Flash 中。每种串行接口都定义了相应的通信协议，其中包含兼容的命令集和序列。

以及27节，STM32F401xD(E) 器件自举程序的说明，这个芯片支持通过USB的DFU下载，只需要在器件上电时设置BOOT1=0，BOOT0=1，就可以进入自举程序模式。

首先通过官方提供的DFU File Manager，将HEX文件转换为DFU文件，再通过官方提供的DfuSe工具，就可以将程序下载程序到硬件中了。

![DFU File Manager](/stm32-beginning/07-1.DFU-File-Manager.png)

![DFU File Manager](/stm32-beginning/07-2.DfuSe.png)

出于方便的考虑，可以在板子上烧一个bootloader，比如说[STM32_HID_Bootloader](https://github.com/Serasidis/STM32_HID_Bootloader)，这样就不用每次进DFU模式和转换文件了。

