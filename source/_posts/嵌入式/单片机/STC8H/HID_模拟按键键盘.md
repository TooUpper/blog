---
title: "HID_模拟键盘按键"
date: 2024-09-21 16:45:00
categories: "嵌入式"
tags: 
- "STC8H"
---



**如何去阅读 USB HID 设备开发指南**

**如何去看得懂官方的示例教程**

**按照我们的需求改造示例教程**

PC 通过 HID 协议与键盘进行交互，那么就可以分为两部分：

**STC8H 向 PC 发送按键报告**

```c
// usb_desc.c
// 按照我们自己的需求修改描述符中的内容
// 1.修改产品描述符 -> 也就是 PC 端中所显示的文本信息
char code PRODUCTDESC[12] = {
    0x0c,0x03, // 0x0ec 为描述符的长度，与 PRODUCTDESC[12] 中的 12 对应   
	'T',0,     
    'U',0,
	'-',0,
	0x2e,0x95, // 键盘的 ASCII: \u952e\u76d8
	0xd8,0x76
};

// 2.修改制造商信息 
// 修改方式同上
char code MANUFACTDESC[18] = {
    0x12,0x03,
    'T',0,
    'O',0,
    'O',0,
    'U',0,
    'P',0,
    'P',0,
    'E',0,
    'R',0,
};
// 其他内容照 STC8H 示例即可
// ...
// 注意查看 HIDREPORTDESC -> HID 报告的内容
=============================================================
// usb_desc.h
// 修改对应的头文件
extern char code MANUFACTDESC[18];
extern char code PRODUCTDESC[12];
// ...
=============================================================
// usb_req_class.c
// 按照我们自己的需求修改实现的方式
// 发送数据给PC，要传递进来8个字节长度的数组
void usb_class_in(BYTE key[8]) {
    BYTE i;
    
    if (DeviceState != DEVSTATE_CONFIGURED)
        return;		
		
    if (!UsbInBusy) {
        IE2 &= ~0x80;   //EUSB = 0;
        UsbInBusy = 1;
        usb_write_reg(INDEX, 1);
        for (i=0; i<8; i++) {
            usb_write_reg(FIFO1, key[i]);
        }
        usb_write_reg(INCSR1, INIPRDY);
        IE2 |= 0x80;    //EUSB = 1;
    }
}
=============================================================
// MatrixKey.c    
/*	扫描按键，如果需要感知按下或者弹起的状态，那么就传递进来按下和弹起的函数
	如果有某一个方向不想感知，那么可以直接传递 NULL
*/
u16 MK_Scan( void(*key_down)(u8,u8), void (*key_up)(u8,u8)) {	
	for(i = 0 ; i < 4 ; i++) {
        
		//1. 拉低第1行
		set_row(i);
			
		//2. 判定列
		for(j = 0 ; j < 4 ; j++){
			if(get_col(j) == 0 && is_up(status,i ,j)	){ // 按下了
				set_down(status ,i , j);
					
				if(key_down != NULL) key_down(i,j);
					
			}else if(get_col(j) == 1 && is_down(status,i,j) ){ // 弹起了
				set_up(status,i,j);
					
				if(key_up != NULL) key_up(i,j);
			}
		}
	}	
	return status; // 返回16个按键的状态
}
=============================================================
// main.c
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
    COMx_InitDefine COMx_Init;				
    COMx_Init.UART_Mode = UART_8bit_BRTx;
    COMx_Init.UART_BRT_Use = BRT_Timer1;
    COMx_Init.UART_BaudRate  = 115200ul;
    COMx_Init.UART_RxEnable  = ENABLE;	
    COMx_Init.BaudRateDouble = DISABLE;			
    UART_Configuration(UART1, &COMx_Init);	
    
  	NVIC_UART1_Init(ENABLE,Priority_1);		
    UART1_SW(UART1_SW_P30_P31);	
}

/* 准备好的键位对应的字母、功能键

Q		 W	 E	   R	
											  					   
A		 S	 D	   F	
											  					   
CapsLock C	 V     Backspace	
											  					   
Ctrl_L	 Gui Space Enter					

*/

/* 准备好键位对应的字节数据
0x14、0x1A、0x08、0x15
				   				   
0x04、0x16、0x07、0x09
				   				   
0x39、0x06、0x19、0x2A				   
				   
0xE0、0xE3、0x2C、0x58

*/

//为了更好的对应键位和字节，需要使用一个二维数组来组装字节数据
u8 key_arr[4][4]={
	{0x14		,   0x1A 		,		0x08		, 0x15},
	{0x04		,   0x16		,		0x07		,	0x09},
	{0x39		,   0x06		,		0x19		,	0x2A},
	{0xE0		,   0xE3		,		0x2C		,	0x58}
};

u16 key_arr2[] = {
	0x14, 0x1A, 0x08, 0x15,
	0x04, 0x16, 0x07,	0x09,
  0x39, 0x06, 0x19,	0x2A,
  0xE0, 0xE3, 0x2C,	0x58
};
u8 num = 2;
void key_down(u8 row, u8 col){	
} 

void key_up(u8 row, u8 col){
	// 如果我们要做成键盘，那么就必须走HID协议，此时串口就无法使用了
	//printf("up: %d-%d\n" , (int)row , (int)col);	
	u8 dat[8]={0}; // 一开始就是8个0
	
	num--;
	
	usb_class_in(dat);
}

u16 last_status;  // 1111 1111 1111 1111

void main() {
	u8 i;
	P_SW2 |= 0x80;  //扩展寄存器(XFR)访问使能   EAXSFR();

    usb_init(); //初始化USB信息，这样PC就知道我们是键盘了.

    EA = 1;

	//自己的代码
	UART_Config();
	MK_Init();
    
    while (1) {       
		u8 dat[8] = {0};
		u8 num = 2;
       //usb_class_in();  // 发送按键数据给PC
        
		//1. 先扫描16个按键的状态，并且获取到16个按键的数据【按的是哪些键】
		u16 status = MK_Scan(key_down , key_up);   // 1111 1101 1011 1111
			
		// 如果这次的键位数据和以前的键位数据一样，那么就不用发送键盘数据给PC了。
		if(status == last_status){
			delay_ms(20);
			continue;
		}
	
			//2. 遍历取出status的每一个二进制位，看看这个bit是否是 0 。 0-按下，1-弹起
			for( i= 0 ; i < 16 ; i++){  // 1111 1111  1011 0110
				
			//3. 依次取出来二进制位，判定是否是 0
				
			//3.1 status是16位 ，判断 第 0 位是否是 0,  status & (1<<0) == 0 
			//3.2 status是16位 ，判断 第 1 位是否是 0， status & (1<<1) == 0 
			//3.2 status是16位 ，判断 第 2 位是否是 0， status & (1<<2) == 0 
			//3.2 status是16位 ，判断 第 4 位是否是 0， status & (1<<3) == 0 
				if( ( status & (1<<i) ) == 0){ // 是按下
										
					//4. 判断是功能键吗？组装功能键  0   Modifierkeys (D0:LCtrl D1:LShift D2:LAlt D3:LGui D4:RCtrl D5:RShift D6:RAlt D7:RGui)
					if(i == 12){  //  Ctrl_L
						dat[0] |=  1<<0;
					}else if(i == 13){  // Windows键
						dat[0] |=  1<<3;
					}else { 
						//5. 判断是普通键吗？组装普通键 第 2 ~ 7 字节，用来装普通的按键 
						dat[num++] = key_arr2[i];
					}
					//5. 处理num的问题
					if(num > 7) num = 7;					
				}				
			}																			
			//2.7. 发送数据
			usb_class_in(dat);   // 0000 0000
			
		  //为了记录这次发送键位数据给PC的时候，16个按键是个什么状态。
			last_status = status;  // 1111 1111 1111 1111
		
			delay_ms(20);   
    }
}
```

