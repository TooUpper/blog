---
title: "HID协议"
date: 2024-09-19 11:31:00
categories: "嵌入式"
tags: 
- "STC8H"
---

HID（Human Interface Device）协议是一种用于**连接人机交互设备（如键盘、鼠标、游戏控制器等）到计算机**或其他主机设备的通信协议。它是 **USB（通用串行总线）设备的一部分**，专门用于**处理和管理人机交互设备的数据传输**。

**HID 协议概述**

HID 协议最初是为 USB 而设计的，但随着技术的发展，它也被应用在其他通信方式上，例如 **蓝牙**。HID 设备以一种标准化的方式报告其数据，使得主机可以理解设备的输入和输出，无需依赖具体的硬件驱动。HID 的典型设备包括：

- 键盘、鼠标、触控板、游戏手柄、条码扫描仪、远程控制设备

**HID 设备的工作原理**

HID 协议定义了**如何描述设备的功能**以及**如何传输数据**。通过使用 HID 协议，**设备和主机之间可以在无需特定驱动程序的情况下通信**。

HID 设备通过 **描述符（Descriptor）** 向主机传达其自身信息。描述符是一个**数据结构**（例如数组），详细说明了设备的输入/输出功能、设备的类型、报告格式等。主机通过解析描述符，可以知道如何与设备交互和处理其数据。

**通信流程**

USB HID通讯时序可以大致分为以下几个步骤：

1. 设备连接和初始化：设备被插入USB端口后，会进行初始化和配置，包括分配USB地址和设置通信端点等。
2. 主机发送设备描述符：主机会向设备发送请求，要求设备提供自己的描述符信息，包括设备类型、厂商信息、设备功能等。
3. 设备响应描述符请求：设备接收到主机的请求后，会根据请求提供相应的设备描述符信息，包括设备类型、厂商信息、设备功能等。
4. 主机发送配置描述符：主机会向设备发送请求，要求设备提供自己的配置描述符信息，包括端点数量、数据传输速率、电源需求等。
5. 设备响应配置描述符请求：设备接收到主机的请求后，会根据请求提供相应的配置描述符信息，包括端点数量、数据传输速率、电源需求等。
6. 主机发送数据：主机会向设备发送数据包，数据包中包含了控制信息和数据内容。
7. 设备接收和处理数据：设备接收到主机发送的数据包后，会进行处理和响应，包括识别控制信息和处理数据内容。
8. 设备发送数据：设备会向主机发送数据包，数据包中包含了控制信息和数据内容。
9. 主机接收和处理数据：主机接收到设备发送的数据包后，会进行处理和响应，包括识别控制信息和处理数据内容。
10. 完成通讯：通讯完成后，设备和主机会进行断开连接和资源释放等操作。

需要注意的是，USB HID通讯过程中的具体时序和流程可能会因为具体的应用场景和设备而有所不同，上述步骤仅供参考。

![HIDtongxinliucheng](/public/image/嵌入式/MCU/STC8H/HIDtongxinliucheng.png)

**通信方式：中断传输**

**中断传输（Interrupt Transfer）** 是 HID 协议中的核心通信方式。主机会定期轮询 HID 设备，询问其是否有新的输入数据。这种传输方式确保主机能够及时接收到输入数据（例如鼠标移动、按键输入等）。

当有数据变化时，设备通过中断传输将输入报告发送给主机，主机根据报告内容执行相应的操作，如在屏幕上移动鼠标光标、显示键盘输入的字符等。

> 也就是说，当我通过单片机按键输入 Q 之后必须也要判断按键弹起不然他就会一直输出 Q

## STC8H

**学习官方 USB HID 范例**

