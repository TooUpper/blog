---
title: "MFRC522"
date: 2024-12-16 21:48:00
categories: "IC"
tags: 
- "IC"
---

MFRC522 是一款应用广泛的非接触读卡器芯片,集成了在 13.56MHz 下的多种非接触通信方式和协议，具有很高的技术集成度。

## 规格说明

**工作电压**：3.3V

**工作电流**：10-26mA

**控制方式**：SPI、I2C、UART

## 引脚说明

![yjsm](/public/image/嵌入式/IC/MFRC522/yjsm.png)

| **管脚号** | **管脚名** | **类型** | **管脚描述**                                                 |
| :--------: | :--------: | :------: | ------------------------------------------------------------ |
|     1      |     A1     |    G     | 地地址线                                                     |
|     2      |    PVDD    |    P     | 管脚电源                                                     |
|     3      |    DVDD    |    P     | 数字电源                                                     |
|     4      |    DVSS    |    G     | 数字地                                                       |
|     5      |    PVSS    |    G     | 管脚电源地                                                   |
|     6      |   NRSTPD   |    I     | 复位脚。为低电平时，内部功能模块包括振荡器均停止工作，输入管脚与外部断开。 |
|            |            |          | 该管脚上的上升沿可用来开启内部复位相位。                     |
|     7      |   SIGIN    |    I     | 通信接口输入：接收数字数据流、串行数据流                     |
|     8      |   SIGOUT   |    O     | 通信接口输出：输出串行数据流                                 |
|     9      |    SVDD    |    P     | S²C 管脚电源：向 S²C 管脚供电                                |
|     10     |    TVSS    |    G     | 发送端 TX1 和 TX2 输出级的地                                 |
|     11     |    TX1     |    O     | 载波发送管脚 1                                               |
|     12     |    TVDD    |    P     | 发送驱动器电源                                               |
|     13     |    TX2     |    O     | 载波发送管脚 2                                               |
|     14     |    TVSS    |    G     | 发送驱动器电源地                                             |
|     15     |    AVDD    |    P     | 模拟电源                                                     |
|     16     |    VMID    |    P     | 内部参考电压                                                 |
|     14     |     RX     |    I     | RF 信号输入                                                  |
|     18     |    AVSS    |    G     | 模拟地                                                       |
|     19     |    AUX1    |    O     | 用于测试的辅助输出                                           |
|     20     |    AUX2    |    O     | 用于测试的辅助输出                                           |
|     21     |   OSCIN    |    I     | 晶振反向放大器；也就是外部时钟输入<br />该引脚还可以作外部时钟（fosc = 27.12 MHz）的输入 |
|     22     |   OSCOUT   |    O     | 晶振反向放大器输出                                           |
|     23     |    IRQ     |    O     | 中断请求输出：指示一个中断事件                               |
|     24     |    ALE     |    I     | 地址锁存使能：高电平时将 AD0-AD5 所存到内部地址所存          |
|   25-31    |   D1-D7    |   I/O    | 8 位双向数据总线 注：不支持 8 位并行接口<br />注：如果主控制器选择 I2C 作为数字主控制器接口，那么这些管脚可以用来定义 PC 地址<br />注：对于串行接口，这些管脚可以用作测试信号或 I/O |
|     32     |     AO     |    I     | 地址线                                                       |

## 读写模式

### ISO/IEC 14443A

ISO/IEC 14443A 是一种通信协议，用于支持 A 型非接触式智能卡与读写器之间的交互。

典型应用：广泛应用于 MIFARE 卡（如 MIFARE Classic、MIFARE DESFire）等系统。

**通信方式：**

- 调制方式：
  - 读写器发出的信号使用**ASK（幅移键控）调制**。
  - 数据传输采用**曼彻斯特编码**。
- 数据速率：
  - 支持 106 kbps 的数据传输速率。
- 防冲突机制：
  - 支持多卡防冲突机制，可在多张卡同时存在的情况下识别和通信。

**工作流程：**

1. **读写器初始化**：向周围发送询问信号，检测是否有卡片进入其工作范围。
2. **卡片应答**：卡片收到信号后，回应自己的唯一标识符（UID）。
3. **身份验证**：读写器和卡片进行认证，以确认双方合法性。
4. **数据传输**：开始交换数据，如读卡信息、写入数据等。
5. **终止通信**：完成通信后，读写器发送停止信号。

### ISO/IEC 14443B

ISO/IEC 14443B 是另一种通信协议，与 A 型不同，B 型主要用于特定的安全场景和一些非接触式卡片类型（如电子护照、健康卡等）。

**通信方式：**

- 调制方式：
  - 读写器发出的信号也使用**ASK 调制**。
  - 数据传输采用**BPSK（二进制相移键控）编码**。
- 数据速率：
  - 与 A 型相同，支持 106 kbps 的数据传输速率。
- 防冲突机制：
  - 支持独特的防冲突机制，与 A 型不同。

**工作流程：**

1. **读写器初始化**：向周围发送询问信号，检测是否有卡片进入其工作范围。
2. **卡片应答**：B 型卡片采用不同的应答机制，通常会发送特定的 ATQB 信号。
3. **数据传输**：双方进行数据交换，通常会进行更复杂的加密和验证。
4. **终止通信**：读写器发送停止信号，结束通信。

> 这是两种**非接触式智能卡通信协议**，它们定义了如何在读卡器（例如 RC522）和非接触式卡之间进行通信。也就是说只要是这两个通信协议的非接触式卡都可以与 RC522 进行通信。

## 通信方式

### I2C

支持 I2C 总线接口可以使得主机可以用较少的管脚连接到 MFRC522I2C 接口操作遵循 I2C 总线接口规范。该接口只能工作在从机模式。

![I2Cdlt](/public/image/嵌入式/IC/MFRC522/I2Cdlt.png)

在标准模式、快速模式和高速模式中，MFRC522 可用作从接收器或从发送器 SDA 是一个双向数据线，通过一个上拉电阻连接到正电压。不传输数据时，SDA 和 SCL 均为高电平。标准模式下,I2C 总线的传输速率为 100kBd,快速模式下为 400kBd,高速模式下为 3. 4Mbit/s。
如果选择 I2C 总线接口，管脚 SCL 和 SDA 管脚具有符合 I2C 接口规范的尖峰脉冲抑制功能。

