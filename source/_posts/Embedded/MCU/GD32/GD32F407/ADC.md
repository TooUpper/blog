---
title: "ADC"
date: 2024-11-10 11:15:00
categories: "GD32"
tags: 
- "GD32F407"
---

在嵌入式开发中，**ADC**（Analog-to-Digital Converter，模拟-数字转换器）是一种将模拟信号转换为数字信号的硬件组件。它的主要功能是将输入的模拟信号（如温度、压力、电压等）转换为相应的数字信号，使微控制器（MCU）或处理器能够读取和处理这些信息。

ADC采样的频率越高，得到的数字信号就越接近原来的模拟信号，也就是保真度越高，但是需要更多的资源和计算功耗。

![F407ADCguocheng](/public/image/Embedded/MCU/GD32/GD32F407/F407ADCguocheng.png)

## 分类

**逐次逼近型 ADC (SAR-ADC)**

- **工作原理**：逐步逼近输入信号的电压值，每次比较输入信号与参考电压，逐步调整数字输出直到找到最接近的值。
- **特点**：中等速度、高精度、低功耗，适用于常规数据采集。

**Flash ADC**

- **工作原理**：使用多个并行的比较器同时比较输入信号与不同的参考电压级别，直接输出数字结果。
- **特点**：极高速、低分辨率、硬件复杂，适用于实时信号处理。

**Sigma-Delta ADC**

- **工作原理**：通过过采样将模拟信号转换为数字信号，使用噪声整形将高频噪声移至高频区，然后通过数字滤波器还原数字信号。
- **特点**：高分辨率、精度高、速度较慢，适用于高精度测量。

**Pipeline ADC**

- **工作原理**：将转换过程分为多个阶段，每个阶段逐步完成一部分转换，最终合并结果输出。
- **特点**：高速、适中分辨率，适用于视频、通信等高速应用。

**Dual-Slope ADC**

- **工作原理**：首先对输入信号进行积分，然后用已知参考电压再次积分，通过比较两个积分时间来确定输入信号的数字值。
- **特点**：高精度、速度较慢、抗噪声能力强，适用于精密测量。

**并行型 ADC**

- **工作原理**：通过多个并行的比较器同时对输入信号进行比较，快速得出数字结果。
- **特点**：极高速、低分辨率，硬件需求高，适合高速数据采集。

**差分型 ADC**

- **工作原理**：测量两个输入端点之间的电压差，消除共模噪声，提高精度。
- **特点**：抗共模噪声能力强，适用于差分信号的测量。

**异步 ADC**

- **工作原理**：不依赖固定时钟，根据输入信号的变化动态调整采样频率。
- **特点**：灵活，适用于时钟不稳定或需要动态调整采样频率的特殊场合。

**温控型 ADC**

- **工作原理**：通过比较器对输入信号进行测量，采用热计数器方法，最终通过计数来确定数字值。
- **特点**：高精度，适用于精密仪器和特殊测量系统。

## 逐次逼近型工作原理

逐次逼近型 ADC 的工作原理是将模拟信号通过**采样**转换为离散的数字信号，然后再通过**量化**、**编码**等处理，最终得到对应的数字表示。

**采样**

采样（Sampling）是将连续的模拟信号转化为离散的数字信号的过程；ADC 通过以固定周期的脉冲采集模拟型号，这样就可以得到一个在时间上离散的信号。

![F407ADCcaiyang](/public/image/Embedded/MCU/GD32/GD32F407/F407ADCcaiyang.png)

**量化**

虽然采样可以把连续信号变成一系列离散的值，但这些离散的值本身可能是无限精细的（理论上是连续的），计算机无法直接处理这样的无限精度。量化的目的是将这些信号值映射到有限数量的离散数字上，确保信号可以被计算机和数字设备正确处理。

量化是把**采样得到的连续信号值**转换成**离散的数字值**的过程。它将信号值映射到**一定数量的离散级**别上，这些离散的数字值是计算机能理解和处理的。

