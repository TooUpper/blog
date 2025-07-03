---
title: "DMA"
date: 2024-11-2 16:38:00
categories: "ARM"
tags: 
- "GD32F407"
---

**DMA（Direct Memory Access，直接内存访问）**是一种数据传输机制，它允许嵌入式系统中的外设在无需CPU干预的情况下，直接与内存之间进行数据交换。

## DMA 传输方式

DMA的作用就是实现数据的直接传输，而去掉了传统数据传输需要CPU寄存器参与的环节，主要涉及四种情况的数据传输，但本质上是一样的，都是从内存的某一区域传输到内存的另一区域（外设的数据寄存器本质上就是内存的一个存储单元）。四种情况的数据传输如下：

**内存到内存（Memory-to-Memory）**：数据在内存之间传输。

**外设到内存（Peripheral-to-Memory）**：将外设数据写入内存。

**内存到外设（Memory-to-Peripheral）**：将内存数据写入外设。

**外设到外设（Peripheral-to-Peripheral）**：将外设数据写入外设。

> 外设到外设的方式可以理解成内存到内存，因为 DMA 的传输模式本身其实并不存在外设到外设这种传输方式；

### DMA 三种传输模式的数据流

![F407DMAsangechuanshuliu.png](/public/image/Embedded/MCU/GD32/GD32F407/F407DMAsangechuanshuliu.png)

- 外设到存储器：通过 AHB 外设主机接口从外设读取数据，通过 AHB 存储器主机接口向存 储器写入数据；
- 存储器到外设：通过 AHB 存储器主机接口从存储器读取数据，通过 AHB 外设主机接口向 外设写入数据；
- 存储器到存储器：通过 AHB 外设主机接口从存储器读取数据，通过 AHB 存储器主机接口 向存储器写入数据。

### 外设握手

为了保证数据的有效传输，DMA 控制器中引入了外设和存储器的握手机制，包括请求信号和应 答信号：

- **请求信号：**由外设发出，表明外设已经准备好发送或接收数据；
- **应答信号：**由 DMA 控制器响应，表明 DMA 控制器已经发送 AHB 命令去访问外设。

**握手机制**

![F407DMAwoshoujizhi](/public/image/Embedded/MCU/GD32/GD32F407/F407DMAwoshoujizhi.png)

## 传输模式

支持单数据传输和多数据传输模式：

- **多数据传输模式：**在存储器数据宽度和外设数据宽度不同的时候，自动打包/解包数据；
- **单数据传输模式：**当且仅当 FIFO 空的时候从源地址读取数据，存进 FIFO，然后把 FIFO 的数据写到目标地址。

**FIFO**

DMA 控制器的每个通道都有一个 4 字深度的 FIFO 用于缓冲数据，从源地址读取的数据会先暂时保存在 FIFO 中，再传输到目的地址。根据 FIFO 的配置，DMA 控制器支持两种数据处理模式： 单数据传输模式和多数据传输模式。**在存储器到存储器模式下，DMA 控制器仅支持多数据传输 模式。**

## DMA 参数

在使用DMA时，通常需要配置以下主要参数：

- **direction（传输方向）**

  - 定义 DMA 数据传输的方向，有以下几种选项：
    - DMA_PERIPH_TO_MEMORY: 从外设传输到内存。
    - DMA_MEMORY_TO_PERIPH: 从内存传输到外设。
    - DMA_MEMORY_TO_MEMORY: 从内存传输到内存（一些 DMA 控制器可能不支持）。

  **periph_addr（外设地址）**

  - 设置 DMA 读取数据的源地址，通常是外设的寄存器地址（如 USART 的数据寄存器地址）。这个地址通常是固定的，不会变化。

  **memory0_addr（内存地址）**

  - 设置 DMA 将数据传输到的内存地址，或者在读取模式下 DMA 从此内存地址获取数据传输到外设。这个地址在递增模式下会自动递增。

  **periph_inc（外设地址递增模式）**

  - 设置外设地址是否递增：
    - DMA_PERIPH_INCREASE_ENABLE: 每次传输后外设地址递增。
    - DMA_PERIPH_INCREASE_DISABLE: 外设地址保持不变（常用）。

  **memory_inc（内存地址递增模式）**

  - 设置内存地址是否递增：
    - DMA_MEMORY_INCREASE_ENABLE: 每次传输后内存地址递增。
    - DMA_MEMORY_INCREASE_DISABLE: 内存地址保持不变。

  **periph_memory_width（数据宽度）**

  - 定义传输的数据宽度，有以下几种选择：
    - DMA_PERIPH_WIDTH_8BIT: 8 位传输（字节 Byte, 8位）。
    - DMA_PERIPH_WIDTH_16BIT: 16 位传输（半字 Half-word, 16位）。
    - DMA_PERIPH_WIDTH_32BIT: 32 位传输（字 Word, 32位）。

  **circular_mode（循环模式）**

  - 定义 DMA 传输是否启用循环模式：
    - DMA_CIRCULAR_MODE_ENABLE: 启用循环模式，在传输完指定数据量后自动重新开始。
    - DMA_CIRCULAR_MODE_DISABLE: 禁用循环模式，传输一次完成后停止。

  **priority（优先级）**

  - 设置 DMA 通道的优先级：
    - DMA_PRIORITY_LOW: 低优先级。
    - DMA_PRIORITY_MEDIUM: 中等优先级。
    - DMA_PRIORITY_HIGH: 高优先级。
    - DMA_PRIORITY_VERY_HIGH: 最高优先级。

  **number（传输数据量）**

  - 设置 DMA 传输的数据项数量（如传输多少个字节或字）。DMA 会根据这个数量确定何时完成传输。

