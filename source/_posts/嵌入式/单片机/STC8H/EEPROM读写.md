---
title: "EEPROM读写"
date: 2024-09-19 20:46:00
categories: "嵌入式"
tags: 
- "STC8H"
---

**EEPROM**（**Electrically Erasable Programmable Read-Only Memory**，电可擦除可编程只读存储器）是一种**非易失性存储器**，可以在**掉电后仍保留数据**。它允许**单字节**或**多字节**的数据被擦除和重新写入，而不需要整个芯片被擦除。嵌入式系统、微控制器以及各种电子设备中常使用 EEPROM 来存储需要在掉电后保留的数据，比如用户配置、校准参数等。

**ISP（In-System Programming）**

ISP，即在系统编，是一种允许开发者在目标系统（如电路板）上直接对芯片进行编程的技术。这意味着开发者无需将芯片从电路板上取下，即可通过特定的接口（如串口、JTAG、SPI等）将程序代码烧录到芯片内部的 Flash 存储器中。

**IAP（In-Application Programming）**

IAP，即在应用编程，是一种允许在应用程序运行时对芯片内部存储器进行编程或更新的技术。与 ISP 不同，IAP 通常需要在芯片内部划分出特定的存储区域（如引导区、运行程序区和下载区），并在应用程序中集成相应的编程逻辑。

## STC8H

**官方文档介绍**

![ROMguanfangwendangjieshao](/public/image/嵌入式/MCU/STC8H/ROMguanfangwendangjieshao.png)

**官方文档内部结构图**

![ROMjiegoutu](/public/image/嵌入式/MCU/STC8H/ROMjiegoutu.png)

STC8 系列单片机中都包含有 Flash 数据存储器（EEPROM）。以字节为单位进行读/写数据，以 512 字节为页单位进行擦除，可在线反复编程擦写 10 万次以上，提高了使用的灵活性和方便性。

**数据存储器**

STC8H系列单片机内部集成的RAM可用于存放程序执行的中间结果和过程数据。

![shujucunchuqi.png](/public/image/嵌入式/MCU/STC8H/shujucunchuqi.png)

```c
// 读写 String 字符串
#include "Config.h"
#include "UART.h"
#include "EEPROM.h"
#include <string.h>

void UART_config(void) {
    // >>> 记得添加 NVIC.c, UART.c, UART_Isr.c <<<
    COMx_InitDefine		COMx_InitStructure;					//结构定义
    COMx_InitStructure.UART_Mode      = UART_8bit_BRTx;	//模式, UART_ShiftRight,UART_8bit_BRTx,UART_9bit,UART_9bit_BRTx
    COMx_InitStructure.UART_BRT_Use   = BRT_Timer1;			//选择波特率发生器, BRT_Timer1, BRT_Timer2 (注意: 串口2固定使用BRT_Timer2)
    COMx_InitStructure.UART_BaudRate  = 115200ul;			//波特率, 一般 110 ~ 115200
    COMx_InitStructure.UART_RxEnable  = ENABLE;				//接收允许,   ENABLE或DISABLE
    COMx_InitStructure.BaudRateDouble = DISABLE;			//波特率加倍, ENABLE或DISABLE
    UART_Configuration(UART1, &COMx_InitStructure);		//初始化串口1 UART1,UART2,UART3,UART4

    NVIC_UART1_Init(ENABLE,Priority_1);		//中断使能, ENABLE/DISABLE; 优先级(低到高) Priority_0,Priority_1,Priority_2,Priority_3
    UART1_SW(UART1_SW_P30_P31);		// 引脚选择, UART1_SW_P30_P31,UART1_SW_P36_P37,UART1_SW_P16_P17,UART1_SW_P43_P44
}

#define     Max_Length          100      //读写EEPROM缓冲长度
u8  xdata   tmp[Max_Length];        //EEPROM操作缓冲

void main() {

    u16 addr_sector = 0x0000;
    char *str = "HelloWorld!abc123!";
    u16 str_length = strlen(str);	// 获取str的长度

    UART_config();
		
		EA = 1;

    // 擦除扇区, 一次性擦除一个扇区512字节, 从0x0000开始, 0x01FF
//    EEPROM_SectorErase(u16 EE_address);
    EEPROM_SectorErase(addr_sector);

//    // 写入数据. 字符串\int\long\float
////    EEPROM_write_n(u16 EE_address,u8 *DataAddress,u16 number);
    EEPROM_write_n(addr_sector, str, str_length);


    // 读取数据. 字符串\int\long\float
//    EEPROM_read_n(u16 EE_address,u8 *DataAddress,u16 number);
    EEPROM_read_n(addr_sector, tmp, str_length);
		
		// 添加字符串结束符
		tmp[str_length] = '\0';
			
		printf(">>存储的字符串: %s\n", str);
		printf(">>读到的字符串: %s\n", tmp);
		if(strcmp(str, tmp) == 0){
			printf("两个字符串相等\n");
		}else {
			printf("两个字符串不等\n");
		}
    
    while(1) {

    }
}
```