![F407ADClianghua](/public/image/Embedded/MCU/GD32/GD32F407/F407ADClianghua.png)

**分辨率：**是指数字信号可以表示的**离散级别**的数量，通常由**位数（bit）**来表示。

例如，8位（8-bit）的分辨率表示量化系统能够表示 2<sup>8</sup> = 256 个不同的离散等级，16位（16-bit）则能表示 2<sup>16</sup> =  65536 个不同的等级。

**编码**

将量化后的数字值转化为标准的数字格式的过程，用 0 和 1 组合为每个量化区间编号,代替原来的电压值。

**ADC 转换流程图**

![ADCzhuanhuanliuchengtu](/public/image/Embedded/MCU/STC8H/ADCzhuanhuanliuchengtu.png)

## 运行模式

在 ARM32 的 ADC 模块中，针对单个通道或多个通道的转换，有几种不同的 **运行模式** 来控制采样和转换的行为：

**单次运行模式（Single Conversion Mode）**

- **工作原理**：在单次运行模式下，ADC 每次启动转换时只转换一个通道的数据。每次转换完成后，ADC 停止工作，直到下一次触发才能进行新的转换。

- **触发方式**：可以通过外部触发信号（例如定时器中断）或软件触发启动。

**连续运行模式（Continuous Conversion Mode）**

- **工作原理**：在连续运行模式下，ADC 一旦启动，就会不断地进行转换，转换完成后立刻开始下一次转换，直到停止为止。转换通道可以是单个通道，也可以是多个通道（如果使用扫描模式）。 

- **触发方式**：通常通过软件或外部触发启动，启动后 ADC 将连续进行转换。

**扫描运行模式（Scan Mode）**

- **工作原理：**在扫描模式下，ADC 会按照用户预先设置的顺序，自动依次对多个输入通道进行采样和转换。这些通道可以是同一组或不同组的模拟输入。扫描模式通常与 **连续转换模式** 或 **间断转换模式** 配合使用，这样可以在不干扰其他通道转换的情况下，完成多个通道的采样。

**间断运行模式（Discontinuous Mode）**

- **工作原理**：在间断运行模式下，ADC 会在多个通道的转换之间插入空闲时间，形成一个“间断”采样的过程。与连续模式不同，间断模式在转换时会分批次处理多个通道，每批次完成几个通道的采样，再进行下一批次的转换。
- **触发方式**：通常用于扫描模式下，选择如何将多个通道分批次进行转换。

## 同步模式

当需要有多个 ADC 模块工作时，可以使用 ADC 同步模式。在 ADC 同步模式下，根据 ADC_SYNCCTL 寄存器中 SYNCM[4:0]位所选的模式，转换的启动可以是 ADC0/ADC1/ADC2 的交替触发或同步触发。

![F407ADCtongbumoshi](/public/image/Embedded/MCU/GD32/GD32F407/F407ADCtongbumoshi.png)

> 当 ADC 工作在同步模式，而非独立模式时，如果需要再将 ADC 配置成其他同步模式，则需要**在配置成其他同步模式前，首先将 ADC 配置成独立模式。**

## 常规通道和注入通道

**常规通道**

常规通道是 ADC 的标准通道，主要用于常规的采样和转换。它们通常是逐一进行采样的，即采样一个通道，转换完成后，再切换到下一个通道。

**注入通道**

注入通道是一种更为复杂和灵活的 ADC 通道，它允许在常规转换过程中进行优先级较高的快速采样。插入通道具有更高的优先级，可以在常规转换进行的同时，进行插入通道的转换。

每个注入通道（如 ADC_INJ1, ADC_INJ2, ADC_INJ3, ADC_INJ4）通常有独立的数据寄存器，可以存储不同通道的转换结果。

每个注入通道都对应有一个数据寄存器所以当我们需要同时采集不超过四个，且也不想用 DMA 时我们就可以采用注入通道；当转换完成后我们通过 EOIC 标志位去获取对应通道中的数据；只会在最后一个转换完成后产生一次标志位；