## DMA 执行流程

![F407DMAkuangtu](/public/image/Embedded/MCU/GD32/GD32F407/F407DMAkuangtu.png)

USART 向 DMA 请求传输数据的完整流程：

1. **USART 外设触发 DMA 请求**

- 在 APB 总线上的 USART 外设在准备好要**发送**或**接收**的数据时，会**向 DMA 控制器发出 DMA 请求信号**。这个请求信号通过总线桥传递到 DMA 控制器。

2. **DMA 控制器接收请求**

- USART 的 DMA 请求到达相应的 DMA 控制器时，该 DMA 控制器会根据预设配置选择合适的 DMA 通道进行数据传输。
- “仲裁器”会根据通道的优先级判断哪个通道的请求优先执行。仲裁器确保当多个通道请求同时发生时，高优先级的请求优先得到处理。

3. **DMA 控制器将数据搬运到内存**

- DMA 控制器根据配置将数据从 USART 的数据寄存器搬运到指定的内存地址。这个过程利用 AHB 总线进行数据传输。

- 如果是“外设到内存”模式，DMA 会从 USART 数据寄存器读取数据并写入到 SRAM 的指定内存地址。

4. **地址和传值自动更新**

- 每次搬运一个字节或字的数据后，DMA 会根据配置选择是否递增内存地址。对于连续数据搬运，通常设置内存地址递增，而外设地址保持不变，因为数据总是从 USART 的同一个数据寄存器读取。
- 传输过程中，外设每传输一次数据，CNT 减 1。如果寄存 器 DMA_CHxCTL 的 CMEN 位或 SBMEN 位置 1，在每次传输完成时，CNT会由硬件自动重新装载。

5. **传输完成或中途抢占**

- DMA 会持续传输，直到传输的字节数达到预设值。如果在传输过程中有更高优先级的 DMA 请求发起（如来自另一个 USART 或其他外设），高优先级请求可以中断当前传输并优先完成。

- 当所有预定数据传输完毕后，DMA 控制器会在相应的通道设置传输完成标志 FTF，并可以触发中断通知 CPU 数据传输完成。

> 以 Hhello 为例，上述流程会执行五次，直到 CNT == 0，将 FTF 位置1；

## DMA 通道映射表

**DMA通道外设映射表**，用于配置 DMA 控制器与外设之间的连接关系

![F407DMAtongdaoyinshe](/public/image/Embedded/MCU/GD32/GD32F407/F407DMAtongdaoyinshe.png)

## 传输控制器

DMA 和外设均可配置为传输控制器：

- DMA 作为传输控制器：可配置数据传输长度，最大为 65535。
- 外设作为传输控制器：数据传输的完成取决于外设的最后一个传输请求。

> 默认情况下都是 DMA 作为传输控制器，可通过 dma_flow_controller_config 函数进行修改

## 实现

功能：通过 DMA 实现 USART 的数据收发

### 代码

