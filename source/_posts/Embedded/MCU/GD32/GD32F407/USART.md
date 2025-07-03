---
title: "USAET"
date: 2024-10-29 16:09:00
categories: "ARM"
tags: 
- "GD32F407"
---

USART（Universal Synchronous/Asynchronous Receiver Transmitter）**通用同步/异步收发器**是一种在嵌入式系统中广泛应用的**串行通信协议**。它支持**同步**和**异步**两种传输模式，其中同步模式在**时钟信号**的帮助下能够精确同步数据传输，而异步模式则不需要额外的时钟信号支持。同时它还支持多种通信模式，包括**全双工**、**半双工**和**单线模式**，并能够通过简单配置在同步和异步传输之间切换。

GD32F407 中 USART 协议的详细介绍：

**传输模式**

- **异步模式**：与 UART 类似，不需要时钟信号，仅使用预设的波特率进行数据传输。

- **同步模式**：依赖时钟信号进行数据同步传输，通常用于需要更高传输速率和更精确同步的场景。

**USART 通信的数据帧结构**

在异步模式下，USART 通信的数据帧结构通常包括以下几个部分：

- **起始位**：1 位，通常为低电平（逻辑 0），用于通知接收端即将开始传输数据。
- **数据位**：8-9 位，可配置，表示实际传输的数据内容（一般配置为 8 位）。
- **校验位**（可选）：1 位，用于错误检测（奇校验或偶校验）。
- **停止位**：产生0.5，1，1.5或者2个停止位，表示数据帧结束，通常为高电平（逻辑 1）。

例如，一个 8N1 格式的数据帧包括 1 位起始位、8 位数据、1 位停止位，没有校验位。

在同步模式下，由于时钟信号的同步作用，数据帧可以更加简化，不再需要起始位和停止位的区分。

**工作模式**

USART 支持多种模式：

- **全双工模式**：发送端和接收端独立工作，同时收发数据。
- **半双工模式**：发送和接收共享同一条数据线，无法同时进行收发。
- **半双工单线模式**：只有一条单独的数据线，用于资源受限的场景。
- **多处理器模式**：适合主机与多个从机的场景。主机广播地址，选中的从机响应以避免混乱。

**波特率控制**

- 波特率（Baud Rate）定义了数据传输速率（每秒传输的比特数），例如 9600、115200。**发送端和接收端的波特率必须一致才能正确传输和解析数据**。USART 硬件中通常有专门的波特率寄存器来设置传输速率。

**错误检测机制**

USART 协议包含多种错误检测机制，用于确保数据的可靠性：

- **帧错误（FERR）**：当接收的数据帧的停止位不是高电平时，标记为帧错误。这通常表示起始位检测错误，或者信号受到干扰。
- **过载错误（ORERR）**：当接收方的接收缓冲区满，新的数据到达时会产生过载错误。
- **奇偶校验错误（PERR）**：如果启用奇偶校验功能，当接收的数据位和校验位不匹配时，标记为奇偶校验错误。
- **噪声检测（NERR）**：USART 会检测数据线上的噪声干扰，确保接收到的数据是稳定的。

**USART 同步模式（时钟）**

在同步模式下，USART 会在 TX（发送）和 RX（接收）线上加上一条时钟线 SCLK（串行时钟）。数据会根据时钟上升沿或下降沿进行采样和发送，使发送方和接收方时刻保持同步。

- **主从配置**：通常，发送方为主设备，提供时钟信号；接收方为从设备，接收并使用主设备的时钟信号。
- **时钟极性（CPOL）** 和 **相位（CPHA）**：USART 在同步模式下，可以设置时钟极性和相位，以控制数据位相对于时钟边沿的位置。

**中断机制**

USART 支持多种中断，以便微控制器处理不同的事件。例如：

- **接收中断**：接收到一帧数据时触发中断，应用程序可以在中断处理程序中读取数据。
- **发送完成中断**：数据帧发送完成时触发中断，应用程序可以在中断处理程序中进行相应操作（比如发送下一个数据帧）。
- **错误中断**：当发生错误（例如帧错误、奇偶校验错误）时，USART 会触发错误中断。

USART 是一种灵活的串行通信协议，既可以在异步模式下工作，简化电路复杂度，又可以在同步模式下工作，提高数据传输速度和稳定性。它的多种错误检测机制、中断机制和可配置的帧格式使其非常适合嵌入式系统的可靠通信需求。在使用 USART 时，需要合理配置波特率、数据帧格式、校验机制等，确保通信的可靠性和准确性。