**每个字节后面必须跟一个应答位。数据传输时高位在前。一次数据传输发送的字节数不限，但必须符合读/写周期格式。**

**应答**

应答是在一个数据字节结束后强制产生的。应答相应的时钟脉冲由主机产生。在应答时钟脉冲周期内，数据发送器释放 SDA 线(高电平)。在应时钟脉冲期间，**接收器拉低 SDA 线**使得它在该时钟脉冲的高电平时间内保持低电平。

![I2Cyd](/public/image/嵌入式/IC/MFRC522/I2Cyd.png)

**地址**

I2C总线地址与EA管脚的定义有关。

如果 EA 管脚为低电平，则对于所有 MFRC522 器件，器件总线地址的高4位固定为 0101b。 器件总线地址剩余的3位(ADR_0，ADR_1，ADR_2)可由用户自由配置，这样就可以防止与其它 I2C 器件产生冲突。

如果 EA 管脚设置为高电平，则 ADR_0-ADR_5 完全由外部管脚来确定。ADR_6 总是设置为0。

在这两种模式下，外部地址编码都在复位条件释放后立即锁定。不考虑使用管脚上的进一步变化。通过配置外部连线，I2C总线的地址管脚还可用作测试信号的输出。

![I2Cdz](/public/image/嵌入式/IC/MFRC522/I2Cdz.png)

### UART

![UARTlx](/public/image/嵌入式/IC/MFRC522/UARTlx.png)

通过对 TestPinEnReg寄存器的 RS23LineEn 位清零，信号 DTRQ 和 MX 可以禁止。

**传输速度**

默认的传输速率为 9.6kBaud。要改变传输速率，主机控制器必须向 SerialSpeedReg 寄存器写 入一个新的传输速率值。位 BR TO[2:0] 和位 BR_T1[4:0] 定义的因数用来设置 SerialSpeedReg 中的传输速率。

![UARTslb](/public/image/嵌入式/IC/MFRC522/UARTslb.png)

**帧格式**

|   位   | 长度 | 值   |
| :----: | :--: | :--- |
| 起始位 | 1 位 | 0    |
| 数据位 | 8 位 | 数值 |
| 结束位 | 1 位 | 1    |

对于数据和地址字节，LSB 位必须最先发送。传输过程中不使用奇偶校验位。

**发送和读写数据时，第一个字节的内容为地址，第二个字节开始的才是数据**

第一个字节的 MSB 位设置使用的模式。MSB 位设置为 1 时从 MFRC522 读取数据，MSB 位设置 为 0 时将数据写入 MFRC522。第一个字节的位6保留为将来使用。

![UARTdz](/public/image/嵌入式/IC/MFRC522/UARTdz.png)

### SPI

MFRC522 支持 SPI 接口与主机的高速通信，接口可处理高达 10Mbit/s 的数据速率。在与主机通信时，MFRC522 作为一个从机，从外设主机上接收数据来设置寄存器发送和接 收与 RF 接口通信有关的数据。

![SPItxfs](/public/image/嵌入式/IC/MFRC522/SPItxfs.png)

在SPI通信中 MFRC522 作为从机。SPI 时钟信号 SCK 必须由主机产生。数据通过 MOSI 线从主机传输到从机。通过 MIS0 线数据从MFRC522 发回到主机。
MOSI 和 MISO 传输每个字节时都是**高位在前**。MOSI 和 MISO 上的数据在时钟的上升沿必须保持不变，在时钟的下降沿改变。

**地址**

第一个字节的 MSB 位定义了使用模式。MSB 位设置为 1 时,从 MFRC522 读取数据 MSB 位设置为 0 时，将数据写入 MFRC522。第一个字节的位 6-1 定义地址，LSB 位应当设置为 0。

![SPIdz](/public/image/嵌入式/IC/MFRC522/SPIdz.png)

### 不同接口的接线方式

![jxfs](/public/image/嵌入式/IC/MFRC522/jxfs.png)

## RFID

**RFID**（Radio Frequency Identification）是一种通过无线电波进行物体识别和数据交换的技术。RFID 系统可以在没有接触的情况下，从一定距离内读取标签中的信息。

### 组成

一个完整的 RFID 系统通常由三部分组成：**RFID 标签**（Tag）、**RFID 读写器**（Reader）、**中间件或计算机系统**（Middleware/Backend System）。

**RFID 标签（Tag）**

RFID 标签是被贴在物品上或集成到物体内的设备，内含一个微芯片和天线。RFID 标签的主要作用是存储与物体相关的信息（如物品编号、存储数据等），并通过无线电波与 RFID 读写器进行通信。

**RFID 读写器（Reader）**

RFID 读写器负责发送和接收射频信号，它通过天线发射一个射频信号到 RFID 标签，同时接收从标签反射回来的信号并解码，从而获取标签上的数据。

**中间件或计算机系统（Middleware/Backend System）**

这个系统通常负责处理从 RFID 读写器接收到的标签信息。它可能会包括数据库、管理软件、企业资源规划（ERP）系统等，用来存储和分析标签数据，并执行后续的决策或操作。

### 工作原理

**激活**：RFID 读写器通过天线发射电磁波（射频信号），激活附近的 RFID 标签。对于被动标签，读写器的信号为其提供能量。

**响应**：被激活的 RFID 标签通过其天线将存储的信息（如卡号或其他数据）以无线电波的形式传回读写器。

**解码与处理**：读写器接收标签的信号，并将其解码。然后，读写器将信息传输到计算机系统或中间件，以便进一步处理（如查询数据库、进行验证等）。

**反馈（可选）**：在某些系统中，读写器可能需要发送反馈或执行其他操作，例如解锁门禁、发送警报等。

## IC 卡

S50 是一种**非接触式智能卡**，可以被 RC522 读写模块读取和写入。

RC522 模块和非接触式 IC 卡的通信是基于射频识别技术（RFID）。当卡片靠近读写器时，读写器发出一个射频信号，IC 卡中的天线通过电磁感应获取能量，并响应 RC522 发来的请求，传输数据。