## 数据对齐

 ARM32 的 ADC 数据宽度通常为 12 位（例如 STM32F4 系列），但每个 ADC 数据寄存器通常是 16 位宽。因此，12 位的转换结果会存储在 16 位的寄存器中；所以我们这里需要指定数据的对齐方式。

两种对齐方式：

**左对齐 （Left Align）**：

- 在左对齐模式下，12 位的 ADC 结果会被存储在 16 位的数据寄存器的高 12 位，而低 4 位会被填充为 0。
- 例如，如果 ADC 转换的结果是 0x0ABC（12 位的值），那么存储到寄存器中的数据为 `0xABC0`。

**右对齐（Right Align）**：

- 在右对齐模式下，12 位的 ADC 结果会存储在 16 位数据寄存器的低 12 位，而高 4 位会被填充为 0。
- 例如，如果 ADC 转换的结果是 0x0ABC（12 位的值），那么存储到寄存器中的数据为 `0x0ABC`。

## 模块框图

![F407yihuofeimen](/public/image/Embedded/MCU/STM32/STM32F407/F407ADCcxukuangtu.png)

> 当采样任务少于 4 个时，可以使用注入组，因为注入组的每个通道都有一个独立的寄存器，可以同时进行四个采集任务

**输入端口**

- **V_REF+ / V_REF-**：基准电压输入端，用于确定 ADC 的量程范围。转换的数字结果在 `V_REF-` 对应 0，`V_REF+` 对应最大值（如 4095 在 12 位 ADC）。

- **V_DDA / V_SSA**：供电引脚，用于提供 ADC 模块的工作电源。

- **ADCx_IN0 到 ADCx_IN15**：这些是模拟输入通道。可以连接到外部模拟信号，通过多路复用器（Analog Mux）选择要转换的通道。

**多路复用器（Analog Mux）**

- 多路复用器负责在多个输入通道中选择一个进行转换。此 ADC 模块最多可以支持 16 个通道的输入信号，允许在不同的通道之间切换。

- **GPIO 端口**：ADC 输入通道可以通过 GPIO 配置连接到传感器等外部设备，获得输入的模拟信号。

- 还有内置温度传感器、内部参考电压（V_REFINT）和电池电压监测端（V_BAT）。

**通道分组**

**Injected Channels（注入通道）**：最多可以配置 4 个注入通道。这些通道用于优先级较高的采样任务，一般在特定触发事件时被触发。

**Regular Channels（常规通道）**：最多可以配置 16 个常规通道，用于定期进行采样，转换完成后的数据存储在常规数据寄存器中。

**触发控制**

- **注入通道触发控制**：图中的 JEXTSEL 和 JEXTEN 控制注入通道的触发源，可以选择来自不同定时器（TIMx）的触发信号。

- **常规通道触发控制**：EXTSEL 和 EXTEN 控制常规通道的触发源，也可选择来自不同定时器（TIMx）的触发信号。

- 可以选择的触发源包括多种定时器的输出事件（如 TIM1_CH4, TIM2_TRGO 等），在外部事件发生时启动 ADC 转换。

**ADC 模块**

- 核心的 ADC 模块负责将模拟信号转换为数字信号。该模块可以通过 ADCCLK 时钟驱动，提供一定的采样速率。
- ADC 数据寄存器
  - **Regular Data Register**：用于存储常规通道的转换结果。
  - **Injected Data Registers**：用于存储注入通道的转换结果，每个注入通道有一个独立的数据寄存器。

**模拟看门狗（Analog Watchdog）**

- 模拟看门狗是一种用于监控输入信号的硬件功能。它可以设定一个上限和下限（12 位精度），当 ADC 输入信号超出设定的阈值范围时，会触发报警或中断。
- 主要用于安全监控或报警用途，例如检测温度、电压或电流是否超出安全范围。

**标志和中断控制**

