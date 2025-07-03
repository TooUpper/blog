---
title: "I2C"
date: 2024-11-09 09:05:00
categories: "ARM"
tags: 
- "GD32F407"
---

在嵌入式开发中，**I2C（Inter-Integrated Circuit）**是一种用于**短距离通信**的**半双工同步串行**通信**协议**，仅需两条线（数据线 SDA 和时钟线 SCL）即可实现**多主多从**通信。主设备通过发送**时钟信号**控制通信节奏，以**字节**为单位与从设备进行数据交换。每个从设备具有**唯一**地址，主设备可以通过寻址选择特定从设备进行读写。

## 主要特性

- **半双工通信**：I2C 使用两根信号线即可完成通信，具备半双工特性，即数据在同一时间内只能沿一个方向传输。

- **双线连接**：I2C 使用两条线进行通信：

  - **SDA（数据线）**：负责数据的双向传输。

  - **SCL（时钟线）**：提供同步信号，由主设备产生。

- **多主多从**：IIC 支持多主多从结构，多个主设备可以连接到同一条总线上，但同时只有一个主设备可以主动控制总线。

- **从设备地址**：每个从设备通过唯一的 7 位或 10 位地址被识别，主设备通过地址区分和选择要通信的从设备。

> SDA 和 SCL 都是双向线，当总线空闲时，两条线都是高电平；连接到总线的设备输出极必须是开漏或者开集，以提供线与功能。

## I2C 时序

IIC 的传输速率一般是 100kbps，最高可以达到 400kbps（快速模式）或更高（高速模式可达 3.4Mbps），并且遵循一定的时序约束：

- **时钟同步**：主设备通过 SCL 控制传输节奏，数据必须在 SCL 高电平时保持稳定，只有在 SCL 低电平期间数据才允许变化。
- **确认应答（ACK/NACK）**：每传输 8 位数据后，接收方在第九个时钟周期内向发送方反馈 ACK（低电平）或 NACK（高电平）以表示数据接收状态。

## I2C 通信信号

**起始条件**

- 主设备将 SDA 线从高电平拉至低电平，同时保持 SCL 线为高电平，表示通信开始。所有从设备都会检测到该起始信号，准备响应。

**从设备地址**

- 主设备在起始信号后发送从设备的7位地址（或10位地址，取决于协议版本）并在地址后加上 1 位读写位。
  - 读写位为 0 表示写操作（主设备向从设备发送数据），为 1 表示读操作（主设备从从设备读取数据）。

- 总线上的所有设备都接收地址信号，只有与地址匹配的从设备会响应。

**应答（ACK）**

- 匹配的从设备将 SDA 线拉低，表示“应答”（ACK），通知主设备可以进行下一步。若没有从设备应答，则 SDA 保持高电平（非应答，NACK）。

**数据传输**

- 数据以字节为单位传输，每字节包含 8 位。

- 每发送完一字节数据，从设备（写操作时）或主设备（读操作时）都需要发送一个 ACK。

- 数据传输过程中，SDA 数据线在 SCL 时钟线的高电平期间保持稳定，数据变化发生在低电平期间。

**停止条件**

- 当通信结束时，主设备将 SDA 从低电平拉至高电平，同时保持 SCL 为高电平，表示通信完成。

- 所有从设备返回待机模式，等待下一次通信。

## I2C 通信过程

### **I2C 写入过程：**

1. **起始信号（START）**
2. **发送从设备地址 + 写标志（0）**
3. **接收从设备的应答（ACK）**
4. **发送数据字节**
5. **接收从设备的应答（ACK）**
6. **重复步骤 4 和 5，直到发送完所有数据**
7. **发送停止信号（STOP）**

### **I2C 读取过程：**

1. **起始信号（START）**
2. **发送从设备地址 + 写标志（0）**
3. **接收从设备的应答（ACK）**
4. **起始信号（START）**
5. **发送从设备地址 + 读标志（1）**
6. **接收从设备的应答（ACK）**
7. **读取数据字节**
8. **主设备发送应答（ACK）或不应答（NACK）**
9. **重复步骤 7 和 8，直到读取完所有数据**
10. **发送停止信号（STOP）**

## I2C 通信规则

- **上升沿**：I2C 协议在 **SCL（时钟线）上升沿捕获数据**。当 SCL 线上升沿到来时，从设备或主设备读取当前 SDA（数据线）上的数据位。
- **高电平**：在 **SCL 高电平期间传输数据**。SDA 线上发送的数据位在 SCL 高电平时被稳定保持。
- **下降沿**：当 SCL 从高到低时，I²C 通信方可 **在 SDA 上准备下一位数据**。
- **低电平**：SCL 低电平期间用于 **设置下一个数据位**，主设备或从设备可以改变 SDA 线上的状态。

这样每次上升沿都捕获数据位，确保数据的稳定性，而下降沿或低电平则允许调整数据线上的数据。

## 时序图

![image-20241109111048626](C:\Users\kay\Desktop\image-20241109111048626.png)

![F407yihuofeimen](/public/image/Embedded/MCU/GD32/GD32F407/F407yihuofeimen.png)

## 电路图

![image-20241111211544630](C:\Users\kay\AppData\Roaming\Typora\typora-user-images\image-20241111211544630.png)

## 实现

### 软件实现