```c
// DMA 配置（内存 -> 外设）
void DMA_USART0_Tx_Config(void) {
	// 使能外设DMA1时钟
	rcu_periph_clock_enable(RCU_DMA1);
	// 配置 DMA 的传输模式(多数据传输/单数据传输)
	// 这里选择单数据传输
	dma_single_data_parameter_struct  dma_single_init;
    // 结构体初始化函数
	dma_single_data_para_struct_init(&dma_single_init);

	// 外设地址
	dma_single_init.periph_addr = (uint32_t)(&USART_DATA(USART0));
	// 是否要使能外设地址偏移
	dma_single_init.periph_inc = DMA_PERIPH_INCREASE_DISABLE;
	// 内存地址，发送时候才知道，暂时无法配置	
	// dma_single_init.memory0_addr = 0U;		
	// 使能内存地址偏移
	dma_single_init.memory_inc = DMA_MEMORY_INCREASE_ENABLE;	
	// 数据宽度，即每次搬运的数据大小
	dma_single_init.periph_memory_width = DMA_PERIPH_WIDTH_8BIT;	
	// 是否需要循环执行		
	dma_single_init.circular_mode = DMA_CIRCULAR_MODE_DISABLE;	
	// 数据搬运的方向 内存 -> 外设
	dma_single_init.direction = DMA_MEMORY_TO_PERIPH;	
	// 数据传输的位数，即我们需要发送少多少个（periph_memory_width）大小的数据
    // 这也要等到发送时候才能知道
	// init_struct->number = 0U;
	// 优先级可以默认
	dma_single_init.priority = DMA_PRIORITY_LOW;	
 	// DMA 初始化配置
	dma_single_data_mode_init(DMA1, DMA_CH7, &dma_single_init);
	// DMA 通道功能选择
	dma_channel_subperipheral_select(DMA1, DMA_CH7, DMA_SUBPERI4);
}

// MDA 配置（外设 -> 内存）
void MDA1_USART0_Rx_Config(void) {
    // 清空 DAM1 下 DMA_CH5 通道的配置
	dma_deinit(DMA1, DMA_CH5);
    // 使能DMA1外设时钟
	rcu_periph_clock_enable(RCU_DMA1);
	// 创建DMA配置文件
	dma_single_data_parameter_struct dma_usart0_rx_init;
	// 初始化DMA配置
	dma_single_data_para_struct_init(&dma_usart0_rx_init);
	// 配置数据搬运的方向
	dma_usart0_rx_init.direction = DMA_PERIPH_TO_MEMORY;
    // 配置外设地址
	dma_usart0_rx_init.periph_addr = (uint32_t)(&USART_DATA(USART0));
	// 配置内存地址
	dma_usart0_rx_init.memory0_addr = (uint32_t)dts;
	// 配置内存地址偏移使能
	dma_usart0_rx_init.memory_inc = DMA_MEMORY_INCREASE_ENABLE;
    // 关闭外设地址偏移使能
	dma_usart0_rx_init.periph_inc = DMA_PERIPH_INCREASE_DISABLE;
	// 配置数据发送宽度（每一次发送的数据宽度）
	dma_usart0_rx_init.periph_memory_width = DMA_PERIPH_WIDTH_8BIT;
    // 是否重复执行
	dma_usart0_rx_init.circular_mode = DMA_CIRCULAR_MODE_DISABLE;
	// 配置传输的数据长度
	dma_usart0_rx_init.number = 5;
	// 初始化DMA配置	
	dma_single_data_mode_init(DMA1, DMA_CH5, &dma_usart0_rx_init);
	// 选择DMA1通道5下的具体执行编号
	dma_channel_subperipheral_select(DMA1, DMA_CH5, DMA_SUBPERI4);
	// 使能DMA1
	dma_channel_enable(DMA1, DMA_CH5);	
}
=========================================================================
// 如果使用 USART，那么在 USART 配置文件中中也要开启 DMA 使能
/*=================DMA配置配置=================*/
// 启用 USART 的 DMA 发送功能    
usart_dma_transmit_config(USART0, USART_TRANSMIT_DMA_ENABLE);
// 启用 USART 的 DMA 接收功能  
usart_dma_receive_config(USART0, USART_RECEIVE_DMA_ENABLE); 

==========================================================================
// UART0 中断处理函数
void USART0_IRQHandler(void) {	
	if(SET == usart_interrupt_flag_get(USART0, USART_INT_FLAG_IDLE)) {
		usart_data_receive(USART0);
		// LDLE 串口空中断时先关闭 DMA 使能，方便清除标志位后开启
		dma_channel_disable(DMA1, DMA_CH5);
		printf("rece_data = %c, %c, %c, %c, %c\n", dts[0], dts[1], dts[2], dts[3], dts[4]);
		// 清除 TFT 标志位(置0)，当传输完成后它会自动置 1
        // 所以此处并不需要进行阻塞判断
		dma_flag_clear(DMA1, DMA_CH5, DMA_FLAG_FTF);
		// 重新使能 DMA 标志位
		dma_channel_enable(DMA1, DMA_CH5);
	}	
} 

void send_data(uint8_t dat) {
    // 配置内存数据地址
	dma_memory_address_config(DMA1, DMA_CH7, DMA_MEMORY_0, (uint32_t)(&dat));
    // 配置数据传输的位数(个数)
	dma_transfer_number_config(DMA1, DMA_CH7, 1);
    // 使能 DMA1
	dma_channel_enable(DMA1, DMA_CH7);
	// 发送完后 FTF 会置 1
    // 为了保证发送完成，此处需要阻塞进行等待
	while(RESET == dma_flag_get(DMA1, DMA_CH7, DMA_FLAG_FTF));
	// 清除中断标志位
	dma_flag_clear(DMA1, DMA_CH7, DMA_FLAG_FTF);
}
================================================================
// 如果需要启用 DMA 中断，则需要在进行 DMA 配置时添加如下配置    
// 1. 使能 NIVC
nvic_irq_enable(DMA1_Channel0, 2, 2);
// 配置 DMA 中断源
// 使能 DMA 中断
dma_interrupt_enable(DMA1, DMA_CH0, DMA_INT_FTF);
// 最后在使能 DMA
// 2. 编写 NVIC 使能时指定的中断处理函数
void DMA1_Channel0_IRQHandler(void) {
    if(SET == dma_interrupt_flag_get(DMA1, DMA_CH0, DMA_INT_FLAG_FTF)) {
        // 清除标志位，我们需要查看寄存器的标志位说明了解是否需要手动清除
        dma_interrupt_flag_clear(DMA1, DMA_CH0, DMA_INT_FLAG_FTF);
        // 业务逻辑
        for(int i = 0; i < sizeof(dst) / sizeof(dst[0]); i++) {
            printf("%d\n", (int)dst[i]);
        }
    }   
}
```