**PC 向 STC8H 发送状态更新报告**

接收 PC 端发送过来的报告是模拟功能键的功能

例如：按下 NumLock键：指示灯亮起，

```c
// 对 PC 返回的报告进行响应或者处理是 STC8H 自动响应的我们只需找到处理函数然后调用我们的代码即可
// 当PC发送数据给键盘的时候，会调用这个函数
// 查看 usb_desc.c中返回值内容的格式
/*
Output Report:  【要站在PC的角度看输出的输入】  PC --------u8 dat--------> 键盘

数字锁定键 NumLock    大小写切换键  CapsLock   滚动锁定键  ScrollLock

0   LEDs (D0:NumLock D1:CapLock D2:ScrollLock)
*/
// usb_req_class.c
extern void hanlde_usb_out(BYTE led);
void usb_class_out() {
    
	 if (usb_bulk_intr_out(UsbBuffer, 1) == 1){		
			//直接调用我们的函数
			hanlde_usb_out( UsbBuffer[0]);		 
		}		
		/*
    BYTE led;
    
    if (usb_bulk_intr_out(UsbBuffer, 1) == 1)
    {
        P4M0 &= ~0x01;
        P4M1 &= ~0x01;
        P6M0 &= ~0xe0;
        P6M1 &= ~0xe0;
        P40 = 0;
        
        led = UsbBuffer[0];
        LED_NUM = !(led & 0x01);
        LED_CAPS = !(led & 0x02);
        LED_SCROLL = !(led & 0x04);
    }*/
}
===============================================================
// main.c
//处理键盘收到PC的数据  
// 0   LEDs (D0:NumLock D1:CapLock D2:ScrollLock)
void hanlde_usb_out(BYTE led){ 
	
	/*
		PC返回的值是这样的，按下第一次返回0，按下第二次就返回1，按下第三次就返回0，按下第四次就返回1，以此类推...
	*/
	//LED1 = led & 0x01;
	
	//if( led & (1<<0) ) LED1 = ~LED1; //亮起来;  按下了数字锁定键
	//if( led & (1<<1) ) LED2 = ~LED2; //亮起来;  按下了大小写切换键
	//if( led & (1<<2) ) LED3 = ~LED3; //亮起来;  按下了滚动锁定键
	
	LED1 = !(led & 0x01);     //亮起来;  按下了数字锁定键
	LED5 = !(led & 0x02);     //亮起来;  按下了大小写切换键
	LED8 = !(led & 0x04);     //亮起来;  按下了滚动锁定键	
}
```

