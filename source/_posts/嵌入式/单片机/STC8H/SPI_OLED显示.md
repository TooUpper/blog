---
title: "SPI_OLED显示"
date: 2024-09-21 21:42:00
categories: "嵌入式"
tags: 
- "STC8H"
---

SPI（串行外设接口）是一种广泛应用的**全双工同步串行通信协议**，通常用于微控制器与外部设备之间的高速数据传输。它采用**主从架构**，通过**主设备生成时钟信号**并控制数据流动，通过四条主要信号线**（MOSI、MISO、SCLK和SS）**实现**全双工通信**。数据传输通过时钟的**上升或下降沿**进行，且支持可配置的时钟极性（CPOL）和相位（CPHA），以适应不同设备的要求。尽管SPI的传输速度较高，且实现相对简单，但其缺点在于需要较多的引脚和短距离传输的限制，适合用于传感器、存储器、显示屏及其他外设的接口。整体来说，SPI因其速度和灵活性在嵌入式系统中得到了广泛应用。

**全双工：**全双工指的是通信的双方可以同时进行数据的发送和接收，彼此互不干扰。(也就是说需要至少两条线进行通信)

**同步：**指数据的传输是通过一个共同的时钟信号进行协调的。所有参与通信的设备都依赖于同一时钟信号来发送和接收数据。

**串行通信：** 数据通过一条数据线依次**一位一位**的发送。

发送设备和接收设备在**同一时刻**依据**同一个时钟信号**进行数据的采样和传输。

**SPI 协议通信过程**

1. **主设备选择从设备**：通过将对应从设备的 **SS** 线拉低，选中某个从设备进行通信。

2. **时钟同步**：主设备生成 **SCLK**，并在时钟上升沿或下降沿驱动数据。

3. **数据传输**：

   - 在时钟脉冲的作用下，主设备通过 **MOSI** 线发送数据，同时从设备通过 **MISO** 线发送数据。

   - 数据在时钟的上升沿或下降沿采样，具体依赖于时钟极性和相位设置。


4. **完成通信**：数据传输结束后，主设备将 **SS** 线拉高，终止对从设备的选择。

**通信过程图**

![SPItongxinliucheng](/public/image/嵌入式/MCU/STC8H/SPItongxinliucheng.jpg)

> MOSI: 主设备输出，从设备输入（接收）
>
> MISO: 主设备输入（接收），从设备输出
>
> SCLK: 主设备生成的时钟信号
>
> SS(NSS): 选择从设备的的信号（当每次传输完成后，主设备拉高 SS 信号，结束通信）

## OLED显示屏

本次示例屏采用的是中景园的 0.96 寸 OLED 显示屏  SSD1306 驱动带字库

**如图所示**

![SPIOLED](/public/image/嵌入式/MCU/STC8H/SPIOLED.png)

> 右图中左侧六个引脚的为字库芯片
>
> 右图中右侧三个引脚的为线性稳压器

**引脚说明**

![SPIyinjiaoshuom](/public/image/嵌入式/MCU/STC8H/SPIyinjiaoshuom.png)

> **GND**：逻辑电路的接地端。必须连接到地。
>
> **VCC**：OLED 的电源输入端。必须连接到电源。
>
> **CLK / SCL**：串行时钟输入端。
>
> **MOSI / SDA**：串行数据输入端。
>
> **DC**：数据/命令控制引脚。当引脚拉高时，SDA输入作为显示数据处理；当引脚拉低时，SDA输入传递到命令寄存器。
>
> **CS1**：OLED 芯片选择引脚；低电平使能，高电平禁止。
>
> **FSO**：字库芯片的数据输出引脚。
>
> **CS2**：字库芯片选择引脚；低电平使能，高电平禁止。

**原理图**

![SPIyuanlitu](/public/image/嵌入式/MCU/STC8H/SPIyuanlitu.png)

> X096-2864KSWAG01：OLED 模块
>
> 8 * 2.54 ：8个引脚，引脚间距为 2.54
>
> GT20L16S1Y：字库 IC
>
> ME6206a33XG：线性稳压器

**函数先根据你要展示的内容，去字库芯片中查找他的位置，然从指定位置中读出这些字，然后展示在 OLED 上**

## STC8H

**引脚图**