## 示例代码

```c
//rc522.c
#include "bsp_rc522.h"
#include "board.h"

/******************************************************************
 * 函 数 名 称：RC522_Init
 * 函 数 说 明：IC卡感应模块配置
 * 函 数 形 参：无
 * 函 数 返 回：无
 * 作       者：LC
 * 备       注：
******************************************************************/
void RC522_Init(void){
    //开启时钟
    RCC_APB2PeriphClockCmd(RCC_GPIO, ENABLE);

    GPIO_InitTypeDef  GPIO_InitStructure;

    // SDA SCK MOSI RST
    GPIO_InitStructure.GPIO_Pin = GPIO_CS|GPIO_SCK|GPIO_MOSI|GPIO_RST;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(PORT_GPIO, &GPIO_InitStructure);
    GPIO_SetBits(PORT_GPIO, GPIO_CS|GPIO_SCK|GPIO_MOSI|GPIO_RST);

    // MISO
    GPIO_InitStructure.GPIO_Pin = GPIO_MISO;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_Init(PORT_GPIO, &GPIO_InitStructure);
}

////////////////软件模拟SPI与RC522通信///////////////////////////////////////////
/* 软件模拟SPI发送一个字节数据，高位先行 */
void RC522_SPI_SendByte( uint8_t byte ){
    uint8_t n;
    for( n=0;n<8;n++ ){
            if( byte&0x80 )
                    RC522_MOSI_1();
            else
                    RC522_MOSI_0();

            delay_us(200);
            RC522_SCK_0();
            delay_us(200);
            RC522_SCK_1();
            delay_us(200);

            byte<<=1;
    }
}

/* 软件模拟SPI读取一个字节数据，先读高位 */
uint8_t RC522_SPI_ReadByte( void ){
    uint8_t n,data;
    for( n=0;n<8;n++ ){
            data<<=1;
            RC522_SCK_0();
            delay_us(200);

            if( RC522_MISO_GET()==1 )
                    data|=0x01;

            delay_us(200);
            RC522_SCK_1();
            delay_us(200);

    }
    return data;
}

//////////////////////////STM32对RC522寄存器的操作//////////////////////////////////
/*  读取RC522指定寄存器的值
    向RC522指定寄存器中写入指定的数据
    置位RC522指定寄存器的指定位
    清位RC522指定寄存器的指定位
*/

/**
  * @brief  ：读取RC522指定寄存器的值
        * @param  ：Address:寄存器的地址
  * @retval ：寄存器的值
*/
uint8_t RC522_Read_Register( uint8_t Address ){
    uint8_t data,Addr;

    Addr = ( (Address<<1)&0x7E )|0x80;

    RC522_CS_Enable();
    RC522_SPI_SendByte( Addr );
    data = RC522_SPI_ReadByte();//读取寄存器中的值
    RC522_CS_Disable();

    return data;
}

/**
  * @brief  ：向RC522指定寄存器中写入指定的数据
  * @param  ：Address：寄存器地址
                                      data：要写入寄存器的数据
  * @retval ：无
*/
void RC522_Write_Register( uint8_t Address, uint8_t data ){
    uint8_t Addr;

    Addr = ( Address<<1 )&0x7E;

    RC522_CS_Enable();
    RC522_SPI_SendByte( Addr );
    RC522_SPI_SendByte( data );
    RC522_CS_Disable();
}

/**
  * @brief  ：置位RC522指定寄存器的指定位
  * @param  ：Address：寄存器地址
                                      mask：置位值
  * @retval ：无
*/
void RC522_SetBit_Register( uint8_t Address, uint8_t mask ){
    uint8_t temp;
    /* 获取寄存器当前值 */
    temp = RC522_Read_Register( Address );
    /* 对指定位进行置位操作后，再将值写入寄存器 */
    RC522_Write_Register( Address, temp|mask );
}

/**
  * @brief  ：清位RC522指定寄存器的指定位
  * @param  ：Address：寄存器地址
                      mask：清位值
  * @retval ：无
*/
void RC522_ClearBit_Register( uint8_t Address, uint8_t mask ){
    uint8_t temp;
    /* 获取寄存器当前值 */
    temp = RC522_Read_Register( Address );
    /* 对指定位进行清位操作后，再将值写入寄存器 */
    RC522_Write_Register( Address, temp&(~mask) );
}

///////////////////STM32对RC522的基础通信///////////////////////////////////
/*
    开启天线
    关闭天线
    复位RC522
    设置RC522工作方式
*/

/**
  * @brief  ：开启天线
  * @param  ：无
  * @retval ：无
*/
void RC522_Antenna_On( void ){
    uint8_t k;
    k = RC522_Read_Register( TxControlReg );
    /* 判断天线是否开启 */
    if( !( k&0x03 ) )
       RC522_SetBit_Register( TxControlReg, 0x03 );
}

/**
  * @brief  ：关闭天线
  * @param  ：无
  * @retval ：无
*/
void RC522_Antenna_Off( void ){
    /* 直接对相应位清零 */
    RC522_ClearBit_Register( TxControlReg, 0x03 );
}

/**
  * @brief  ：复位RC522
  * @param  ：无
  * @retval ：无
*/
void RC522_Rese( void ){
    RC522_Reset_Disable();
    delay_us ( 1 );
    RC522_Reset_Enable();
    delay_us ( 1 );
    RC522_Reset_Disable();
    delay_us ( 1 );
    RC522_Write_Register( CommandReg, 0x0F );
    while( RC522_Read_Register( CommandReg )&0x10 )
            ;

    /* 缓冲一下 */
    delay_us ( 1 );
    RC522_Write_Register( ModeReg, 0x3D );       //定义发送和接收常用模式
    RC522_Write_Register( TReloadRegL, 30 );     //16位定时器低位
    RC522_Write_Register( TReloadRegH, 0 );      //16位定时器高位
    RC522_Write_Register( TModeReg, 0x8D );      //内部定时器的设置
    RC522_Write_Register( TPrescalerReg, 0x3E ); //设置定时器分频系数
    RC522_Write_Register( TxAutoReg, 0x40 );     //调制发送信号为100%ASK
}

/**
  * @brief  ：设置RC522的工作方式
  * @param  ：Type：工作方式
  * @retval ：无
  M500PcdConfigISOType
*/
void RC522_Config_Type( char Type ){
    if( Type=='A' ){
        RC522_ClearBit_Register( Status2Reg, 0x08 );
        RC522_Write_Register( ModeReg, 0x3D );
        RC522_Write_Register( RxSelReg, 0x86 );
        RC522_Write_Register( RFCfgReg, 0x7F );
        RC522_Write_Register( TReloadRegL, 30 );
        RC522_Write_Register( TReloadRegH, 0 );
        RC522_Write_Register( TModeReg, 0x8D );
        RC522_Write_Register( TPrescalerReg, 0x3E );
        delay_us(2);
        /* 开天线 */
        RC522_Antenna_On();
    }
}

/////////////////////////STM32控制RC522与M1卡的通信///////////////////////////////////////
/*
    通过RC522和M1卡通讯（数据的双向传输）
    寻卡
    防冲突
    用RC522计算CRC16（循环冗余校验）
    选定卡片
    校验卡片密码
    在M1卡的指定块地址写入指定数据
    读取M1卡的指定块地址的数据
    让卡片进入休眠模式
*/

/**
  * @brief  ：通过RC522和ISO14443卡通讯
* @param  ：ucCommand：RC522命令字
 *          pInData：通过RC522发送到卡片的数据
 *          ucInLenByte：发送数据的字节长度
 *          pOutData：接收到的卡片返回数据
 *          pOutLenBit：返回数据的位长度
  * @retval ：状态值MI_OK，成功
*/
char PcdComMF522 ( uint8_t ucCommand, uint8_t * pInData, uint8_t ucInLenByte, uint8_t * pOutData, uint32_t * pOutLenBit ){
    char cStatus = MI_ERR;
    uint8_t ucIrqEn   = 0x00;
    uint8_t ucWaitFor = 0x00;
    uint8_t ucLastBits;
    uint8_t ucN;
    uint32_t ul;

    switch ( ucCommand )
    {
       case PCD_AUTHENT:                //Mifare认证
          ucIrqEn   = 0x12;                //允许错误中断请求ErrIEn  允许空闲中断IdleIEn
          ucWaitFor = 0x10;                //认证寻卡等待时候 查询空闲中断标志位
          break;

       case PCD_TRANSCEIVE:                //接收发送 发送接收
          ucIrqEn   = 0x77;                //允许TxIEn RxIEn IdleIEn LoAlertIEn ErrIEn TimerIEn
          ucWaitFor = 0x30;                //寻卡等待时候 查询接收中断标志位与 空闲中断标志位
          break;

       default:
         break;

    }

    RC522_Write_Register ( ComIEnReg, ucIrqEn | 0x80 );                //IRqInv置位管脚IRQ与Status1Reg的IRq位的值相反
    RC522_ClearBit_Register ( ComIrqReg, 0x80 );                        //Set1该位清零时，CommIRqReg的屏蔽位清零
    RC522_Write_Register ( CommandReg, PCD_IDLE );                //写空闲命令
    RC522_SetBit_Register ( FIFOLevelReg, 0x80 );                        //置位FlushBuffer清除内部FIFO的读和写指针以及ErrReg的BufferOvfl标志位被清除

    for ( ul = 0; ul < ucInLenByte; ul ++ )
                  RC522_Write_Register ( FIFODataReg, pInData [ ul ] );                    //写数据进FIFOdata

    RC522_Write_Register ( CommandReg, ucCommand );                                        //写命令

    if ( ucCommand == PCD_TRANSCEIVE )
                        RC522_SetBit_Register(BitFramingReg,0x80);                                  //StartSend置位启动数据发送 该位与收发命令使用时才有效

    ul = 1000;//根据时钟频率调整，操作M1卡最大等待时间25ms

    do                                                                                                                 //认证 与寻卡等待时间
    {
         ucN = RC522_Read_Register ( ComIrqReg );                                                        //查询事件中断
         ul --;
    } while ( ( ul != 0 ) && ( ! ( ucN & 0x01 ) ) && ( ! ( ucN & ucWaitFor ) ) );                //退出条件i=0,定时器中断，与写空闲命令

    RC522_ClearBit_Register ( BitFramingReg, 0x80 );                                        //清理允许StartSend位

    if ( ul != 0 )
    {
                        if ( ! ( RC522_Read_Register ( ErrorReg ) & 0x1B ) )                        //读错误标志寄存器BufferOfI CollErr ParityErr ProtocolErr
                        {
                                cStatus = MI_OK;

                                if ( ucN & ucIrqEn & 0x01 )                                        //是否发生定时器中断
                                  cStatus = MI_NOTAGERR;

                                if ( ucCommand == PCD_TRANSCEIVE )
                                {
                                        ucN = RC522_Read_Register ( FIFOLevelReg );                        //读FIFO中保存的字节数

                                        ucLastBits = RC522_Read_Register ( ControlReg ) & 0x07;        //最后接收到得字节的有效位数

                                        if ( ucLastBits )
                                                * pOutLenBit = ( ucN - 1 ) * 8 + ucLastBits;           //N个字节数减去1（最后一个字节）+最后一位的位数 读取到的数据总位数
                                        else
                                                * pOutLenBit = ucN * 8;                                           //最后接收到的字节整个字节有效

                                        if ( ucN == 0 )
            ucN = 1;

                                        if ( ucN > MAXRLEN )
                                                ucN = MAXRLEN;

                                        for ( ul = 0; ul < ucN; ul ++ )
                                          pOutData [ ul ] = RC522_Read_Register ( FIFODataReg );
                                        }
      }
                        else
                                cStatus = MI_ERR;
    }

   RC522_SetBit_Register ( ControlReg, 0x80 );           // stop timer now
   RC522_Write_Register ( CommandReg, PCD_IDLE );

   return cStatus;
}

/**
  * @brief  ：寻卡
* @param  ucReq_code，寻卡方式
*                      = 0x52：寻感应区内所有符合14443A标准的卡
 *                     = 0x26：寻未进入休眠状态的卡
 *         pTagType，卡片类型代码
 *                   = 0x4400：Mifare_UltraLight
 *                   = 0x0400：Mifare_One(S50)
 *                   = 0x0200：Mifare_One(S70)
 *                   = 0x0800：Mifare_Pro(X))
 *                   = 0x4403：Mifare_DESFire
  * @retval ：状态值MI_OK，成功
*/
char PcdRequest ( uint8_t ucReq_code, uint8_t * pTagType ){
   char cStatus;
   uint8_t ucComMF522Buf [ MAXRLEN ];
   uint32_t ulLen;

   RC522_ClearBit_Register ( Status2Reg, 0x08 );        //清理指示MIFARECyptol单元接通以及所有卡的数据通信被加密的情况
   RC522_Write_Register ( BitFramingReg, 0x07 );        //        发送的最后一个字节的 七位
   RC522_SetBit_Register ( TxControlReg, 0x03 );        //TX1,TX2管脚的输出信号传递经发送调制的13.46的能量载波信号

   ucComMF522Buf [ 0 ] = ucReq_code;                //存入寻卡方式
        /* PCD_TRANSCEIVE：发送并接收数据的命令，RC522向卡片发送寻卡命令，卡片返回卡的型号代码到ucComMF522Buf中 */
   cStatus = PcdComMF522 ( PCD_TRANSCEIVE,        ucComMF522Buf, 1, ucComMF522Buf, & ulLen );        //寻卡

   if ( ( cStatus == MI_OK ) && ( ulLen == 0x10 ) )        //寻卡成功返回卡类型
   {
                 /* 接收卡片的型号代码 */
       * pTagType = ucComMF522Buf [ 0 ];
       * ( pTagType + 1 ) = ucComMF522Buf [ 1 ];
   }
   else
     cStatus = MI_ERR;
     return cStatus;
}

/**
  * @brief  ：防冲突
        * @param  ：Snr：卡片序列，4字节，会返回选中卡片的序列
  * @retval ：状态值MI_OK，成功
*/
char PcdAnticoll ( uint8_t * pSnr ){
    char cStatus;
    uint8_t uc, ucSnr_check = 0;
    uint8_t ucComMF522Buf [ MAXRLEN ];
          uint32_t ulLen;

    RC522_ClearBit_Register ( Status2Reg, 0x08 );                //清MFCryptol On位 只有成功执行MFAuthent命令后，该位才能置位
    RC522_Write_Register ( BitFramingReg, 0x00);                //清理寄存器 停止收发
    RC522_ClearBit_Register ( CollReg, 0x80 );                        //清ValuesAfterColl所有接收的位在冲突后被清除

    ucComMF522Buf [ 0 ] = 0x93;        //卡片防冲突命令
    ucComMF522Buf [ 1 ] = 0x20;

          /* 将卡片防冲突命令通过RC522传到卡片中，返回的是被选中卡片的序列 */
    cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 2, ucComMF522Buf, & ulLen);//与卡片通信

    if ( cStatus == MI_OK)                //通信成功
    {
                        for ( uc = 0; uc < 4; uc ++ )
                        {
         * ( pSnr + uc )  = ucComMF522Buf [ uc ];                        //读出UID
         ucSnr_check ^= ucComMF522Buf [ uc ];
      }

      if ( ucSnr_check != ucComMF522Buf [ uc ] )
                                cStatus = MI_ERR;
    }
    RC522_SetBit_Register ( CollReg, 0x80 );
    return cStatus;
}

/**
 * @brief   :用RC522计算CRC16（循环冗余校验）
        * @param  ：pIndata：计算CRC16的数组
 *            ucLen：计算CRC16的数组字节长度
 *            pOutData：存放计算结果存放的首地址
  * @retval ：状态值MI_OK，成功
*/
void CalulateCRC ( uint8_t * pIndata, u8 ucLen, uint8_t * pOutData ){
    uint8_t uc, ucN;

    RC522_ClearBit_Register(DivIrqReg,0x04);
    RC522_Write_Register(CommandReg,PCD_IDLE);
    RC522_SetBit_Register(FIFOLevelReg,0x80);

    for ( uc = 0; uc < ucLen; uc ++)
            RC522_Write_Register ( FIFODataReg, * ( pIndata + uc ) );

    RC522_Write_Register ( CommandReg, PCD_CALCCRC );

    uc = 0xFF;

    do
    {
        ucN = RC522_Read_Register ( DivIrqReg );
        uc --;
    } while ( ( uc != 0 ) && ! ( ucN & 0x04 ) );

    pOutData [ 0 ] = RC522_Read_Register ( CRCResultRegL );
    pOutData [ 1 ] = RC522_Read_Register ( CRCResultRegM );

}

/**
  * @brief   :选定卡片
  * @param  ：pSnr：卡片序列号，4字节
  * @retval ：状态值MI_OK，成功
*/
char PcdSelect ( uint8_t * pSnr ){
    char ucN;
    uint8_t uc;
          uint8_t ucComMF522Buf [ MAXRLEN ];
    uint32_t  ulLen;
    /* PICC_ANTICOLL1：防冲突命令 */
    ucComMF522Buf [ 0 ] = PICC_ANTICOLL1;
    ucComMF522Buf [ 1 ] = 0x70;
    ucComMF522Buf [ 6 ] = 0;

    for ( uc = 0; uc < 4; uc ++ )
    {
            ucComMF522Buf [ uc + 2 ] = * ( pSnr + uc );
            ucComMF522Buf [ 6 ] ^= * ( pSnr + uc );
    }

    CalulateCRC ( ucComMF522Buf, 7, & ucComMF522Buf [ 7 ] );

    RC522_ClearBit_Register ( Status2Reg, 0x08 );

    ucN = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 9, ucComMF522Buf, & ulLen );

    if ( ( ucN == MI_OK ) && ( ulLen == 0x18 ) )
      ucN = MI_OK;
    else
      ucN = MI_ERR;

    return ucN;

}

/**
  * @brief   :校验卡片密码
  * @param  ：ucAuth_mode：密码验证模式
  *                     = 0x60，验证A密钥
  *                     = 0x61，验证B密钥
  *           ucAddr：块地址
  *           pKey：密码
  *           pSnr：卡片序列号，4字节
  * @retval ：状态值MI_OK，成功
*/
char PcdAuthState ( uint8_t ucAuth_mode, uint8_t ucAddr, uint8_t * pKey, uint8_t * pSnr ){
    char cStatus;
          uint8_t uc, ucComMF522Buf [ MAXRLEN ];
    uint32_t ulLen;

    ucComMF522Buf [ 0 ] = ucAuth_mode;
    ucComMF522Buf [ 1 ] = ucAddr;
          /* 前俩字节存储验证模式和块地址，2~8字节存储密码（6个字节），8~14字节存储序列号 */
    for ( uc = 0; uc < 6; uc ++ )
            ucComMF522Buf [ uc + 2 ] = * ( pKey + uc );

    for ( uc = 0; uc < 6; uc ++ )
            ucComMF522Buf [ uc + 8 ] = * ( pSnr + uc );
    /* 进行冗余校验，14~16俩个字节存储校验结果 */
    cStatus = PcdComMF522 ( PCD_AUTHENT, ucComMF522Buf, 12, ucComMF522Buf, & ulLen );
          /* 判断验证是否成功 */
    if ( ( cStatus != MI_OK ) || ( ! ( RC522_Read_Register ( Status2Reg ) & 0x08 ) ) )
      cStatus = MI_ERR;

    return cStatus;

}

/**
  * @brief   :在M1卡的指定块地址写入指定数据
  * @param  ：ucAddr：块地址
  *           pData：写入的数据，16字节
  * @retval ：状态值MI_OK，成功
*/
char PcdWrite ( uint8_t ucAddr, uint8_t * pData ){
    char cStatus;
          uint8_t uc, ucComMF522Buf [ MAXRLEN ];
    uint32_t ulLen;

    ucComMF522Buf [ 0 ] = PICC_WRITE;//写块命令
    ucComMF522Buf [ 1 ] = ucAddr;//写块地址

          /* 进行循环冗余校验，将结果存储在& ucComMF522Buf [ 2 ] */
    CalulateCRC ( ucComMF522Buf, 2, & ucComMF522Buf [ 2 ] );

        /* PCD_TRANSCEIVE:发送并接收数据命令，通过RC522向卡片发送写块命令 */
    cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 4, ucComMF522Buf, & ulLen );

                /* 通过卡片返回的信息判断，RC522是否与卡片正常通信 */
    if ( ( cStatus != MI_OK ) || ( ulLen != 4 ) || ( ( ucComMF522Buf [ 0 ] & 0x0F ) != 0x0A ) )
      cStatus = MI_ERR;

    if ( cStatus == MI_OK )
    {
                        //memcpy(ucComMF522Buf, pData, 16);
                        /* 将要写入的16字节的数据，传入ucComMF522Buf数组中 */
      for ( uc = 0; uc < 16; uc ++ )
                          ucComMF522Buf [ uc ] = * ( pData + uc );
                        /* 冗余校验 */
      CalulateCRC ( ucComMF522Buf, 16, & ucComMF522Buf [ 16 ] );
      /* 通过RC522，将16字节数据包括2字节校验结果写入卡片中 */
      cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 18, ucComMF522Buf, & ulLen );
                        /* 判断写地址是否成功 */
                        if ( ( cStatus != MI_OK ) || ( ulLen != 4 ) || ( ( ucComMF522Buf [ 0 ] & 0x0F ) != 0x0A ) )
        cStatus = MI_ERR;
    }
    return cStatus;
}

/**
  * @brief   :读取M1卡的指定块地址的数据
  * @param  ：ucAddr：块地址
  *           pData：读出的数据，16字节
  * @retval ：状态值MI_OK，成功
*/
char PcdRead ( uint8_t ucAddr, uint8_t * pData ){
    char cStatus;
          uint8_t uc, ucComMF522Buf [ MAXRLEN ];
    uint32_t ulLen;

    ucComMF522Buf [ 0 ] = PICC_READ;
    ucComMF522Buf [ 1 ] = ucAddr;
          /* 冗余校验 */
    CalulateCRC ( ucComMF522Buf, 2, & ucComMF522Buf [ 2 ] );
    /* 通过RC522将命令传给卡片 */
    cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 4, ucComMF522Buf, & ulLen );

          /* 如果传输正常，将读取到的数据传入pData中 */
    if ( ( cStatus == MI_OK ) && ( ulLen == 0x90 ) )
    {
                        for ( uc = 0; uc < 16; uc ++ )
        * ( pData + uc ) = ucComMF522Buf [ uc ];
    }
    else
      cStatus = MI_ERR;

    return cStatus;

}

/**
  * @brief   :让卡片进入休眠模式
  * @param  ：无
  * @retval ：状态值MI_OK，成功
*/
char PcdHalt( void ){
        uint8_t ucComMF522Buf [ MAXRLEN ];
        uint32_t  ulLen;

  ucComMF522Buf [ 0 ] = PICC_HALT;
  ucComMF522Buf [ 1 ] = 0;

  CalulateCRC ( ucComMF522Buf, 2, & ucComMF522Buf [ 2 ] );
         PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 4, ucComMF522Buf, & ulLen );

  return MI_OK;

}    
==================================================================
// rc522.h
#ifndef _BSP_RC522_H
#define _BSP_RC522_H

#include "stm32f10x.h"

#ifndef u8
#define u8 uint8_t
#endif

#ifndef u16
#define u16 uint16_t
#endif

#ifndef u32
#define u32 uint32_t
#endif

#define RCC_GPIO        RCC_APB2Periph_GPIOA
#define PORT_GPIO       GPIOA
//SDA
#define GPIO_CS         GPIO_Pin_1
//SCK
#define GPIO_SCK        GPIO_Pin_2
//MOSI
#define GPIO_MOSI       GPIO_Pin_3
//RST
#define GPIO_RST        GPIO_Pin_5
//MISO
#define GPIO_MISO       GPIO_Pin_4

/* IO口操作函数 */
#define   RC522_CS_Enable()         GPIO_WriteBit(PORT_GPIO, GPIO_CS,Bit_RESET)
#define   RC522_CS_Disable()        GPIO_WriteBit(PORT_GPIO, GPIO_CS,Bit_SET)

#define   RC522_Reset_Enable()      GPIO_WriteBit(PORT_GPIO, GPIO_RST,Bit_RESET)
#define   RC522_Reset_Disable()     GPIO_WriteBit(PORT_GPIO, GPIO_RST,Bit_SET)

#define   RC522_SCK_0()             GPIO_WriteBit(PORT_GPIO, GPIO_SCK,Bit_RESET)
#define   RC522_SCK_1()             GPIO_WriteBit(PORT_GPIO, GPIO_SCK,Bit_SET)

#define   RC522_MOSI_0()            GPIO_WriteBit(PORT_GPIO, GPIO_MOSI,Bit_RESET)
#define   RC522_MOSI_1()            GPIO_WriteBit(PORT_GPIO, GPIO_MOSI,Bit_SET)

#define   RC522_MISO_GET()          GPIO_ReadInputDataBit(PORT_GPIO, GPIO_MISO)

//RC522命令字
#define PCD_IDLE              0x00               //取消当前命令
#define PCD_AUTHENT           0x0E               //验证密钥
#define PCD_RECEIVE           0x08               //接收数据
#define PCD_TRANSMIT          0x04               //发送数据
#define PCD_TRANSCEIVE        0x0C               //发送并接收数据
#define PCD_RESETPHASE        0x0F               //复位
#define PCD_CALCCRC           0x03               //CRC计算

//Mifare_One卡片命令字
#define PICC_REQIDL           0x26               //寻天线区内未进入休眠状态
#define PICC_REQALL           0x52               //寻天线区内全部卡
#define PICC_ANTICOLL1        0x93               //防冲撞
#define PICC_ANTICOLL2        0x95               //防冲撞
#define PICC_AUTHENT1A        0x60               //验证A密钥
#define PICC_AUTHENT1B        0x61               //验证B密钥
#define PICC_READ             0x30               //读块
#define PICC_WRITE            0xA0               //写块
#define PICC_DECREMENT        0xC0               //扣款
#define PICC_INCREMENT        0xC1               //充值
#define PICC_RESTORE          0xC2               //调块数据到缓冲区
#define PICC_TRANSFER         0xB0               //保存缓冲区中数据
#define PICC_HALT             0x50               //休眠

/* RC522  FIFO长度定义 */
#define DEF_FIFO_LENGTH       64                 //FIFO size=64byte
#define MAXRLEN  18

/* RC522寄存器定义 */
// PAGE 0
#define     RFU00                 0x00    //保留
#define     CommandReg            0x01    //启动和停止命令的执行
#define     ComIEnReg             0x02    //中断请求传递的使能（Enable/Disable）
#define     DivlEnReg             0x03    //中断请求传递的使能
#define     ComIrqReg             0x04    //包含中断请求标志
#define     DivIrqReg             0x05    //包含中断请求标志
#define     ErrorReg              0x06    //错误标志，指示执行的上个命令的错误状态
#define     Status1Reg            0x07    //包含通信的状态标识
#define     Status2Reg            0x08    //包含接收器和发送器的状态标志
#define     FIFODataReg           0x09    //64字节FIFO缓冲区的输入和输出
#define     FIFOLevelReg          0x0A    //指示FIFO中存储的字节数
#define     WaterLevelReg         0x0B    //定义FIFO下溢和上溢报警的FIFO深度
#define     ControlReg            0x0C    //不同的控制寄存器
#define     BitFramingReg         0x0D    //面向位的帧的调节
#define     CollReg               0x0E    //RF接口上检测到的第一个位冲突的位的位置
#define     RFU0F                 0x0F    //保留
// PAGE 1
#define     RFU10                 0x10    //保留
#define     ModeReg               0x11    //定义发送和接收的常用模式
#define     TxModeReg             0x12    //定义发送过程的数据传输速率
#define     RxModeReg             0x13    //定义接收过程中的数据传输速率
#define     TxControlReg          0x14    //控制天线驱动器管教TX1和TX2的逻辑特性
#define     TxAutoReg             0x15    //控制天线驱动器的设置
#define     TxSelReg              0x16    //选择天线驱动器的内部源
#define     RxSelReg              0x17    //选择内部的接收器设置
#define     RxThresholdReg        0x18    //选择位译码器的阈值
#define     DemodReg              0x19    //定义解调器的设置
#define     RFU1A                 0x1A    //保留
#define     RFU1B                 0x1B    //保留
#define     MifareReg             0x1C    //控制ISO 14443/MIFARE模式中106kbit/s的通信
#define     RFU1D                 0x1D    //保留
#define     RFU1E                 0x1E    //保留
#define     SerialSpeedReg        0x1F    //选择串行UART接口的速率
// PAGE 2
#define     RFU20                 0x20    //保留
#define     CRCResultRegM         0x21    //显示CRC计算的实际MSB值
#define     CRCResultRegL         0x22    //显示CRC计算的实际LSB值
#define     RFU23                 0x23    //保留
#define     ModWidthReg           0x24    //控制ModWidth的设置
#define     RFU25                 0x25    //保留
#define     RFCfgReg              0x26    //配置接收器增益
#define     GsNReg                0x27    //选择天线驱动器管脚（TX1和TX2）的调制电导
#define     CWGsCfgReg            0x28    //选择天线驱动器管脚的调制电导
#define     ModGsCfgReg           0x29    //选择天线驱动器管脚的调制电导
#define     TModeReg              0x2A    //定义内部定时器的设置
#define     TPrescalerReg         0x2B    //定义内部定时器的设置
#define     TReloadRegH           0x2C    //描述16位长的定时器重装值
#define     TReloadRegL           0x2D    //描述16位长的定时器重装值
#define     TCounterValueRegH     0x2E
#define     TCounterValueRegL     0x2F    //显示16位长的实际定时器值
// PAGE 3
#define     RFU30                 0x30    //保留
#define     TestSel1Reg           0x31    //常用测试信号配置
#define     TestSel2Reg           0x32    //常用测试信号配置和PRBS控制
#define     TestPinEnReg          0x33    //D1-D7输出驱动器的使能管脚（仅用于串行接口）
#define     TestPinValueReg       0x34    //定义D1-D7用作I/O总线时的值
#define     TestBusReg            0x35    //显示内部测试总线的状态
#define     AutoTestReg           0x36    //控制数字自测试
#define     VersionReg            0x37    //显示版本
#define     AnalogTestReg         0x38    //控制管脚AUX1和AUX2
#define     TestDAC1Reg           0x39    //定义TestDAC1的测试值
#define     TestDAC2Reg           0x3A    //定义TestDAC2的测试值
#define     TestADCReg            0x3B    //显示ADCI和Q通道的实际值
#define     RFU3C                 0x3C    //保留
#define     RFU3D                 0x3D    //保留
#define     RFU3E                 0x3E    //保留
#define     RFU3F                                          0x3F    //保留

/* 和RC522通信时返回的错误代码 */
#define         MI_OK                 0x26
#define         MI_NOTAGERR           0xcc
#define         MI_ERR                0xbb

/**********************************************************************/

void RC522_Init(void);/* IO口初始化 */
////////////////软件模拟SPI与RC522通信///////////////////////////////////////////
void RC522_SPI_SendByte( uint8_t byte );/* 软件模拟SPI发送一个字节数据，高位先行 */
uint8_t RC522_SPI_ReadByte( void );/* 软件模拟SPI读取一个字节数据，先读高位 */
uint8_t RC522_Read_Register( uint8_t Address );//读取RC522指定寄存器的值
void RC522_Write_Register( uint8_t Address, uint8_t data );//向RC522指定寄存器中写入指定的数据
void RC522_SetBit_Register( uint8_t Address, uint8_t mask );//置位RC522指定寄存器的指定位
void RC522_ClearBit_Register( uint8_t Address, uint8_t mask );//清位RC522指定寄存器的指定位
/////////////////////STM32对RC522的基础通信///////////////////////////////////
void RC522_Antenna_On( void );//开启天线
void RC522_Antenna_Off( void );//关闭天线
void RC522_Rese( void );//复位RC522
void RC522_Config_Type( char Type );//设置RC522的工作方式
/////////////////////////STM32控制RC522与M1的通信///////////////////////////////////////
char PcdComMF522 ( uint8_t ucCommand, uint8_t * pInData, uint8_t ucInLenByte, uint8_t * pOutData, uint32_t * pOutLenBit );//通过RC522和ISO14443卡通讯
char PcdRequest ( uint8_t ucReq_code, uint8_t * pTagType );//寻卡
char PcdAnticoll ( uint8_t * pSnr );//防冲突
void CalulateCRC ( uint8_t * pIndata, u8 ucLen, uint8_t * pOutData );//用RC522计算CRC16（循环冗余校验）
char PcdSelect ( uint8_t * pSnr );//选定卡片
char PcdAuthState ( uint8_t ucAuth_mode, uint8_t ucAddr, uint8_t * pKey, uint8_t * pSnr );//校验卡片密码
char PcdWrite ( uint8_t ucAddr, uint8_t * pData );//在M1卡的指定块地址写入指定数据
char PcdRead ( uint8_t ucAddr, uint8_t * pData );//读取M1卡的指定块地址的数据
char PcdHalt( void );//让卡片进入休眠模式

#endif
========================================================================
// main.c
#include "stm32f10x.h"
#include "board.h"
#include "bsp_uart.h"
#include "stdio.h"
#include "string.h"
#include "bsp_rc522.h"

/* 卡的ID存储，32位,4字节 */
u8 ucArray_ID [ 4 ];
uint8_t ucStatusReturn;    //返回状态

int main(void){
    int i = 0;

    uint8_t read_write_data[16]={0};//读写数据缓存
    uint8_t card_KEY[6] ={0xff,0xff,0xff,0xff,0xff,0xff};//默认密码

    board_init();
    uart1_init(115200U);

    printf ("Init....\r\n");
    RC522_Init( );//IC卡IO口初始化
    RC522_Rese( );//复位RC522
    printf ("Start!\r\n");

    while(1){

        /* 寻卡（方式：范围内全部），第一次寻卡失败后再进行一次，寻卡成功时卡片序列传入数组ucArray_ID中 */
        if ( ( ucStatusReturn = PcdRequest ( PICC_REQALL, ucArray_ID ) ) != MI_OK ){
                        ucStatusReturn = PcdRequest ( PICC_REQALL, ucArray_ID );
        }

        if ( ucStatusReturn == MI_OK  ){
                /* 防冲突操作，被选中的卡片序列传入数组ucArray_ID中 */
                if ( PcdAnticoll ( ucArray_ID ) == MI_OK ){
                        //输出卡ID
                        printf("ID: %X %X %X %X\r\n", ucArray_ID [ 0 ], ucArray_ID [ 1 ], ucArray_ID [ 2 ], ucArray_ID [ 3 ]);

                         //选卡
                        if( PcdSelect(ucArray_ID) != MI_OK ){    
                            printf("PcdSelect failure\r\n");         
                        }

                        //校验卡片密码
                        //数据块6的密码A进行校验（所有密码默认为16个字节的0xff）
                        if( PcdAuthState(PICC_AUTHENT1B, 6, card_KEY,  ucArray_ID) != MI_OK )
                        {    printf("PcdAuthState failure\r\n");      }

                        //往数据块4写入数据read_write_data
                        read_write_data[0] = 0xaa;//将read_write_data的第一位数据改为0xaa
                        if( PcdWrite(4,read_write_data) != MI_OK )
                        {    printf("PcdWrite failure\r\n");          }

                        //将read_write_data的16位数据，填充为0（清除数据的意思）
                        memset(read_write_data,0,16);
                        delay_us(8);

                        //读取数据块4的数据
                        if( PcdRead(4,read_write_data) != MI_OK )
                        {    printf("PcdRead failure\r\n");           }

                        //输出读出的数据
                        for( i = 0; i < 16; i++ )
                        {
                                printf("%x ",read_write_data[i]);
                        }
                        printf("\r\n");
                }
        }
    }

}
```



https://github.com/TooUpper/SensorDrive