- Flags（标志位）

  ：用于指示 ADC 的状态。常见的标志位包括：

  - **OVR（Overrun）**：DMA 数据覆盖错误标志。
  - **EOC（End of Conversion）**：常规通道转换完成标志。
  - **JEOC（Injected End of Conversion）**：注入通道转换完成标志。
  - **AWD（Analog Watchdog）**：模拟看门狗事件标志。

- **中断使能位**：用于控制是否允许 ADC 模块生成中断。包括 OVRIE（覆盖中断）、EOCIE（转换完成中断）、JEOCIE（注入完成中断）、AWDIE（看门狗中断）等。

- **NVIC 中断**：ADC 可将中断信号发送至 NVIC（嵌套向量中断控制器）进行处理。

**DMA 支持**

- ADC 转换完成后可以通过 DMA（直接存储器访问）自动将数据转移到内存，而不需要 CPU 参与。
- **DMA request**：当常规通道完成转换后可以触发 DMA 请求，将数据自动传输到指定的存储区域。

## 特性表

在数据手册中：

![F407ADCshujutexingbiao](/public/image/Embedded/MCU/GD32/GD32F407/F407ADCshujutexingbiao.png)

**温度传感器特性表**

![F407ADCwenduchuanganqitexbiao](/public/image/Embedded/MCU/GD32/GD32F407/F407ADCwenduchuanganqitexbiao.png)

## 温度计算

**温度传感器计算公式如下：**

![F407ADCwendujisuan](/public/image/Embedded/MCU/GD32/GD32F407/F407ADCwendujisuan.png)

## 实现

功能：通过 ADC 采集芯片内部的问题