![SPISTC8Hyinjiaotu](/public/image/嵌入式/MCU/STC8H/SPISTC8Hyinjiaotu.png)

**字库 IC 中地址计算方法，以 GB2313为例**

![SPIICziku](/public/image/嵌入式/MCU/STC8H/SPIICziku.png)

### 实现

```c
// 示例里的函数讲解

// 1.以写入 GB2313 字符集的汉字为例
// 以左上角为原点，显示：12864，带中文字库
OLED_Display_GB2312_string(0,0,"12864，带中文字库");

// 2.根据字库IC提供的计算方法算出该字符点阵在字库IC中的位置
u32 fontaddr=0;
void OLED_Display_GB2312_string(u8 x,u8 y,u8 *text) {
	u8 i=0;
	u8 addrHigh,addrMid,addrLow; 
	u8 fontbuf[32];
	while(text[i]>0x00)
	{
		if((text[i]>=0xb0)&&(text[i]<=0xf7)&&(text[i+1]>=0xa1))
		{
			//¹ú±ê¼òÌå£¨GB2312£©ºº×ÖÔÚ¾§ÁªÑ¶×Ö¿âICÖÐµÄµØÖ·ÓÉÒÔÏÂ¹«Ê½À´¼ÆËã£º
			//Address = ((MSB - 0xB0) * 94 + (LSB - 0xA1)+ 846)*32+ BaseAdd;BaseAdd=0
			//ÓÉÓÚµ£ÐÄ8Î»µ¥Æ¬»úÓÐ³Ë·¨Òç³öÎÊÌâ£¬ËùÒÔ·ÖÈý²¿È¡µØÖ·
			fontaddr=(text[i]-0xb0)*94;
			fontaddr+=(text[i+1]-0xa1)+846;
			fontaddr=fontaddr*32;
			
			addrHigh=(fontaddr&0xff0000)>>16;   //µØÖ·µÄ¸ß8Î»,¹²24Î»
			addrMid=(fontaddr&0xff00)>>8;       //µØÖ·µÄÖÐ8Î»,¹²24Î»
			addrLow=(fontaddr&0xff);            //µØÖ·µÄµÍ8Î»,¹²24Î»
			
			OLED_get_data_from_ROM(addrHigh,addrMid,addrLow,fontbuf,32);
			//È¡32¸ö×Ö½ÚµÄÊý¾Ý£¬´æµ½"fontbuf[32]"
			OLED_Display_16x16(x,y,fontbuf);
			//ÏÔÊ¾ºº×Öµ½LCDÉÏ£¬yÎªÒ³µØÖ·£¬xÎªÁÐµØÖ·£¬fontbuf[]ÎªÊý¾Ý
			x+=16;
			i+=2;
    }
		else if((text[i]>=0xa1)&&(text[i]<=0xa3)&&(text[i+1]>=0xa1))
		{
			
			fontaddr=(text[i]-0xa1)*94;
			fontaddr+=(text[i+1]-0xa1);
			fontaddr=fontaddr*32;
			
			addrHigh=(fontaddr&0xff0000)>>16;
			addrMid=(fontaddr&0xff00)>>8;
			addrLow=(fontaddr&0xff);
			
			OLED_get_data_from_ROM(addrHigh,addrMid,addrLow,fontbuf,32);
			OLED_Display_16x16(x,y,fontbuf);
			x+=16;
			i+=2;
    }
		else if((text[i]>=0x20)&&(text[i]<=0x7e))
		{
			unsigned char fontbuf[16];
			fontaddr=(text[i]-0x20);
			fontaddr=(unsigned long)(fontaddr*16);
			fontaddr=(unsigned long)(fontaddr+0x3cf80);
			
			addrHigh=(fontaddr&0xff0000)>>16;
			addrMid=(fontaddr&0xff00)>>8;
			addrLow=fontaddr&0xff;
			
			OLED_get_data_from_ROM(addrHigh,addrMid,addrLow,fontbuf,16);
			OLED_Display_8x16(x,y,fontbuf);
			x+=8;
			i+=1;
    }
		else 
			i++;
  }
}

//向SSD1306写入一个字节。
//mode:数据/命令标志 0,表示命令;1,表示数据;
void OLED_WR_Byte(u8 dat,u8 cmd) {	
	u8 i;	
    // 判断是命令（0）还是数据（1），将 DC 置 0/1
	if(cmd) { 
	  OLED_DC_Set(); // DC: 1
	}
	else {
	  OLED_DC_Clr(); // DC: 0	
	}	  
    // CS1 置 0，选择 OLED 芯片（SSD1306）
    // 表示要向 SSD1306 中写数据了
	OLED_CS_Clr(); 
	for(i=0;i<8;i++) { // 写一个字节	8 位		  
		OLED_SCL_Clr(); // 将 SCL(时钟) 置 0
		if(dat&0x80) { // 从高位开始，依次判断 0 / 1
		   OLED_SDA_Set(); // 1 就将 SDA  置 1
		}
		else {
		   OLED_SDA_Clr(); // 0 就将 SDA 置 0
		}
		OLED_SCL_Set(); // 将 SCL 值 1，也就是上升沿
		dat<<=1;   
	}				 		  
	OLED_CS_Set(); // 将 CS 置 1（不选择 OLED）
	OLED_DC_Set(); // 拉高作为数据  
}

// 从相关地址中：(计算出汉字在字库中的地址，然后将地址写入，然后从写入的地址中读出数据)
//（addrHigh：地址高字节,addrMid：地址中字节,addrLow：地址低字节）
// 中连续读出 DataLen 个字节的数据到 pbuff 的地址
// 连续读取
// 由于担心8位单片机有乘法溢出问题，所以分三部取地址
void OLED_get_data_from_ROM(u8 addrHigh,u8 addrMid,u8 addrLow,u8 *pbuff,u8 DataLen) {
	u8 i;
	OLED_ROM_CS_Clr();
    // 0x03 通常是“读取数据”的命令
    // 表示准备从指定的地址开始读取数据。
	Send_Command_to_ROM(0x03);
    //送指令到晶联讯字库IC
	Send_Command_to_ROM(addrHigh);
	Send_Command_to_ROM(addrMid);
	Send_Command_to_ROM(addrLow);
    // 从字库 IC 中读取出来 
	for(i=0;i<DataLen;i++) {
		*(pbuff+i)=Get_data_from_ROM();
  }
	OLED_ROM_CS_Set();
}

//从晶联讯字库IC中取汉字或字符数据（1个字节）
u8 Get_data_from_ROM(void) {
	u8 i,read=0;
	for(i=0;i<8;i++) {
		OLED_SCL_Clr();
		read<<=1;
        // OLED_READ_FS0: 字库芯片的数据输出引脚。
		if(OLED_READ_FS0) { 
			read++;
    }
		OLED_SCL_Set();
  }
	return read;
}
```