```c
#include "stm32f1xx_hal.h"

// 定义 SCL 和 SDA 所在的端口和引脚
#define I2C_SCL_PORT GPIOB
#define I2C_SCL_PIN  GPIO_PIN_6
#define I2C_SDA_PORT GPIOB
#define I2C_SDA_PIN  GPIO_PIN_7

// 定义 SCL 和 SDA 的操作宏
#define I2C_SCL_HIGH() HAL_GPIO_WritePin(I2C_SCL_PORT, I2C_SCL_PIN, GPIO_PIN_SET)
#define I2C_SCL_LOW()  HAL_GPIO_WritePin(I2C_SCL_PORT, I2C_SCL_PIN, GPIO_PIN_RESET)
#define I2C_SDA_HIGH() HAL_GPIO_WritePin(I2C_SDA_PORT, I2C_SDA_PIN, GPIO_PIN_SET)
#define I2C_SDA_LOW()  HAL_GPIO_WritePin(I2C_SDA_PORT, I2C_SDA_PIN, GPIO_PIN_RESET)

// 读取 SDA 状态
#define I2C_SDA_READ() HAL_GPIO_ReadPin(I2C_SDA_PORT, I2C_SDA_PIN)

// 延时函数
void I2C_Delay(void) {
    for (volatile int i = 0; i < 10; i++); // 调整延时以符合时序
}

// 起始信号
void I2C_Start(void) {
    I2C_SDA_HIGH();
    I2C_SCL_HIGH();
    I2C_Delay();
    I2C_SDA_LOW();  // SDA 从高到低，产生起始信号
    I2C_Delay();
    I2C_SCL_LOW();  // SCL 拉低，准备传输数据
}

// 停止信号
void I2C_Stop(void) {
    I2C_SCL_LOW();
    I2C_SDA_LOW();
    I2C_Delay();
    I2C_SCL_HIGH();  // SCL 拉高
    I2C_Delay();
    I2C_SDA_HIGH();  // SDA 从低到高，产生停止信号
    I2C_Delay();
}

// 发送应答信号
void I2C_SendNACK(void) {
    I2C_SCL_LOW();
    I2C_SDA_HIGH();  // SDA 拉高，表示非应答
    I2C_Delay();
    I2C_SCL_HIGH();  // SCL 拉高，发送 NACK
    I2C_Delay();
    I2C_SCL_LOW();
}

// 接收应答信号
int I2C_ReceiveACK(void) {
    I2C_SCL_LOW();
    I2C_SDA_HIGH();  // 释放 SDA，等待应答
    I2C_Delay();
    I2C_SCL_HIGH();  // SCL 拉高，接收应答信号
    int ack = (I2C_SDA_READ() == GPIO_PIN_RESET);  // SDA 低电平表示 ACK，高电平表示 NACK
    I2C_Delay();
    I2C_SCL_LOW();
    return ack;
}

// 发送字节
void I2C_SendByte(uint8_t data) {
    for (int i = 0; i < 8; i++) {
        I2C_SCL_LOW();
        if (data & 0x80) {
            I2C_SDA_HIGH();
        } else {
            I2C_SDA_LOW();
        }
        data <<= 1;
        I2C_Delay();
        I2C_SCL_HIGH();  // SCL 拉高，发送一位数据
        I2C_Delay();
    }
    I2C_SCL_LOW();  // 一字节发送完，拉低 SCL 准备接收 ACK
}


// 接收字节
uint8_t I2C_ReceiveByte(void) {
    uint8_t data = 0;
    I2C_SDA_HIGH();  // 释放 SDA，准备接收数据
    for (int i = 0; i < 8; i++) {
        data <<= 1;
        I2C_SCL_LOW();
        I2C_Delay();
        I2C_SCL_HIGH();  // 拉高 SCL，接收一位数据
        if (I2C_SDA_READ()) {
            data |= 0x01;
        }
        I2C_Delay();
    }
    I2C_SCL_LOW();
    return data;
}
```



### 硬件实现





















## 问题

一、**在软件时间时，高低电平的延时时间该怎么算**

在软件模拟 I²C 时，延时的设置取决于目标的通信速率。

**对于 100 kbps 的标准模式**：

- 每比特 10 微秒。
- 因此，高电平和低电平每次的延时各需要 **5 微秒**，从而实现 10 微秒的总周期。

**对于 400 kbps 的快速模式**：

- 每比特 2.5 微秒。
- 在这种情况下，高电平和低电平每次的延时各 **1.25 微秒**。

在 **I2C 通信协议** 中，传输 **一个 bit** 需要一个 **高电平**和**低电平**，这是因为 **SCL（时钟信号）** 是控制数据传输时序的时钟线，而 **SDA（数据线）** 是实际传输数据的线。每传输一个 bit，**SCL 的时钟周期**（一个高电平和一个低电平）就用来完成数据位的传输。

二、**I2C 协议为什么要专门指定为开漏输出**

可能会出现这么一种情况，主设备将数据线拉高，从设备将数据线拉低，这时候如何是推挽则会造成短路，这是电路中绝不允许出现的情况，而开漏则不会有这种问题；因为他只能输出低电平，高电平要靠外部的上拉电阻进行提供；

**避免电平冲突**：主设备和从设备都可以安全地拉低总线电平，而不会出现高低电平冲突的情况。

**总线仲裁**：虽然单主单从不需要仲裁，但在通信中如果出现错误或异常，开漏设计可以让设备可靠地检测总线状态，并进行恢复。

**方便扩展**：即使现在是单主单从，未来可以轻松扩展成多主或多从系统，而不需要修改硬件设计。