## 引脚说明

**USART 引脚描述**

![F407uartyinjiaoshuom](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407uartyinjiaoshuom.png)

> 硬件流控制功能通过 nCTS 和 nRTS 引脚来实现。通过将 USART_CTL2 寄存器中 RTSEN 位置 1 来使能 RTS 流控，将 USART_CTL2 寄存器中 CTSEN 位置 1 来使能 CTS 流控。
>
> **nCTS：**USART 发送器监视 nCTS 输入引脚来决定数据帧是否可以发送。如果 USART_STAT0 寄存器中 TBE 位是 0 且 nCTS 为低电平，发送器发送数据帧。在发送期间，若 nCTS 信号变为高电平，发 送器将会在当前数据帧发送完成后停止发送。
>
> **nRTS：**USART接收器输出 nRTS，它用于反映接收缓冲区状态。当一帧数据接收完成，nRTS变成高电 平，这样是为了阻止发送器继续发送下一帧数据。当接收缓冲区满时，nRTS保持高电平，可 以通过读USART_DATA寄存器来清零。

**引脚连线**

对于芯片和 PC 机之间的通信，则不能直接相连，因为电平不兼容。所以要中间要接一个RS232的转换器，将TTL电平转换为RS232电平

![USARTyinjiaolianxian](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/USARTyinjiaolianxian.png)

## USART 内部框图

![USARTneibukuangtu](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/USARTneibukuangtu.png)

> **SW_RX：**数据接收引脚，内部引脚，只用于单线和智能卡模式。

## USART 数据帧

USART 数据帧开始于起始位，结束于停止位。USART_CTL0 寄存器中 WL 位可以设置数据长度。 将 USART_CTL0 寄存器中 PCEN 置位，最后一个数据位可以用作校验位。若 WL 位为 0，第七位 为校验位。若WL位置 1，第八位为校验位。USART_CTL0 寄存器中 PM 位用于选择校验位的计 算方法。

![USARTshujuzhen](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/USARTshujuzhen.png)

根据图片中的数据传输示例，可以看到数据从 bit0 开始发送，依次到 bit7。这意味着数据是从低位（Least Significant Bit, LSB）开始传输的。

> **空闲帧**：在数据帧之间的空闲状态，没有数据传输。（局域网协议使用，串口不需要）
>
> **断开帧**：在特定情况下用于表示连接断开的状态。（局域网协议使用，串口不需要）

**GD32 中 USART 的发送流程实现**

1. 在 USART_CTL0 寄存器中置位 UEN位，使能 USART； 
2. 通过 USART_CTL0 寄存器的WL设置字长； 
3. 在 USART_CTL1 寄存器中写 STB[1:0] 位来设置停止位的长度； 
4. 如果选择了多级缓存通信方式，应该在 USART_CTL2 寄存器中使能 DMA（DENT位）；
5. 在 USART_BAUD 寄存器中设置波特率；
6. 在 USART_CTL0 寄存器中设置 TEN 位；
7. 等待 TBE 置位；
8.  向 USART_DATA 寄存器写数据；
9. 若 DMA 未使能，每发送一个字节都需重复步骤 7-8； 
10.  等待 TC = 1，发送完成

**GD32 中 USART 的接收流程实现**

上电后，USART接收器使能按以下步骤进行： 

1. 在USART_CTL0寄存器中置位UEN位，使能USART； 
2. 写USART_CTL0寄存器的WL去设置字长；
3. 在USART_CTL1寄存器中写STB[1:0]位来设置停止位的长度；
4. 如果选择了多级缓存通信方式，应该在USART_CTL2寄存器中使能DMA（DENR位）；
5. 在USART_BAUD寄存器中设置波特率；
6. 在USART_CTL0中设置REN位。 

接收器在使能后若检测到一个有效的起始脉冲便开始接收码流。在接收一个数据帧的过程中会检测噪声错误，奇偶校验错误，帧错误和过载错误。 

当接收到一个数据帧，USART_STAT0 寄存器中的 RBNE 置位，如果设置了 USART_CTL0 寄存器中相应的中断使能位 RBNEIE，将会产生中断。在 USART_STAT0 寄存器中可以观察接收状态标志。