> 函数先根据你要展示的内容，去字库芯片中查找他的位置，然从指定位置中读出这些字，然后展示在 OLED 上

## 问题

一、**SPI 和 UART 的区别**

二、**write 的代码也就是在控制数据点的高低，也没有显示的写入 OLED 的代码呀，为什么 OLED 可以保存并显示内容呢**

在 `OLED_WR_Byte()` 函数中，虽然没有显式写入 SSD1306 内存的代码，但实际的数据传输是通过 **SPI 或 I2C** 接口的硬件控制实现的。

**具体的写入过程**

1. **数据传输过程**：
   每个字节通过 SDA 引脚按位传输。`OLED_SDA_Set()` 和 `OLED_SDA_Clr()` 控制数据线的电平状态（高或低），代表传输的每一位。每次时钟 `OLED_SCL_Set()` 上升沿时，SSD1306 会读取数据引脚上的电平，形成一个位数据。

2. **命令或数据的区分**：
   `OLED_DC_Set()` 和 `OLED_DC_Clr()` 用于区分当前传输的是**数据**还是**命令**。高电平表示数据，低电平表示命令。这决定了 SSD1306 如何处理接收到的信息。

3. **芯片选择 (CS)**：
   `OLED_CS_Clr()` 拉低 CS 信号，告诉 SSD1306 开始通信。传输完成后，`OLED_CS_Set()` 结束通信。SSD1306 会根据传输的内容将数据存储到相应的显存地址中，并在下一个刷新周期内更新显示内容。

三、**时钟极性*CKP/Clock Polarity）**

四、**时钟相位**

[SPI协议详解（图文并茂+超详细） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/290620901)