```c
#include "gd32f4xx.h"
#include "systick.h"
#include <stdio.h>
#include "main.h"
#include "BSP_USART.h"
/*
	通过 ADC 采集芯片内部温度
*/
// ADC 配置	
static void adc_config(void) {
	// 
	adc_deinit();
	rcu_periph_clock_enable(RCU_ADC0);	
	// 分频
	// 查看时钟树发现 ADC 组大支持 40MHz
	// 而 ADC 的两个时钟来源均超过 40 所以我们要先进行选择和分频
	// 选择时钟来源和分频 选择APB2，4分频
	adc_clock_config(ADC_ADCCK_PCLK2_DIV4);
	// 因为我们就一个 ADC 工作所以这里要选择独立模式
	// 独立模式是相对于同步模式来说的，所以在同步中设置
	adc_sync_mode_config(ADC_SYNC_MODE_INDEPENDENT);
	// 设置工作模式，单次还是多次，扫描还是不扫描
	// 只有 ADC0 支持内部通道，芯片温度属于内部通道
	adc_special_function_config(ADC0, ADC_SCAN_MODE,  DISABLE);// 失能扫描
	adc_special_function_config(ADC0, ADC_CONTINUOUS_MODE, DISABLE); // 非连续 
	// 是否打开插入通道(自动转换)，我们使用常规通道所以不打开
	adc_special_function_config(ADC0, ADC_INSERTED_CHANNEL_AUTO, DISABLE);
	// 设置分辨率
	adc_resolution_config(ADC0, ADC_RESOLUTION_12B);
	// 设置对齐方向
	adc_data_alignment_config(ADC0, ADC_DATAALIGN_RIGHT);
	// 设置转换通道的个数（包括常规通道和插入通道）
	// 我们这里只用到一个常规通道，所以就打开一个，如果是注入通道就选择注入同到
	adc_channel_length_config(ADC0, ADC_ROUTINE_CHANNEL, 1);
	// 将通道和引脚对应起来，或者说绑定起来
    // 注入通道方法为 adc_INSERTED_channel_config
	// 参数一、指定要配置的 ADC 外设
	// 参数二、要使用的通道编号，也就是在 ADC 的那个通道中进行转换
	// 参数三、通道编号，具体的哪个物理输入通道进行采样。
	// 参数四、设置 ADC 采样的时常
    // 如果想要对外部引脚的外设进行采样,
	adc_routine_channel_config(ADC0, 0, ADC_CHANNEL_16, ADC_SAMPLETIME_480);
	// 在对内部引脚进行采样时，需要特别设置一下的参数
	adc_channel_16_to_18(ADC_TEMP_VREF_CHANNEL_SWITCH, ENABLE);
	// 使能 ADC
	adc_enable(ADC0);	
	// ADC 使能后需等待 1 时间后才能采样，
	delay_1us(1);
	// 延迟14个CK_ADC以等待ADC稳定；	
	adc_calibration_enable(ADC0);	 
}

// 读取温度
void get_mcu_temperature(void) {	
	// 触发采集
	// 我们这里通过软件进行触发
	adc_software_trigger_enable(ADC0, ADC_ROUTINE_CHANNEL);
	// 等待 EOC 标志位置为，也就是等待采集成功
	while(RESET == adc_flag_get(ADC0, ADC_FLAG_EOC));
	// 常规序列转换结束时硬件置位，软件写 0 或读 ADC_RDATA 寄存器清除。
	uint16_t mcu_temperature_code = adc_routine_data_read(ADC0);
   	// 这里最好不要重复清除，当多任务时候，上面清楚后另一个任务可能就会立即写入
	// 这里如果后面在清除，可能会影响后续转换的值
	//adc_flag_clear(ADC0, ADC_FLAG_EOC);
	// 将对应的编码值转换为温度
    // 查看手册可以得到温度的转换公式：
    // 25V 下的电压为 1.45
    // 当前的电压值：(mcu_temperature_code / 4096 * 3.3) = 1.45
    // 4.1mV 要记得改为 V ==> 4.1 1 1000
    // ((1.45 - 当前的电压值) /  (4.1 / 1000)) + 25
	float mcu_temperature = (1.45 - (3.3 * mcu_temperature_code / 4096)) / (4.1 / 1000) + 25;
	printf("mcu_temperature = %f\n", mcu_temperature);	
}

int main(void) {
    systick_config();
    usart0_init();
	adc_config();
	
	printf("App_start...");
    
    while(1){
		delay_1ms(1000);
		get_mcu_temperature();		
	}
}   
============================================================================
// 在常规通道组中，多个通道通信采样时候只有最后一个会触发 EOC 标志位
// 所以当我们需要通过进行多个通道采样的时候，就需要放一次采样一次，以此来得到真确的结果；
// 该方法同时只能有一个通道进行工作，效率比较低    
// 添加电位器引脚的GPIO配置
rcu_periph_clock_enable(RCU_GPIOC);
gpio_mode_set(GPIOC, GPIO_MODE_ANALOG, GPIO_PUPD_NONE, GPIO_PIN_4);    
    
// 读取温度和电位器
void get_mcu_temperature(void) {	
	// 先将内部温度对应的物理通道放入通道进行采样
    adc_routine_channel_config(ADC0, 0, ADC_CHANNEL_16, ADC_SAMPLETIME_480);    
	adc_software_trigger_enable(ADC0, ADC_ROUTINE_CHANNEL);
	while(RESET == adc_flag_get(ADC0, ADC_FLAG_EOC));
	uint16_t mcu_temperature_code = adc_routine_data_read(ADC0);
	float mcu_temperature = (1.45 - (3.3 * mcu_temperature_code / 4096)) / (4.1 / 1000) + 25;
	printf("mcu_temperature = %f\n", mcu_temperature);
    
    // 再将电位器对应的物理通道放入通道进行采样
    adc_routine_channel_config(ADC0, 0, ADC_CHANNEL_14, ADC_SAMPLETIME_480);    
	adc_software_trigger_enable(ADC0, ADC_ROUTINE_CHANNEL);
	while(RESET == adc_flag_get(ADC0, ADC_FLAG_EOC));
	uint16_t mcu_temperature_code = adc_routine_data_read(ADC0);
	float mcu_dianweiqi = (3.3 * mcu_temperature_code / 4096);
	printf("mcu_dianweiqi = %f\n", mcu_dianweiqi);	
}   
==================================================================================
// 使用 DMA 实现多个通道的数据采集
// 当同时有多个通道时，只有最后一个通道会返回 EOC 标志位，所以我们可以使用 DMA 让每次采集完就把数据搬运出来
// 而不是等所有通道都采集完之后 EOC 置位才进行采集；    
/*************** ADC 中的 DMA配置 *********************/
// 当 DMA 使能后，每次转换完成后都启用 DMA 传输
adc_dma_request_after_last_enable(ADC0);
// DMA 使能
adc_dma_mode_enable(ADC0);
// 要放在 ADC 使能之前
// 触发采集，软件触发
adc_software_trigger_enable(ADC0, ADC_ROUTINE_CHANNEL);
// DMA 初始化
// DMA 配置
void adc_dma_config(void) {
	// 重置下 DMA
	dma_deinit(DMA1, DMA_CH0);	
	// 使能外设DMA1时钟
	rcu_periph_clock_enable(RCU_DMA1);
	// 配置 DMA 的传输模式(多数据传输/单数据传输)
	// 这里选择单数据传输
	dma_single_data_parameter_struct  dma_single_init;
    // 结构体初始化函数
	dma_single_data_para_struct_init(&dma_single_init);
	// 外设地址
	dma_single_init.periph_addr = (uint32_t)(&ADC_RDATA(ADC0));
	// 是否要使能外设地址偏移
	dma_single_init.periph_inc = DMA_PERIPH_INCREASE_DISABLE;
	// 内存地址，发送时候才知道，暂时无法配置	
	dma_single_init.memory0_addr = (uint32_t)dta;		
	// 使能内存地址偏移
	dma_single_init.memory_inc = DMA_MEMORY_INCREASE_ENABLE;	
	// 数据宽度，即每次搬运的数据大小
	dma_single_init.periph_memory_width = DMA_PERIPH_WIDTH_16BIT;	
	// 是否需要循环执行		
	dma_single_init.circular_mode = DMA_CIRCULAR_MODE_ENABLE;	
	// 数据搬运的方向 内存 -> 外设
	dma_single_init.direction = DMA_PERIPH_TO_MEMORY;	
	// 数据传输的位数，即我们需要发送少多少个（periph_memory_width）大小的数据
    // 这也要等到发送时候才能知道
	dma_single_init.number = 2U;
	// 优先级可以默认
	dma_single_init.priority = DMA_PRIORITY_HIGH;	
 	// DMA 初始化配置
	dma_single_data_mode_init(DMA1, DMA_CH0, &dma_single_init);
	// DMA 通道功能选择
	dma_channel_subperipheral_select(DMA1, DMA_CH0, DMA_SUBPERI0);	
	// 使能DMA1
	dma_channel_enable(DMA1, DMA_CH0);	
}

void adc_gpio_config(void) {
	rcu_periph_clock_enable(RCU_GPIOC);
	gpio_mode_set(GPIOC, GPIO_MODE_ANALOG, GPIO_PUPD_NONE, GPIO_PIN_4);    
}

// 读取温度和电位器电压
void get_mcu_temperature(void) {	
	float mcu_dianweiqi = (3.3 * dta[0] / 4095);	
	float mcu_temperature = (1.45 - (3.3 * dta[1] / 4095)) / (4.1 / 1000) + 25;
	printf("mcu_dianweiqi = %f\n", mcu_dianweiqi);	
	printf("mcu_temperature = %f\n", mcu_temperature);	
} 
```

> 注意在使用常规通道时候，不要多次清除 EOC 标志位；在多任务时可能会出现数据错误问题；
>
> adc_flag_clear(ADC0, ADC_FLAG_EOC); // 清除 EOC 标志位；
>
> adc_routine_data_read(); // 读取对应寄存器也会清楚 EOC 标志位；
>
> 二者不需要同时存在