在接收过程中，需使能 REN 位，不然当前的数据帧将会丢失；

**发送数据的两种情况**

正在发送过程中的情况：

- 当你尝试向 USART_DATA 寄存器写入数据时，如果此时已有数据正在发送，那么新写入的数据会被存储在发送缓冲区中。
- 当当前的发送过程完成后，缓冲区中的数据会依次传输到发送移位寄存器中，继续进行发送。

没有发送过程的情况：

- 如果在写入 USART_DATA 时，没有数据正在发送（即 TBE 位，表示发送缓冲区空，Transmission Buffer Empty，已经被清空），那么新的数据将立即被传输到发送移位寄存器中，准备进行发送。
- 在这种情况下，TBE 位会首先被清零（表示缓冲区当前不为空），然后迅速设置为 1，表明新的数据可以进行发送。

在 GD32 系列的微控制器中，**接收缓冲区**和**发送缓冲区**通常是指 USART 外设中用于接收和发送数据的硬件寄存器数（USART_DATA）;

## 具体实现

**串口通信过程**

![F407chuankoutongxinguoc](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407chuankoutongxinguoc.png)

**代码实现**

```c
#include "gd32f4xx.h"
#include "systick.h"
#include <stdio.h>
#include "main.h"

int8_t dataBuf[1024] = {0};
int8_t index = 0;

// GPIO 配置
void GPIO_USART_Config(void) {
    // 启用GPIOA的外设时钟。
    rcu_periph_clock_enable(RCU_GPIOA);
    // 配置引脚的工作模式为复用模式
    gpio_mode_set(GPIOA, GPIO_MODE_AF, GPIO_PUPD_NONE, GPIO_PIN_9);
    // 配置为特定的功能复用模式，指定复用哪一个模式
    gpio_af_set(GPIOA, GPIO_AF_7, GPIO_PIN_9);
    // 配置GPIOA引脚的输出选项，Rx是输出引脚所以配了个推挽
    gpio_output_options_set(GPIOA, GPIO_OTYPE_PP, GPIO_OSPEED_50MHZ, GPIO_PIN_9);
	
	// PA10配置 A10 是输入所以不需要配置输出
	rcu_periph_clock_enable(RCU_GPIOA);
	gpio_mode_set(GPIOA, GPIO_MODE_AF, GPIO_PUPD_PULLUP, GPIO_PIN_10);
	gpio_af_set(GPIOA, GPIO_AF_7, GPIO_PIN_10);
}

// UART 配置  
// 我们在串口通信时候双方需要指定 数据位个数、校验位、停止位
// 以保证双方通信的正确，所以可以不像官方示例中那样写    
void USART_Config(void) {
    // 启用USART0的外设时钟。
    rcu_periph_clock_enable(RCU_USART0);
	usart_deinit(USART0);
	/*=================USART属性配置=================*/
    // 设置波特率
    usart_baudrate_set(USART0, 115200);
    // 设置数据位个数、校验位、停止位
    usart_word_length_set(USART0, USART_WL_8BIT);
    usart_parity_config(USART0, USART_PM_NONE);
    usart_stop_bit_set(USART0, USART_STB_1BIT);
	/*=================USART发送接收使能配置=================*/
    // 开启发送使能
    usart_transmit_config(USART0, USART_TRANSMIT_ENABLE);
	// 开启接收使能
	usart_receive_config(USART0, USART_RECEIVE_ENABLE);
	/*=================USART接收中断配置=================*/
	// 开启中断使能
	nvic_irq_enable(USART0_IRQn, 2, 2);
	// 配置两个中断源，也就是RBNE和IDLE发生改变了就触发中断
	usart_interrupt_enable(USART0, USART_INT_RBNE);
	usart_interrupt_enable(USART0, USART_INT_IDLE);
    // 开启UART0使能
   usart_enable(USART0);
}

/*重定向C标准库的printf函数，使其通过USART（通用同步异步收发器）进行输出。*/
int fputc(int ch, FILE *f) {
    usart_data_transmit(USART0, (uint8_t)ch);
    while(RESET == usart_flag_get(USART0, USART_FLAG_TBE));
    return ch;
}

void send_data(uint8_t dat) {
    usart_data_transmit(USART0, dat);
    while(RESET == usart_flag_get(USART0, USART_FLAG_TBE));
}

// UART0 中断处理函数
void USART0_IRQHandler(void) {
	// 有数据了
	if(SET == usart_interrupt_flag_get(USART0, USART_INT_FLAG_RBNE)) {
		usart_interrupt_flag_clear(USART0, USART_INT_FLAG_RBNE);		
		int8_t data =  usart_data_receive(USART0);		
		dataBuf[index] = data;
		index++;
	}
	
	if(SET == usart_interrupt_flag_get(USART0, USART_INT_FLAG_IDLE)) {
        // 清除 LDLE 中断标志位
        // 软件先读USART_STAT0，再读USART_DATA可清除该位。
        // usart_interrupt_flag_get 读取的就是 USART_STAT0 寄存器中的标志位
        // USART_STAT0 是状态寄存器，USART_DATA 为数据寄存器
		usart_data_receive(USART0); // 读数据寄存器，以清除 LDLE 中断标志位
		// 使其成为一个字符串
		dataBuf[index] = '\0';
        
		printf("rece_data = %s\n", dataBuf);
        
		index = 0;
	}	
}

int main(void) {
    systick_config();
    GPIO_USART_Config();
    USART_Config();
    printf("Hi,Key");
    while(1){		
	}
}    
```

