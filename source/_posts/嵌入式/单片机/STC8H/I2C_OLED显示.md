---
title: "I2C_OLED显示"
date: 2024-09-15 11:32:00
categories: "嵌入式"
tags: 
- "STC8H"
---

**OLED（有机发光二极管，Organic Light-Emitting Diode）** 是一种显示技术，通过使用有机材料在电流通过时自发光来产生图像。与传统的 LCD（液晶显示器）不同，OLED 不需要背光源，因此可以实现更薄的显示器，并且具备许多独特的优点。

## I2C 协议

I2C（Inter-Integrated Circuit）协议是一种广泛使用的**双线同步串行通信协议**，由飞利浦半导体（现为恩智浦半导体）在1982年发明。该协议以其简单的硬件连接和灵活的设备支持，在嵌入式系统和各种电子设备中得到了广泛应用。

**I2C 特点**

- **双线通信**：I2C 是一种**双线通信**协议，使用两条线进行数据传输
  - **SDA（Serial Data Line）**: 用于数据传输的双向数据线。
  - **SCL（Serial Clock Line）**: 用于同步数据传输的时钟线，由主设备生成时钟信号。
- **主从架构**：支持**多个主设备**和**多个从设备**，可以灵活配置
  - **主设备（Master）**: 控制通信过程，生成时钟信号，并决定是发送数据还是接收数据。
  - **从设备（Slave）**: 被动响应主设备的操作，它根据主设备发出的指令接收或发送数据。
  - **唯一地址**: 每个从设备在总线上都有一个唯一的 7 位或 10 位地址，主设备通过地址识别与哪个从设备通信。
- **同步通信**：时钟信号由主设备控制，从设备被动接收；但多主设备时主设备之间不能同时发起通信。
- **寻址机制**：每个从设备都有一个唯一的地址，主设备通过地址来选择与哪个从设备通信。

**I2C 的通信过程**

1. **起始条件（START Condition）**
   - 主设备将 SDA 线从**高电平拉低**，同时 SCL 保持高电平。这告诉所有从设备即将开始通信。

2. **发送设备地址（Address Phase）**

   - 主设备接着发送从设备的**7 位地址**（或 10 位地址，视协议扩展），紧接着一个**读/写位**（1 表示读，0 表示写）。SDA 在 SCL 时钟的**高电平期间**发送地址。

   - 从设备检查地址是否与自身匹配，匹配的设备将准备回应。

3. **应答（ACK/NACK）**
   - 在地址发送后，从设备将**拉低 SDA 线**表示应答（ACK）。如果 SDA 保持高电平，则表示无应答（NACK），通信终止。

4. **数据传输（Data Transfer Phase）**

   - 主设备开始发送或接收数据（视读/写位）。每次数据传输是 8 位，数据在 SCL 线的**高电平期间**稳定，接收端读取数据。

   - 每 8 位数据后，接收端发送一个应答信号（ACK/NACK）。如果发送结束，接收端会发 NACK，表示数据结束。

5. **停止条件（STOP Condition）**
   - 主设备将 SDA 从**低电平拉高**，同时 SCL 保持高电平，表示数据传输结束，释放总线。

> **通信流程总结：**
>
> 1. 起始条件（START）
> 2. 发送设备地址 + 读/写位
> 3. 应答信号（ACK）
> 4. 传输数据（8 位）
> 5. 接收设备发送应答（ACK/NACK）
> 6. 如果有更多数据，重复步骤 4-5
> 7. 发送停止条件（STOP）
>
> 这种过程可以让主设备与多个从设备通信，因为每个从设备都有独立的地址。

## SSD1306

**SSD1306** 是一款常用的单色 OLED（有机发光二极管）显示控制器驱动芯片，广泛用于小型OLED屏幕中，如 128x64 或 128x32 分辨率的显示屏。它的设计专为驱动OLED面板，并通过 I2C、SPI 或并行接口与微控制器或单片机通信。

### 实现

**引脚图**

![OLEDyinjioatu](/public/image/嵌入式/MCU/STC8H/OLEDyinjioatu.png)

