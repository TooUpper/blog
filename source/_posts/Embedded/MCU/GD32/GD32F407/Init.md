---
title: "Init"
date: 2024-10-24 19:48:00
categories: "ARM"
tags: 
- "GD32F407"
---

## 准备阶段

下载相关文件，GD32F407 的用户手册、数据手册、选型手册、固件使用指南（一般下载固件库时会附带）等。

[官网下载地址](https://www.gd32mcu.com/cn/download/)

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
├── Middleware/  // 中间件组件
├── App/         // 应用层代码
└── User/        // 入口层     
```

**文件移植**

将官方标准固件库中我们需要的的文件拷贝到对应的工程目录中去：

> **CMSIS** --> 官方标准固件库中的 CMSIS 中的代码；
>
> **Firmware** --> 官方标准固件库中的外设驱动；
>
> **Drivers** --> 我们自己封装的驱动，如按键、显示屏等；
>
> **Middleware** --> 项目中的中间件，如上位机的可视化界面等；
>
> **App** --> 具体的业务处理逻辑
>
> **User** --> 程序的入口和一些工具文件
>
> **Project** --> Keil 文件的目录

那么具体都是要拷贝哪些软件呢，

- 将对应目录下文件都拷贝过去，然后将所有 .c 文件添加进项目中然后在解决错误问题(不建议）

- 查看官方的示例代码，看看人家官方示例是需要拷贝那些，
  - 打开 Keil 查看他的工程目录，然后从官方库中复制对应的文件到我们自己的项目中；(注意，一般工程中只会显示 .c 文件但我们要记得将对应的 .h 文件一起拷贝)

**在 Keil 工程中创建工程目录并添加对应的文件**

- 创建 Keil 工程并新建对应的工程目录
- 将工程文件夹下的 .C 文件添加到 Keil 工程中去

![F407Keilyinruwenjian](/public/image/Embedded/MCU/GD32/GD32F407/F407Keilyinruwenjian.png)

**添加 Include PATH 路径**

![F407Keilinclude](/public/image/Embedded/MCU/GD32/GD32F407/F407Keilinclude.png)

### Keil配置

1. **芯片的型号要与所用硬件一致**

""魔术棒" --> "Device" 下的芯片型号与我们用的硬件是否一样，不一样就要改；

2. **编译器版本是否正确**

""魔术棒" --> "Target" 下 ARM Compiler 的版本是否为 "Use default compiler version 6"，如果 “missing compiler version 5” 表示 5 这个版本的不存在那么我们也要去安装；

勾选 "Use Micro LIB" 以优化代码；

> 这里注意，编译器版本最好与官方提供的示例代码中的版本一致，不然会有非常多的一些警告

3. **是否生成 .HEX 文件**

“魔术棒” --> "Output" 下勾选 "Create HEX File";

4. **设置 C 和 C++ 的编译器版本，不然有些宏关键字它可能不识别，设置代码优化级别**

"魔术棒" --> "C/C++" 下的 Language C 设置为 "C99"，Language C++ 设置为 "C++ 11"

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
#include "gd32f4xx.h"
#include "systick.h"
#include <stdio.h>
#include "main.h"
#include "gd32f450i_eval.h" // 删掉

// 整个函数删掉
void led_spark(void) {
    static __IO uint32_t timingdelaylocal = 0U;

    if(timingdelaylocal) {

        if(timingdelaylocal < 500U) {
            gd_eval_led_on(LED1);
        } else {
            gd_eval_led_off(LED1);
        }

        timingdelaylocal--;
    } else {
        timingdelaylocal = 1000U;
    }
}

int main(void) {

    gd_eval_led_init(LED1); // 这个调用语句也删掉
    systick_config();

    while(1) {
    }
}

// gd32f4xx_it.c 中调用了 led_spark() 我们也要讲他删掉
```

2. **编译运行这时他会报错，提示缺少一些文件，我们去看这些文件在我们的目录中是否存在，如果不存在就加进来**

   这里一般情况下是缺少 "core_cmFunc.h"，"core_cm4.h"，"core_cm4_simd.h"，"core_cmlnstr.h"，这四个

3. **此时还是报错，我们点击报错的语句，发现他会跳到某些个 include 处报错：D:/Tools/IDE/keil/keil/ARM/Packs/GigaDevice/GD32F4xx_DFP/3.0.3/Device/F4XX/Include\gd32f4xx_libopt.h(11): error: 'RTE_Components.h' file not found；**

- 注意看他这个报错信息中的路径好像用的不是我们本地的那个呀，那么 ok，我们首先要将 gd32f4xx_libopt.h 包含在我们的项目本地，确保他在本地是存在的，

- gd32f4xx_libopt.h 这个文件是在哪被声明使用的呢？，我们发现，他是在 gd32f4xx.h 中被声明使用的，那么 gd32f4xx.h 这个文件在本地是否存在呢，如果不存在就需要从标准固件库中添加进我们的项目中；

此时，运行它使用的就是我们本地的 gd32f4xx_libopt.h 文件了，这时它有时候还会报错某些引用错误，但这是官方的标准文件是不会出错了，那么大概率就是某些头没有导入，他某些头文件的导入是通过宏来定义的；我们需要在 Keil 中定义某些宏；

4. **简化外设库的使用**

我们看 gd32f4xx_libopt.h 文件发现其中定义了很多的头文件，但是它需要通过宏去判断是否导入这些头文件，所以此时我们需要添加这些宏关键字；

"魔术棒" --> "C/C++" 下的 "Define" 中进行添加；"USE_STDPERIPH_DRIVER,GD32F407"，这两个宏定义，

USE_STDPERIPH_DRIVER 用于在 gd32f4xx.h 中展开 gd32f4xx_libopt.h 头文件（这一句可以不加，它本身会做判断，不存在就自动定义）；

GD32F407 用于在 gd32f4xx_libopt.h 展开标准固件库中的头文件；

## 烧写与验证

在嵌入式开发中，**拿到一块开发板后，烧录方式不能随意选择**，而是需要**根据开发板的硬件支持情况**来选择适合的烧录方法。

**如何查看开发板的烧录方式支持情况**

- **查看数据手册**：芯片的数据手册通常会列出支持的烧录接口和方法，例如 ST-Link、J-Link、UART、USB DFU 等。

- **检查开发板的接口**：查看开发板是否有特定的调试/编程接口，例如 **SWD** 接口、**JTAG** 接口、**USB** 接口、或 **UART** 引脚等。

- **开发板文档**：一些开发板还会提供专门的用户手册，其中会详细说明支持的烧录方法和推荐的工具。

### DAP-Link 下载

1. DAP-Link 连线

![F407DAPlianxian](/public/image/Embedded/MCU/GD32/GD32F407/F407DAPlianxian.png)

2. Keil 设置

Debug 中的设置此处不再赘述，如果发现一直无法识别烧录器则需要换一个烧录器或者换一种烧录方式；

3. 烧录

"主界面" --> "保存" --> "Rebuild" --> "Download" 即可

![F407shaolumulu](/public/image/Embedded/MCU/GD32/GD32F407/F407shaolumulu.png)

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

![F407DAPpezhi01](/public/image/Embedded/MCU/GD32/GD32F407/F407DAPpezhi01.png)

3. 开始进入升级模式。首先按住BOOT0不要松手，然后再按一次RESET进入到升级模式。

进入升级模式成功之后，会在软件中显示设备 **GD DFU DEVICE 1** 。如果没有进入请多次尝试。

![F407DAPpeizhi02](/public/image/Embedded/MCU/GD32/GD32F407/F407DAPpeizhi02.png)

4. 点击 "Connect" 连接开发板。

连接成功之后，会显示出芯片内存大小。

![F407DAPpeizhi03](/public/image/Embedded/MCU/GD32/GD32F407/F407DAPpeizhi03.png)

5. 查看有无读写保护

主界面中选择 "Erase selected pages" ，点击 "Erase"，如果有读写保护则需要解除保护如果没有就不管他

6. 下载测试代码

在主界面，点击 "Browse" ,选择 .hex 文件，之后点击 "Download"；

然后我们按一下复位按键，就可以让程序开始运行。

6. 解除写保护

在主界面，点击 "Edit Option Bytes";

主要内容为修改 SPC 的值为非 0XAA 和非 0XCC 值，比如 0x55

![F407DAPpeizhi04](/public/image/Embedded/MCU/GD32/GD32F407/F407DAPpeizhi04.png)

> 但我测试时候发现，将 WP0 下面的两个沟去掉也可以接触写保护;

## 调整系统主频

芯片选型手册

![F407xinpianxuanxingbiao](/public/image/Embedded/MCU/GD32/GD32F407/F407xinpianxuanxingbiao.png)

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

> 根据手册我们要设置系统主频为 168 MHz，那么要如何实现这个 168MHz 的频率，这时候就要去看板子商家的原理图和用户手册了，看他们是如何实现的，然后选择为对应的方式，如果他们设置了外部就选择外部否则可以选择内部的（通常情况下一般都是默认选择外部高速晶振）；

![F407jinzhenbufen](/public/image/Embedded/MCU/GD32/GD32F407/F407jinzhenbufen.png)

这里我们先简单走一遍 ARM32 系列设置系统主频的流程：

上电复位后他会去执行 Reset_Handler 这个启动代码，它负责在系统复位（Reset）后初始化处理器状态和程序运行环境；同时在这个代码中他会去调用 SystemInit这个初始化函数；

在 SystemInit 这个初始化函数中，他会进行一系列的初始化配置，这里我们只关心它对于系统时钟的配置也就是调用的 SetSysClock 这个函数；

在 SetSysClock 这个函数中他打开了了外部时钟，并设置了锁相环等一些列操作；这里我们关心的是如何将 8MHz 外部高速晶振设置为 168MHz 的系统主频，也就是如何通过锁相环将 8MHz 设置为 168MHz；

```c
// 锁相环配置如下
RCC->PLLCFGR = PLL_M | (PLL_N << 6) | (((PLL_P >> 1) -1) << 16) |
                   (RCC_PLLCFGR_PLLSRC_HSE) | (PLL_Q << 24);
```

在上面的锁相环配置中，我们可以看到，它是通过先除以 PLL_M 乘以 PLL_N 在除以 PLL_P 最终得到一个系统主频的，所以我们只需要修改这三个常量的值就可以了 ；

也就是将 PLL_M 设置为 8，PLL_N 设置为 336，PLL_P 设置为 2 即可；当然你也可以有其他的组合只要得到的是 168MHz 就可以；

> 不理解记住就可以，因为只要是 Cortex-M 系列的他的锁相环基本上都是这样配置的

此时系统主频就被配置为了 168MHz

## 配置 SysTick

设置完系统主频后，我们还需要设置下滴答时钟以得到一个精准的延时；SysTick 的具体原理可以参考 [SysTick]() 这一篇文章，这里只教大家如何去配置；

我们找到 main() 函数中系统滴答时钟的配置语句，然后跳转后找到 SysTick_Config 这个函数，这里面就是对于系统滴答时钟的配置，我们通常使用默认配置不去动他，只需要给他一个具体的周期值即可；

```c
// STM32
RCC_GetClocksFreq(&RCC_Clocks);
SysTick_Config(RCC_Clocks.HCLK_Frequency / 100);
// GD32
SysTick_Config(SystemCoreClock / 1000000U);
```

通过上面语句可以发现在 Cortex-M 架构中，我们只需要设置 SysTick_Config 这个函数的参数即可，在 GD32 中 SystemCoreClock 是调用的一个宏我们只需要修改他为 168000000（系统主频）即可；在 SMT32 中稍微麻烦一点我们要去看他的默认配置也就是 RCC_GetClocksFreq(&RCC_Clocks) 这个函数;

```c
// GD32 中的修改
#define __SYSTEM_CLOCK_168M_PLL_8M_HXTAL        (uint32_t)(168000000)
// STM32 中会通过寄存器去配置所以通常情况下我们不需要去修改它，只需要修改如下即可：
// (我感觉 STM32 可以不需要配置，它会自动去选择时钟源，通过时钟源作为系统滴答时钟进行计算)
// 这一条代码我感觉通常情况下不会用到，除非我们像 GD32 那样去使用；
uint32_t SystemCoreClock = 168000000;
```

> 一般情况下我们将 GD32 中 SysTick 的代码复制到 STM32 中去使用，因为更简单些

## 问题

一、**外部的 8MHz 是如何实现 168MHz 的？**