> 要使用 printf 串口打印功能需要使用嵌入式的 C 库而且不是 stdio.h；
>
> "魔术棒" --> "Target" 下勾选 Use MicroLIB；
>
> **MicroLIB** 是指使用一个轻量级的 C 标准库，专门为资源受限的嵌入式系统设计。它会将 printf 输出重新定向到串口而不是控制台；

**实现步骤**

1. **GPIO 配置**

- 启用指定GPIO端口的外设时钟

- 查看板子的原理图，了解有无外部上拉/下拉电阻，并配置为复用模式
- 在复用为 USART 时可以查看下数据手册以防 Rx 和 Tx 引脚搞错了

![UARTPA9P10](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/UARTPA9P10.png)

> **Default：**引脚的默认功能或连接，表示这是一个通用输入输出（GPIO）引脚。 
>
> **Alternate：**表示该引脚可以被配置为不同的外设功能。(复用)
>
> **Additional：**表示引脚的额外或可选功能。

- 要指定是那一个复用模式，需要在数据手册中查看**端口复用功能摘要表**

![F407PA910duankoufuyongbiao](/public/image/嵌入式/MCU/ARM/Cortex-M4/GD32/GD32F407/F407PA910duankoufuyongbiao.png)

2. **USART 配置**

- 启用指定 USART 端口的外设时钟（在数据手册中可以看到引脚所属 USART 端口）
- 初始化设置端口（可选）
- 配置 USART 的波特率
- 数据位、校验位、停止位（可选）

- 设置发送使能
- 设置接收使能
- 设置中断使能
- 配置中断触发源
- 编写中断处理函数
  - 非必要尽量不要再中断处理函数中进行延时操作，除非确认没有问题
  - 在读取标志位进行判断之后如果需要要将标志位置 0 或者置 1（查看用户手册了解是硬件置位还是软件置位）
- 设置 USART 端口使能

> USART 的数据接收一般都放在中断中，所以需要启用 USART 端口的中断使能
>
> 中断使能后需要指定如何触发中断也就是配置中断源，这个要去查看对应 USART 端口的用户手册，根据我们需要的功能找到需要的标志位，然后查看该标志位的中断触发位将其值 1 / 0；（了解后去找对应的库函数实现）

## 问题

一、**GPIO 输出默认低电平，USART输出引脚则要高电平拉低在作为发送标志，这个引脚是什么时候被拉低的**

当引脚被复用为 **USART 的 TX（发送）引脚**时，默认情况下它的电平状态会由 USART 模块控制，而不是由内部上拉电阻控制。

**具体情况：**

1. **默认高电平**：USART 的 TX 引脚在空闲状态下通常为高电平（逻辑“1”）。这是 USART 通信协议的要求，当没有数据发送时，TX 线会保持在高电平。
2. **发送数据时**：当 USART 开始发送数据时，TX 引脚会根据数据位的状态（0 或 1）改变电平，以形成串行通信的信号。

因此，将引脚复用为 USART 的 TX 时，即使之前配置了内部上拉电阻，它在空闲状态下也会保持为高电平。上拉电阻对 TX 引脚的电平没有影响，因为此时**引脚的控制权由 GPIO 移交给了 USART**。