#### 软件模拟

```c
// main.c
//////////////////////////////////////////////////////////////////////////////////	 
//本程序只供学习使用，未经作者许可，不得用于其它任何用途
//中景园电子
//店铺地址：http://shop73023976.taobao.com/?spm=2013.1.0.0.M4PqC2
//
//  文 件 名   : main.c
//  版 本 号   : v2.0
//  作    者   : HuangKai
//  生成日期   : 2014-0101
//  最近修改   : 
//  功能描述   : OLED 4接口演示例程(51系列)
//              说明: 
//              ----------------------------------------------------------------
//              GND    电源地
//              VCC  接5V或3.3v电源
//              SCL  P10（SCL）
//              SDA  P11（SDA）
//              RES  P12 注：SPI接口显示屏改成IIC接口时需要接RES引脚
//                           IIC接口显示屏用户请忽略
//              ----------------------------------------------------------------
// 修改历史   :
// 日    期   : 
// 作    者   : HuangKai
// 修改内容   : 创建文件
//版权所有，盗版必究。
//Copyright(C) 中景园电子2014/3/16
//All rights reserved
//******************************************************************************/
#include "oled.h"
#include "bmp.h"
#include "STC8G_H_GPIO.h"

void GPIO_Config(void) {
	GPIO_InitTypeDef GPIO_Init;
	GPIO_Init.Mode = GPIO_PullUp;
	GPIO_Init.Pin = GPIO_Pin_2 | GPIO_Pin_3;

	GPIO_Inilize(GPIO_P3, &GPIO_Init);
}

int main(void) {	
		u8 t=' ';

	GPIO_Config();
	OLED_Init();//初始化OLED
	
	OLED_ColorTurn(0);//0正常显示，1 反色显示
  	OLED_DisplayTurn(0);//0正常显示 1 屏幕翻转显示
	while(1) {		
		OLED_DrawBMP(0,0,128,64,BMP1);
		delay_ms(500);
		OLED_Clear();
		OLED_ShowChinese(0,0,0,16);//中
		OLED_ShowChinese(18,0,1,16);//景
		OLED_ShowChinese(36,0,2,16);//园
		OLED_ShowChinese(54,0,3,16);//电
		OLED_ShowChinese(72,0,4,16);//子
		OLED_ShowChinese(90,0,5,16);//科
		OLED_ShowChinese(108,0,6,16);//技
		OLED_ShowString(8,2,"ZHONGJINGYUAN",16);
		OLED_ShowString(20,4,"2014/05/01",16);
		OLED_ShowString(0,6,"ASCII:",16);  
		OLED_ShowString(63,6,"CODE:",16);
		OLED_ShowChar(48,6,t,16);
		t++;
		if(t>'~')t=' ';
		OLED_ShowNum(103,6,t,3,16);
		delay_ms(500);
		OLED_Clear();
	}	  	
}

===========================================================================
// olde.h
#ifndef __OLED_H
#define __OLED_H

#include "Config.h"	  	 
	
#define OLED_CMD  0	//写命令
#define OLED_DATA 1	//写数据

sbit OLED_SCL=P3^2;//SCL
sbit OLED_SDA=P3^3;//SDA
sbit OLED_RES =P1^2;//RES

//-----------------OLED端口定义----------------

#define OLED_SCL_Clr() OLED_SCL=0
#define OLED_SCL_Set() OLED_SCL=1

#define OLED_SDA_Clr() OLED_SDA=0
#define OLED_SDA_Set() OLED_SDA=1

#define OLED_RES_Clr() OLED_RES=0
#define OLED_RES_Set() OLED_RES=1

//OLED控制用函数
void delay_ms(unsigned int ms);
void OLED_ColorTurn(u8 i);
void OLED_DisplayTurn(u8 i);
void OLED_WR_Byte(u8 dat,u8 cmd);
void OLED_Set_Pos(u8 x, u8 y);
void OLED_Display_On(void);
void OLED_Display_Off(void);
void OLED_Clear(void);
void OLED_ShowChar(u8 x,u8 y,u8 chr,u8 sizey);
u32 oled_pow(u8 m,u8 n);
void OLED_ShowNum(u8 x,u8 y,u32 num,u8 len,u8 sizey);
void OLED_ShowString(u8 x,u8 y,u8 *chr,u8 sizey);
void OLED_ShowChinese(u8 x,u8 y,u8 no,u8 sizey);
void OLED_DrawBMP(u8 x,u8 y,u8 sizex, u8 sizey,u8 BMP[]);
void OLED_Init(void);

#endif  
=============================================================
// olde.c
#include "oled.h"
#include "oledfont.h"  	 
//OLED的显存
//存放格式如下.
//[0]0 1 2 3 ... 127	
//[1]0 1 2 3 ... 127	
//[2]0 1 2 3 ... 127	
//[3]0 1 2 3 ... 127	
//[4]0 1 2 3 ... 127	
//[5]0 1 2 3 ... 127	
//[6]0 1 2 3 ... 127	
//[7]0 1 2 3 ... 127 			   
void delay_ms(unsigned int ms) {                         
	unsigned int a;
	while(ms) {
		a=1800;
		while(a--);
		ms--;
	}
	return;
}

//反显函数
void OLED_ColorTurn(u8 i) {
	if(i==0) {
			OLED_WR_Byte(0xA6,OLED_CMD);//正常显示
		}
	if(i==1) {
			OLED_WR_Byte(0xA7,OLED_CMD);//反色显示
		}
}

//屏幕旋转180度
void OLED_DisplayTurn(u8 i) {
	if(i==0) {
			OLED_WR_Byte(0xC8,OLED_CMD);//正常显示
			OLED_WR_Byte(0xA1,OLED_CMD);
		}
	if(i==1) {
			OLED_WR_Byte(0xC0,OLED_CMD);//反转显示
			OLED_WR_Byte(0xA0,OLED_CMD);
		}
}

//延时
void IIC_delay(void) {
	u8 t=1;
	while(t--);
}

//起始信号
void I2C_Start(void) {
	OLED_SDA_Set();
	OLED_SCL_Set();
	IIC_delay();
	OLED_SDA_Clr();
	IIC_delay();
	OLED_SCL_Clr();	 
}

//结束信号
void I2C_Stop(void) {
	OLED_SDA_Clr();
	OLED_SCL_Set();
	IIC_delay();
	OLED_SDA_Set();
}

//等待信号响应
void I2C_WaitAck(void) {//测数据信号的电平
	OLED_SDA_Set();
	IIC_delay();
	OLED_SCL_Set();
	IIC_delay();
	OLED_SCL_Clr();
	IIC_delay();
}

//写入一个字节
void Send_Byte(u8 dat) {
	u8 i;
	for(i=0;i<8;i++) {        
		OLED_SCL_Clr();//将时钟信号设置为低电平
		if(dat&0x80) { //将dat的8位从最高位依次写入
			OLED_SDA_Set();
    	}
		else {
			OLED_SDA_Clr();
    	}
		IIC_delay(); // 延时
		OLED_SCL_Set(); // 拉高信号线（高电平时写入）
		IIC_delay(); 	// 延时
		OLED_SCL_Clr(); // 拉低信号线
		dat<<=1;
  }
}

//发送一个字节
//向SSD1306写入一个字节。
//mode:数据/命令标志 0,表示命令;1,表示数据;
void OLED_WR_Byte(u8 dat,u8 mode) {
	I2C_Start();
	Send_Byte(0x78);
	I2C_WaitAck();
	if(mode){Send_Byte(0x40);}
  else{Send_Byte(0x00);}
	I2C_WaitAck();
	Send_Byte(dat);
	I2C_WaitAck();
	I2C_Stop();
}

//坐标设置
void OLED_Set_Pos(u8 x, u8 y) { 
	OLED_WR_Byte(0xb0+y,OLED_CMD);
	OLED_WR_Byte(((x&0xf0)>>4)|0x10,OLED_CMD);
	OLED_WR_Byte((x&0x0f),OLED_CMD);
}   	  
//开启OLED显示    
void OLED_Display_On(void) {
	OLED_WR_Byte(0X8D,OLED_CMD);  //SET DCDC命令
	OLED_WR_Byte(0X14,OLED_CMD);  //DCDC ON
	OLED_WR_Byte(0XAF,OLED_CMD);  //DISPLAY ON
}
//关闭OLED显示     
void OLED_Display_Off(void) {
	OLED_WR_Byte(0X8D,OLED_CMD);  //SET DCDC命令
	OLED_WR_Byte(0X10,OLED_CMD);  //DCDC OFF
	OLED_WR_Byte(0XAE,OLED_CMD);  //DISPLAY OFF
}		   			 
//清屏函数,清完屏,整个屏幕是黑色的!和没点亮一样!!!	  
void OLED_Clear(void) {  
	u8 i,n;		    
	for(i=0;i<8;i++) {  
		OLED_WR_Byte (0xb0+i,OLED_CMD);    //设置页地址（0~7）
		OLED_WR_Byte (0x00,OLED_CMD);      //设置显示位置—列低地址
		OLED_WR_Byte (0x10,OLED_CMD);      //设置显示位置—列高地址   
		for(n=0;n<128;n++)OLED_WR_Byte(0,OLED_DATA); 
	} //更新显示
}

//在指定位置显示一个字符,包括部分字符
//x:0~127
//y:0~63				 
//sizey:选择字体 6x8  8x16
void OLED_ShowChar(u8 x,u8 y,u8 chr,u8 sizey) {      	
	u8 c=0,sizex=sizey/2;
	u16 i=0,size1;
	if(sizey==8)size1=6;
	else size1=(sizey/8+((sizey%8)?1:0))*(sizey/2);
	c=chr-' ';//得到偏移后的值
	OLED_Set_Pos(x,y);
	for(i=0;i<size1;i++) {
		if(i%sizex==0&&sizey!=8) OLED_Set_Pos(x,y++);
		if(sizey==8) OLED_WR_Byte(asc2_0806[c][i],OLED_DATA);//6X8字号
		else if(sizey==16) OLED_WR_Byte(asc2_1608[c][i],OLED_DATA);//8x16字号
//		else if(sizey==xx) OLED_WR_Byte(asc2_xxxx[c][i],OLED_DATA);//用户添加字号
		else return;
	}
}
//m^n函数
u32 oled_pow(u8 m,u8 n) {
	u32 result=1;	 
	while(n--)result*=m;    
	return result;
}				  
//显示数字
//x,y :起点坐标
//num:要显示的数字
//len :数字的位数
//sizey:字体大小		  
void OLED_ShowNum(u8 x,u8 y,u32 num,u8 len,u8 sizey) {         	
	u8 t,temp,m=0;
	u8 enshow=0;
	if(sizey==8)m=2;
	for(t=0;t<len;t++) {
		temp=(num/oled_pow(10,len-t-1))%10;
		if(enshow==0&&t<(len-1)) {
			if(temp==0) {
				OLED_ShowChar(x+(sizey/2+m)*t,y,' ',sizey);
				continue;
			}else enshow=1;
		}
	 	OLED_ShowChar(x+(sizey/2+m)*t,y,temp+'0',sizey);
	}
}
//显示一个字符号串
void OLED_ShowString(u8 x,u8 y,u8 *chr,u8 sizey) {
	u8 j=0;
	while (chr[j]!='\0') {		
		OLED_ShowChar(x,y,chr[j++],sizey);
		if(sizey==8)x+=6;
		else x+=sizey/2;
	}
}
//显示汉字
void OLED_ShowChinese(u8 x,u8 y,u8 no,u8 sizey) {
	u16 i,size1=(sizey/8+((sizey%8)?1:0))*sizey;
	for(i=0;i<size1;i++) {
		if(i%sizey==0) OLED_Set_Pos(x,y++);
		if(sizey==16) OLED_WR_Byte(Hzk[no][i],OLED_DATA);//16x16字号
//		else if(sizey==xx) OLED_WR_Byte(xxx[c][i],OLED_DATA);//用户添加字号
		else return;
	}				
}

//显示图片
//x,y显示坐标
//sizex,sizey,图片长宽
//BMP：要显示的图片
void OLED_DrawBMP(u8 x,u8 y,u8 sizex, u8 sizey,u8 BMP[]) { 	
  u16 j=0;
	u8 i,m;
	sizey=sizey/8+((sizey%8)?1:0);
	for(i=0;i<sizey;i++) {
		OLED_Set_Pos(x,i+y);
    for(m=0;m<sizex;m++) {      
			OLED_WR_Byte(BMP[j++],OLED_DATA);	    	
		}
	}
} 

//初始化				    
void OLED_Init(void) {
	OLED_RES_Clr();
	delay_ms(200);
	OLED_RES_Set();
	
	OLED_WR_Byte(0xAE,OLED_CMD);//--turn off oled panel
	OLED_WR_Byte(0x00,OLED_CMD);//---set low column address
	OLED_WR_Byte(0x10,OLED_CMD);//---set high column address
	OLED_WR_Byte(0x40,OLED_CMD);//--set start line address  Set Mapping RAM Display Start Line (0x00~0x3F)
	OLED_WR_Byte(0x81,OLED_CMD);//--set contrast control register
	OLED_WR_Byte(0xCF,OLED_CMD); // Set SEG Output Current Brightness
	OLED_WR_Byte(0xA1,OLED_CMD);//--Set SEG/Column Mapping     0xa0左右反置 0xa1正常
	OLED_WR_Byte(0xC8,OLED_CMD);//Set COM/Row Scan Direction   0xc0上下反置 0xc8正常
	OLED_WR_Byte(0xA6,OLED_CMD);//--set normal display
	OLED_WR_Byte(0xA8,OLED_CMD);//--set multiplex ratio(1 to 64)
	OLED_WR_Byte(0x3f,OLED_CMD);//--1/64 duty
	OLED_WR_Byte(0xD3,OLED_CMD);//-set display offset	Shift Mapping RAM Counter (0x00~0x3F)
	OLED_WR_Byte(0x00,OLED_CMD);//-not offset
	OLED_WR_Byte(0xd5,OLED_CMD);//--set display clock divide ratio/oscillator frequency
	OLED_WR_Byte(0x80,OLED_CMD);//--set divide ratio, Set Clock as 100 Frames/Sec
	OLED_WR_Byte(0xD9,OLED_CMD);//--set pre-charge period
	OLED_WR_Byte(0xF1,OLED_CMD);//Set Pre-Charge as 15 Clocks & Discharge as 1 Clock
	OLED_WR_Byte(0xDA,OLED_CMD);//--set com pins hardware configuration
	OLED_WR_Byte(0x12,OLED_CMD);
	OLED_WR_Byte(0xDB,OLED_CMD);//--set vcomh
	OLED_WR_Byte(0x40,OLED_CMD);//Set VCOM Deselect Level
	OLED_WR_Byte(0x20,OLED_CMD);//-Set Page Addressing Mode (0x00/0x01/0x02)
	OLED_WR_Byte(0x02,OLED_CMD);//
	OLED_WR_Byte(0x8D,OLED_CMD);//--set Charge Pump enable/disable
	OLED_WR_Byte(0x14,OLED_CMD);//--set(0x10) disable
	OLED_WR_Byte(0xA4,OLED_CMD);// Disable Entire Display On (0xa4/0xa5)
	OLED_WR_Byte(0xA6,OLED_CMD);// Disable Inverse Display On (0xa6/a7) 
	OLED_Clear();
	OLED_WR_Byte(0xAF,OLED_CMD); /*display ON*/ 
}
```

#### 硬件实现

STC8H 本身支持硬件实现，就是不用上面他们写的函数，给换成我们 STC8H 库函数中的读写函数就可以了。
