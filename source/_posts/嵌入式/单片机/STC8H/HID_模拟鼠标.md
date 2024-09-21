---
title: "HID_模拟鼠标"
date: 2024-09-21 19:58:00
categories: "嵌入式"
tags: 
- "STC8H"
---

```c
// 通过 STC8H 的矩阵键盘模拟 鼠标的功能
// 按照我们自己的需求修改描述符配置
// usb_desc.c
// 定义 鼠标的 配置描述符
/* 我们加了一个 scroll 也就是鼠标滚轮的功能
Input Report:
0   Buttons (D0:LButton D1:RButton D2:MButton [D3:D7]:Pad)
1	X displacement (>0:right; <0:left)
2	Y displacement (>0:down; <0:up)
3 scroll
*/
// 此处我们需要修改的地方有：
// 1. 添加鼠标滚轮的描述符内容：0x09,0x38,Wheel;  鼠标滚轮   修改 1
// 2. 将原本报告的计数从 2 改为3 // 0x95,0x03,REPORT_COUNT(3);   修改2
// 3. 将整个报告描述符的总长度进行修改 HIDREPORTDESC[52]，0x05,0x01, (包括头文件中的内容)
// 4. 将配置描述符中发送的最大字节数从 3 改为 4
// 5. 将配置描述符中的协议长度改为 52 与报告描述符的总长度要一致
// 注意：当我们将鼠标滚轮的功能追加在第四个字节也就是末尾的时候，对应 HIDREPORTDESC 
// 中的位置也要在 x、y，的下面，要保证发送的内容与报告描述符的内容对应起来；
char code HIDREPORTDESC[52] =  // 修改3 --- 头文件的长度
{
    0x05,0x01,              //USAGE_PAGE(Generic Desktop);
    0x09,0x02,              //USAGE(Mouse);
    0xa1,0x01,              //COLLECTION(Application);
    0x09,0x01,              //  USAGE(Pointer);
    0xa1,0x00,              //  COLLECTION(Physical);
    0x05,0x09,              //    USAGE_PAGE(Buttons);
    0x19,0x01,              //    USAGE_MINIMUM(1);
    0x29,0x03,              //    USAGE_MAXIMUM(3);
    0x15,0x00,              //    LOGICAL_MINIMUM(0);
    0x25,0x01,              //    LOGICAL_MAXIMUM(1);
    0x75,0x01,              //    REPORT_SIZE(1);
    0x95,0x03,              //    REPORT_COUNT(3);
    0x81,0x02,              //    INPUT(Data,Variable,Absolute);
    0x75,0x05,              //    REPORT_SIZE(5);
    0x95,0x01,              //    REPORT_COUNT(1);
    0x81,0x01,              //    INPUT(Constant);
    0x05,0x01,              //    USAGE_PAGE(Generic Desktop);
    0x09,0x30,              //    USAGE(X);  x方向移动
    0x09,0x31,              //    USAGE(Y);  y方向移动
	0x09,0x38,              //    Wheel;  鼠标滚轮   修改 1
    0x15,0x81,              //    LOGICAL_MINIMUM(-127);
    0x25,0x7f,              //    LOGICAL_MAXIMUM(127);
    0x75,0x08,              //    REPORT_SIZE(8);
    0x95,0x03,              //    REPORT_COUNT(3);   修改2
    0x81,0x06,              //    INPUT(Data, Variable, Relative);
    0xc0,                   //  END_COLLECTION;
    0xc0,                   //END_COLLECTION;
};

// 配置描述符
char code CONFIGDESC[41] = {
	// ...
    // 遵循的协议版本 HID 1.01协议
    0x09,                   //bLength(9);
    0x21,                   //bDescriptorType(HID);
    0x01,0x01,              //bcdHID(1.01);
    0x00,                   //bCountryCode(0);
    0x01,                   //bNumDescriptors(1);
    0x22,                   //bDescriptorType(HID Report);
    0x34,0x00,              //wDescriptorLength(52);
    
	// 描述鼠标给PC发的数据信息
    0x07,                   //bLength(7);
    0x05,                   //bDescriptorType(Endpoint);
    0x81,                   //bEndpointAddress(EndPoint1 as IN);
    0x03,                   //bmAttributes(Interrupt);
    0x04,0x00,              //wMaxPacketSize(4);   修改4 鼠标1次会给PC发送4个字节
    0x0a,                   //bInterval(10ms); 
};
===============================================
// usb_req_class.c
// 修改处理函数
void usb_class_in(BYTE button[4]) {
    BYTE i;
        
    if (DeviceState != DEVSTATE_CONFIGURED)
        return;

    if (!UsbInBusy ){
        IE2 &= ~0x80;   //EUSB = 0;
        UsbInBusy = 1;
        usb_write_reg(INDEX, 1);
        for (i=0; i<4; i++){
            usb_write_reg(FIFO1, button[i]);
        }
        usb_write_reg(INCSR1, INIPRDY);
        IE2 |= 0x80;    //EUSB = 1;
    }
}    
===============================================
// main.c
#include "GPIO.h"
#include "UART.h"
#include "NVIC.h"
#include "Switch.h"
#include "Delay.h"
#include "MatrixKey.h"

#include "stc.h"
#include "usb.h"
#include "usb_req_class.h"
#include "timer.h"

void UART_Config(void) {
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

/*
0、L_Button、M_Button、R_Button
																	    
0	  0			-y		 -s
																	    
0	  -x		0		 +x
																	    
0	  0			+y		 +s

*/
// 按下按键...
void key_down(u8 row, u8 col){
	
	//1. 准备好4个字节长度的数组，初值都是 0
	u8 dat[4] = {0};	
    
	//2. 判定按下的哪个键位，组装数据	
	//2.1 判定按键
	if(row ==0 && col ==1) dat[0] |= 1<<0; // 左键
	if(row ==0 && col ==3) dat[0] |= 1<<1; // 右键
	if(row ==0 && col ==2) dat[0] |= 1<<2; // 中键
	
	//2.2 判定x方向
	if(row ==2 && col ==1) dat[1] = -10;  // 向左移动
	if(row ==2 && col ==3) dat[1] = 10;   // 向右移动
	
	//2.3 判定y方向
	if(row ==1 && col ==2) dat[2] = -10;  // -y 向上移动
	if(row ==3 && col ==2) dat[2] = 10;   // +y 向下移动
	
	//2.4 判定滚轮
	if(row ==1 && col ==3) dat[3] = -10;  // -s 向下滚动
	if(row ==3 && col ==3) dat[3] = 10;   // +s 向上滚动
	
	//3. 发数据
	usb_class_in(dat);
	
	//printf("dat[0]=%d\n", (int)dat[0]);
	//printf("dat[1]=%d\n", (int)dat[1]);
	//printf("dat[2]=%d\n", (int)dat[2]);
	//printf("dat[3]=%d\n", (int)dat[3]);	
}

void key_up(u8 row, u8 col) {
	u8 dat[4] = {0};
	printf("up..\n");
	
	usb_class_in(dat);
}

void main() {
	P_SW2 |= 0x80;  //扩展寄存器(XFR)访问使能

	// usb初始化
    usb_init();
	
	//初始化矩阵键盘
	MK_Init();
    
    EA = 1;
	UART_Config();
    
    while (1) {
		MK_Scan( key_down, key_up );        
        delay_ms(20);
    }
}
```