1. 下载[8H试验箱](https://www.stcaimcu.com/forum.php?mod=attachment&aid=ODg3Mnw1NTVmMzc3NXwxNzEzNDMzODM4fDF8MTUyNQ==)中的代码（60-HID(Human Interface Device)协议范例）使用其中的示例编译并烧录到开发板中；

2. 将开发板的开关拨动到 HID
3. 打开”设置“ -> 搜索”蓝牙和其他设备设置“并点击 -> 查看 ”输入“中的设备

![HIDshili](/public/image/嵌入式/MCU/STC8H/HIDshili.png)

> 如果提示 ”其他设备“ USB-SERIAL CH340(COM3) 则说明没有将开关拨动到 HID
>
> 如果按照上述步骤执行，但是不显示”STC UST Keyboard“则可能需要多烧录几次（重复1、2、3步骤）

官方示例的作用，就是帮助我们构建了一个HID设备，将设备注册到了PC机中。

**文件说明**

- usb.c 和 usb.h：  USB入口文件，提供USB所有功能，设备信息，配置信息，通讯过程等
- usb_req_std.c 和 usb_req_std.h：设备信息和配置信息通讯过程中的逻辑实现。
- usb_req_class.c 和 usb_req_class.h：通讯过程中的逻辑实现
- usb_vendor.c 和 usb_vendor.h：初始化配置逻辑
- usb_desc.c 和 usb_desc.h: 协议描述信息，内部是协议的常量信息。

**修改 PC 端的显示**

修改 usb_desc.c 中间的内容即可：

1. MANUFACTDESC 制造商信息。

```c
// 原代码
char code MANUFACTDESC[8] =
{
    0x08,0x03,
    'S',0,
    'T',0,
    'C',0,
    // ...
};
===================================
// 修改后代码
char code MANUFACTDESC[18] =
{
    0x12,0x03,
    'T',0, // 'T' 的 Unicode 表示：0x0054
    'O',0, // 'O' 的 Unicode 表示：0x004F
    'O',0,
	'U',0,
	'P',0,
	'P',0,
	'E',0,
    'R',0,
    // ...
};    
```

> `0x12` 和 `0x03` 是用于 USB 描述符中的两个特定字节，
>
> `0x12` 是 **18（十进制）**，表示该字符串描述符的总长度为 **18 字节**。字符串描述符的第一个字节通常用来表示整个描述符的长度，包括描述符本身的所有数据。
>
> `0x03` 是 USB 标准中定义的 **描述符类型代码**，表示这是一个 **字符串描述符（String Descriptor）**。

在上面的代码中要注意除了**字符串描述类型代码**之外要按照**小端序**，即**低位在前，高位在后**。

小端序是指**低字节存储在低地址**，高字节存储在高地址。在多字节数据类型（例如 16 位、32 位、64 位）中，小端序总是将数值的**最低有效字节（LSB, Least Significant Byte）**存储在最前面的地址位置，而**最高有效字节（MSB, Most Significant Byte）**放在最后的地址。（以 'T' 为例在 0x0054 中，54 存储在数组中的低位也就是下标为 2 的位置，00 存储在数组的高位也就是下标为 3 的位置）

2. PRODUCTDESC 产品信息。

```c
// 原来的
char code PRODUCTDESC[26] =
{
    0x1a,0x03,
    'S',0,
    'T',0,
    'C',0,
    ' ',0,
    'H',0,
    'I',0,
    'D',0,
    ' ',0,
    'D',0,
    'e',0,
    'm',0,
    'o',0,
};
=============================================
// 修改后
char code PRODUCTDESC[34] =
{
    0x22,0x03,
    'S',0,
    'Z',0,
	' ',0,
	'I',0,
    't',0,
    'h',0,
    'e',0,
	'i',0,
	'm',0,
	'a',0,
	' ',0,
	0x2e, 0x95, 0xd8, 0x76,  //\u952e \u76d8 键盘
	'4',0,
	'1',0,
	'3',0,
};    
```

> 修改后长度发生变化，长度由第一个字符描述决定。
>
> 需要同时修改头文件长度配置信息
>
> 如果是中文，需要把中文转化为ASCII码格式（[ASCII在线转换](https://www.ip138.com/ascii/)），写入其中
>
> 例如：
>
> - `今晚打母驴`对应ASCII码：`\u4eca\u665a\u6253\u6bcd\u9a74`
> - 汉字低位在前，添加`今`为：`0xca`, `0x4e`

**兼容库函数**

由于提供的官方示例中，`stc.h` `STC8H.h` `config.h`，这些文件和我们需要用到的库函数，在变量定义或者是文件命名上，存在重名等冲突，需要进行修改。

将 STC8H 删掉用我们自己的，config.h 中有用到的不能删，就改个名字，stc.h 中有重复定义的删掉就行；

```c
// main.c
#include "STC8G_H_GPIO.h"
#include "STC8G_H_Soft_UART.h"
#include "STC8G_H_Delay.h"
#include "STC8G_H_NVIC.h"
#include "STC8G_H_Switch.h"
#include "MAtrixKey.h"

#include "stc.h"
#include "usb.h"
#include "usb_req_class.h"
#include "timer.h"

void URAT_Config(void) {
    COMx_InitDefine COMx_InitStructure;					
    COMx_InitStructure.UART_Mode = UART_8bit_BRTx;	
    COMx_InitStructure.UART_BRT_Use   = BRT_Timer1;			
    COMx_InitStructure.UART_BaudRate  = 115200ul;			
    COMx_InitStructure.UART_RxEnable  = ENABLE;				
    COMx_InitStructure.BaudRateDouble = DISABLE;			
    UART_Configuration(UART1, &COMx_InitStructure);	

  	NVIC_UART1_Init(ENABLE,Priority_1);	
    UART1_SW(UART1_SW_P30_P31);		
}

void kay_down(u8 row, u8 col) {
    u8 dat[8] = {0}; 
    dat[2] = 0x14 // Q
    usb_class_in(dat);    
}

void kay_up(u8 row, u8 col) {
    u8 dat[8] = {0};
    usb_class_in(dat);
}

void main() {
    P_SW2 |= 0x80;  //扩展寄存器(XFR)访问使能

    EA = 1;
    
    usb_init();
    URAT_Config();
	MK_Init();
    
    while(1) {
        MK_Scan(kay_down, kay_up);
        delay_ms(20);
    }    
}
===================================================
char code CONFIGDESC[41] =
{
    0x07,                   //bLength(7);
    0x05,                   //bDescriptorType(Endpoint);
    0x81,                   //bEndpointAddress(EndPoint1 as IN);
    0x03,                   //bmAttributes(Interrupt);
    0x08,0x00,              //wMaxPacketSize(8);	// 设备向主机传输最大不能超过 8 个字节
    0x0a,                   //bInterval(10ms);

    0x07,                   //bLength(7);
    0x05,                   //bDescriptorType(Endpoint);
    0x01,                   //bEndpointAddress(EndPoint1 as OUT);
    0x03,                   //bmAttributes(Interrupt);
    0x01,0x00,              //wMaxPacketSize(1); // 设备接收主机传输最大不能超过 1 个字节
    0x0a,                   //bInterval(10ms);
    
}    
```

### 问题

一、**既然 通信方式为中断传输 那为什么我在单片机中写代码时要将语句放入 while(1) 循环中呢？**

虽然 HID 协议使用的是 **中断传输**，但在嵌入式系统中，通常还是需要将相关处理代码放在循环中。这是因为中断传输并不意味着单片机会自动处理所有的通信细节，尤其是在嵌入式开发中。具体原因如下：

1. **中断传输与轮询机制的区别**

- **中断传输** 是一种 **USB 通信方式**，它表示**主机**会定期轮询 HID 设备，询问设备是否有新的数据。这是主机端的行为，即在 **USB 层面**，主机会定时查询设备，确保数据能及时获取。
- 然而，在嵌入式系统（如单片机）中，代码逻辑仍需要主动处理数据的接收、发送和状态检查，这通常通过循环来实现。

> 在 USB 或 HID 协议中，中断传输指的是 **主机周期性地轮询设备**，检查设备是否有数据需要传输。

2. **单片机代码中的循环**

- **实时响应与轮询**：单片机的代码中，放入循环的目的是为了轮询设备的状态。例如，单片机在等待新数据或判断是否有新的中断信号产生时，需要在循环中反复检查某些状态寄存器或标志位。这样，当有数据到达或状态变化时，代码可以及时做出反应。
- **中断并不是自动执行**：中断传输并不意味着所有的数据传输都是自动完成的。中断传输指的是 USB 主机会定期轮询设备，但单片机仍然需要执行具体的处理代码，例如读写数据、处理中断信号等，这些都需要在主循环中完成。

虽然 HID 协议使用中断传输方式，主机（如 PC）会定期轮询 HID 设备，但在嵌入式开发中，单片机的程序仍然需要通过循环来实现持续的任务调度、轮询状态和处理数据。循环确保代码可以实时检查设备的状态，及时响应中断请求。