- 配置 DMA，在相关外设中启用 DMA 的发送过能，（如果需要可开启中断）；
- 在使用中断时候我们通常需要进行标志位判断，这时候我们就需要知道当中断触发时候标志位是 0 还是 1，我们就需要去查看用户手册中当前章节的终端说明或者寄存器中该标志位的说明；

- FTF 标志位在复位时自动置 0，当传输完成后自动置 1，需要我们手动置 0；

### 说明

**重复执行**

当 DMA 完成一轮数据传输后，会自动将目标地址重置回初始地址，并继续进行下一轮的传输。

**usart_dma_transmit_config(USART0, USART_TRANSMIT_DMA_ENABLE);**

这条语句用于启用 USART0 的 DMA 发送功能。它将配置 USART0，使得数据的发送不再通过 CPU 逐个字节传输，而是通过 DMA 控制器来直接将数据从内存传输到 USART 数据寄存器，从而提高数据传输效率，减少 CPU 的负担。

## 问题

一、**DMA 的配置文件重复执行两次后为什么 FTF 标志位的值会从 0 变成 1**

```c
dma_deinit(DMA1, DMA_CH5);
dma_flag_clear(DMA1, DMA_CH7, DMA_FLAG_FTF);
```

上面两行代码可以解决这个问题：

当你 **第一次配置并启动 DMA**，但并没有实际进行数据搬运操作时，FTF（全传输完成标志）会保持为 0，因为没有触发实际的传输操作。但是，当你 **第二次重新配置 DMA** 并执行初始化时，此时不管你有没有使能 DMA；DMA 控制器都会认为之前的传输已经完成，因此它会 **错误地将 `FTF` 置为 1**。

二、**为什么外设 -> 内存需要清除 FTF 标志位，而内存 -> 内存、内存 -> 外设就不需要呢**

问题的关键在于 DMA 配置的时机和操作的顺序，

其实内存 -> 内存、内存 -> 外设也存在这个问题，区别在于，内存 -> 内存、内存 -> 外设二者的发送源要到具体的发送时候才可以确定，也就说我们要到那个时候才会去配置发送源和开启 DMA 使能，那么前面执行的两次初始化配置对于 DMA 来说和 1 次没区别。

三、**当我们通过 USART 向内存写入的数据，超过我们在 PWM 中所定义的个数时，第二次打印的头一个字节会是 PWM 中定义的个数的下一个字节，为什么**

```c
假设，我们定义传输长度为 5

我们第一次传输 hello1，它会显示 hello

我们第二次传输hello,	它会显示1hell

为什么？
```

因为当你外设传递的数据位数，超过你 DMA 配置数据长度时，他会将 DMA 配置数据长度 + 1 位数的字节存在他的缓冲区里面，其他全部丢弃，造成后面的数据就都会错位一个字节；

解决：尽可能定义大的空间把所有数据都接到；

三、**当我们要使用 DMA 功能时候没有头绪要怎么做**

通过用户手册**查看 DMA 关系映射表**：在使用 DMA 时，首先要找到 DMA 与我们要搬运的外设之间的对应关系；

通过用户手册查看 DMA 的关系映射表，找到外设所映射的通道在那个 DMA 中，进行配置即可；

例如，我们要使用 ADC0 时，找到 ADC0 对应的是 DMA1 中通道0的第一个通道 000，那么我们只需要配置 DMA1 并指定通道1，子通道为 000 即可；
