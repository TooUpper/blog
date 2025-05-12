---
title: "Init"
date: 2024-10-27 22:03:00
categories: "ARM"
tags: 
- "STM32F407"
---

## 准备阶段

下载相关文件，STM32F407 的参考手册、数据手册、选型手册、固件使用指南（一般下载固件库时会附带）等。
[STM32标准外设软件库 - 意法半导体STMicroelectronics](https://www.st.com.cn/zh/embedded-software/stm32-standard-peripheral-libraries.html)
[STM32文档手册 - 意法半导体STMicroelectronics](https://www.st.com.cn/zh/microcontrollers-microprocessors/stm32-32-bit-arm-cortex-mcus.html)

## 创建项目与配置

**创建项目**
在工程目录中创捷下列文件夹

```c
// 目录结构
Demo01
└── Docs/		 // 说明文档 README.md
└── Project/	 // 存放 Keil 文件和生成的一些配置文件、hex 文件     
├── CMSIS/       // CMSIS 中 ARM 相关文件
├── Firmware/    // 官方标准固件库中的驱动程序
├── Drivers/     // 外设硬件驱动程序
├── Middleware/  // 上层中间件组件
├── App/         // 应用层入口代码    
```

**文件移植**

将官方标准固件库中我们需要的的文件拷贝到对应的工程目录中去：
**CMSIS** --> 官方标准固件库中的 CMSIS 中的代码；
**Firmware** --> 官方标准固件库中的外设驱动；
**Drivers** --> 我们自己封装的驱动，如按键、显示屏等；
**Middleware** --> 项目中的中间件，如上位机的可视化界面等；
**App** --> 程序的入口和具体的业务处理逻辑；
**Project** --> Keil 工程的目录；

那么具体都是要拷贝哪些软件呢：
查看官方的示例代码，看看人家官方示例是需要拷贝那些：
	打开 Keil 查看他的工程目录，然后从官方库中复制对应的文件到我们自己的项目中；(注意，一般工程中只会显示 .c 文件但我们要记得将对应的 .h 文件一起拷贝)
	注意在每一款芯片的固件库中都包含一个用于导入所有固件库的头文件，这个头文件在示例工程中可能并没有导入，但我们不要忘记导入，他一般存在于示例代码的模板目录下，STM32 中叫做 stm32f4xx_conf.h；

**在 Keil 工程中创建工程目录并添加对应的文件**
创建 Keil 工程并新建对应的工程目录
将工程文件夹下的 .C 文件添加到 Keil 工程中去

![F407Keilyinruwenjian](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407Keilyinruwenjian.png)

**添加 Include PATH 路径**

![F407Keilinclude](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407Keilinclude.png)

**其他 Keil 配置**

1. **芯片的型号要与所用硬件一致**
   "魔术棒" --> "Device" 下的芯片型号与我们用的硬件是否一样，不一样就要改；

2. **编译器版本是否正确**
   "魔术棒" --> "Target" 下 ARM Compiler 的版本是否为 "Use default compiler version 6"，如果 “missing compiler version 5” 表示 5 这个版本的不存在那么我们也要去安装；
   这里的编译工具版本最好和标准库中是实例代码保持一致；
   勾选 "Use Micro LIB" 以优化代码；

3. **是否生成 .HEX 文件**
   “魔术棒” --> "Output" 下勾选 "Create HEX File";

4. **设置 C 和 C++ 的编译器版本，不然有些宏关键字它可能不识别，设置代码优化级别**
   "魔术棒" --> "C/C++" 下的 Language C 设置为 "C990"，Language C++ 设置为 "C++ 11"
   设置编译警告的级别，选择 "Wamings" 中的内容为 "AC5-like Wamings"，表示与 Keil5 一致；
   设置代码优化的级别，选择 "Optimizaion" 中的内容为 "-O1"，级别越高优化程序越高，但是太高了有时候会出错，此处设置为 "-O1" 即可；

5. **添加 Keil 工程中的使用文件的路径**
   "魔术棒" --> "C/C++" 下的 "Include Path" 中进行添加；

6. **选择所使用的烧录器**
   "魔术棒" --> "Debug" 下右侧 "Use" 中的烧录器，此处我们设置为 "CMSIS-DAP Debugger";
   勾选自动复位并运行，左侧 "Setting" --> "Flash Download" 中勾选 "Reset and Run";
   通常如果烧录器连上后可以在  "Setting" --> "Debug" 右侧中的 "SW Device" 中看到；

## 开发与调试

1. **示例的 main.c 中使用的芯片与我们不一致，所以需要删掉他的头文件和其代码**

```c
#include "stm32f4xx.h"
#include <stdio.h>

int main(void) {
	
	while(1) {
	
	}

}
```

## 烧写与验证

在嵌入式开发中，**拿到一块开发板后，烧录方式不能随意选择**，而是需要**根据开发板的硬件支持情况**来选择适合的烧录方法。

**如何查看开发板的烧录方式支持情况**

- **查看数据手册**：芯片的数据手册通常会列出支持的烧录接口和方法，例如 ST-Link、J-Link、UART、USB DFU 等。

- **检查开发板的接口**：查看开发板是否有特定的调试/编程接口，例如 **SWD** 接口、**JTAG** 接口、**USB** 接口、或 **UART** 引脚等。

- **开发板文档**：一些开发板还会提供专门的用户手册，其中会详细说明支持的烧录方法和推荐的工具。

### DAP-Link 下载

1. DAP-Link 连线

![F407DAPlianxian](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407DAPlianxian.png)

2. Keil 设置

Debug 中的设置此处不再赘述，如果发现一直无法识别烧录器则需要换一个烧录器或者换一种烧录方式；

3. 烧录

"主界面" --> "保存" --> "Rebuild" --> "Download" 即可

![F407shaolumulu](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407shaolumulu.png)

### 使用 DFU 烧录

**下载安装 CH340 驱动**

[CH341SER.EXE - 南京沁恒微电子股份有限公司](https://www.wch.cn/download/CH341SER_EXE.html)

**下载安装 DFU 驱动**

[兆易创新GigaDevice-资料下载兆易创新GD32 MCU](https://www.gd32mcu.com/cn/download?kw=DFU&lan=cn)

**下载烧录软件**

[兆易创新GigaDevice-资料下载GigaDevice GD32 MCU](https://www.gd32mcu.com/en/download?kw=GD32+All-In-One+Programmer)

**Keil配置**

“魔术棒” --> "Output" 下勾选 "Create HEX File";

**烧录器使用**

1. 解压后以管理员身份运行
2. 软件配置

![F407DAPpezhi01](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407DAPpezhi01.png)

3. 开始进入升级模式。首先按住BOOT0不要松手，然后再按一次RESET进入到升级模式。

进入升级模式成功之后，会在软件中显示设备 **GD DFU DEVICE 1** 。如果没有进入请多次尝试。

![F407DAPpeizhi02](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407DAPpeizhi02.png)

4. 点击 "Connect" 连接开发板。

连接成功之后，会显示出芯片内存大小。

![F407DAPpeizhi03](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407DAPpeizhi03.png)

5. 查看有无读写保护

主界面中选择 "Erase selected pages" ，点击 "Erase"，如果有读写保护则需要解除保护如果没有就不管他

6. 下载测试代码

在主界面，点击 "Browse" ,选择 .hex 文件，之后点击 "Download"；

然后我们按一下复位按键，就可以让程序开始运行。

6. 解除写保护

在主界面，点击 "Edit Option Bytes";

主要内容为修改 SPC 的值为非 0XAA 和非 0XCC 值，比如 0x55

![F407DAPpeizhi04](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407DAPpeizhi04.png)

> 但我测试时候发现，将 WP0 下面的两个沟去掉也可以接触写保护;

## 调整时钟频率

我们在测试中发现设置为 1S 的时间间隔与实际运行时的时间相差太大，这是我们就要去修改系统的始终频率，让其与我们这个芯片所支持的时钟频率保持一致；

芯片选型手册

![F407xinpianxuanxingbiao](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407xinpianxuanxingbiao.png)

这张图表展示了 GD32F407VET6 芯片的主要参数和特性，以下是对各项的详细说明：

**基本参数**

- **Max Speed (MHz)** - **最大速度**:
  - GD32F407VET6 的主频最高可达 168 MHz，适合处理需要较高运算速度的嵌入式应用场景。

- **Memory (Bytes)** - **内存**:

  - **Flash**: 512 KB 的内部 Flash 存储，用于存储程序代码。

  - **SRAM**: 192 KB 的 SRAM，供程序运行时使用，存储临时数据和变量。

**输入/输出 (I/O)**

- **I/O Pins**: 提供了最多 82 个 GPIO 引脚（通用输入输出引脚），这些引脚可以用于连接外部设备或控制器件。

**定时器 (Timer)**

- **Adv TM (16bit)** - **高级定时器**:
  - 包含 2 个 16 位高级定时器，通常用于需要复杂 PWM 生成的应用，比如电机控制。

- **GPTM (16bit)** - **通用定时器**:
  - 配置了 4 个 16 位通用定时器，适合用于普通计时任务或输入捕获。

- **GPTM (32bit)** - **32 位通用定时器**:
  - 配置了 2 个 32 位通用定时器，适合计时精度要求较高的场景。

- **Basic TM (16bit)** - **基本定时器**:
  - 配置了 2 个基本定时器，适合简单的计数或时间延迟任务。

- **WDG** - **看门狗定时器**:
  - 配置了 2 个看门狗定时器，用于在程序异常时重启系统，以确保系统安全。

- **RTC** - **实时时钟**:
  - 具备 1 个 RTC 实时时钟，通常用于计时任务，如数据记录或时间戳生成。

**通信接口 (Connectivity)**

- **USART + UART**:
  - 提供 4 个 USART 和 2 个 UART 接口，可以用于串口通信，如与其他设备通信或调试信息输出。

- **I²C**:
  - 配置了 3 个 I²C 接口，用于与 I²C 总线设备通信，比如传感器和存储设备。

- **SPI**:
  - 包含 3 个 SPI 接口，支持与 SPI 设备的高速数据交换，比如与 ADC、DAC 通信。

- **CAN 2.0B**:
  - 包含 2 个 CAN 2.0B 接口，用于工业控制网络中的数据传输。

- **USB OTG**:
  - 配置了 USB 全速 (FS) 和高速 (HS) 双模式 OTG（On-The-Go）接口，可以用于 USB 设备或主机功能。

- **I²S**:
  - 具有 2 个 I²S 接口，用于音频数据传输。

- **SDIO**:

- 支持 SDIO 接口，可以与 SD 卡通信，用于数据存储应用。

1. **LCD-SDIO TFT**:
   - 提供 1 个 LCD 接口，用于连接显示屏（如 TFT 屏幕）。
2. **Camera**:
   - 配置 1 个摄像头接口，支持外接摄像头模块，实现图像采集功能。
3. **ETH MAC**:
   - 配置了 1 个 Ethernet MAC 接口，可以用于网络通信应用。

**模拟接口 (Analog Interface)**

- **12-bit ADC Units (CHs)** - **ADC 单元和通道**:
  - 配置了 3 个 12 位 ADC，拥有总共 16 个通道，用于模拟信号采集。

- **12-bit DAC Units** - **DAC 单元**:
  - 配置了 2 个 12 位 DAC，用于将数字信号转换为模拟信号输出。

**存储器接口 (EMMC/SDRAM)**

- EXMC
  - 配置了 1 个 EXMC 外部存储控制器接口，可以连接外部 SDRAM 或 NOR Flash 扩展存储器。

**封装 (Package)**

- LQFP100
  - 该型号的芯片封装类型为 LQFP100（100 引脚的低轮廓方形封装），方便焊接和电路板设计。

根据手册我们要设置系统主频为 168 MHz，那么要如何实现这个 168MHz 的频率，这时候就要去看板子商家的原理图和用户手册了，看他们是如何实现的，然后选择为对应的方式，如果他们设置了外部就选择外部否则可以选择内部的；

![F407jinzhenbufen](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407jinzhenbufen.png)

通过板子的原理图可以看出它使用的是外部 8MHz 所以我们在 system_gd32f4xx.c 中要放开对下面代码的注释

```c
#define __SYSTEM_CLOCK_168M_PLL_8M_HXTAL        (uint32_t)(168000000)
```

- **如果板子有 25MHz 的外部晶振**，则选择 `__SYSTEM_CLOCK_168M_PLL_25M_HXTAL`。

- **如果板子有 8MHz 的外部晶振**，则选择 `__SYSTEM_CLOCK_168M_PLL_8M_HXTAL`。

- **如果没有外部晶振（即使用内部16MHz振荡器）**，则选择 `__SYSTEM_CLOCK_168M_PLL_IRC16M`。

## 调整晶振的频率

我们这个板子使用的是外部高速的 8MHz 我们要将代码中使用的改为 8MHz

![F407xitshizhong](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407xitshizhong.png)

![F407xitongshizhong02](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407xitongshizhong02.png)

板子上使用的是外部的晶振，她本身不知道会用到多大的，所以创建项目时候要记得去修改系统时钟相关的配置；

## 问题

一、**../CMSIS\stm32f4xx.h(115): error: "Please select first the target STM32F4xx device used in your application (in stm32f4xx.h file)"**

这个报错是因为在  stm32f4xx.h 文件中，你还没有选择具体的 STM32F4 系列微控制器型号。

我们需要添加宏已

二、**../CMSIS\core_cmFunc.h(629): error: unknown register name 'vfpcc' in asm**

这个错误通常表示编译器无法识别 `vfpcc` 寄存器，可能是因为当前的编译设置或目标架构不正确。

我们需要修改 ARM 编译器的版本将他改为 5 或者 6；

三、**..\Firmware\misc.c(114): warning:  #223-D: function "assert_param" declared implicitly**

这个警告表示 `assert_param` 函数在使用时没有找到相应的声明。

这是我们要点击报错信息，找到这条语句，右键跳转到他的定义文件中去，这时候大概会提示文件不存在，例如提示 stm32f4xx_fmc.c 文件不存在，他有一个头文件用于导入大部分的估

四、**.\Objects\project.sct(7): error: L6236E: No section matches selector - no section to be FIRST/LAST.**

表示没有添加启动文件
