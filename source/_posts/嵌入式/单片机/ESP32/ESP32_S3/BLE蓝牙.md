---
title: "BLE蓝牙"
date: 2024-12-10 19:13:00
categories: "ESP32"
tags: 
- "ESP32-S3"
---

BLE（Bluetooth Low Energy，蓝牙低功耗）是一种无线技术标准，旨在提供低功耗和低延迟的无线通信。它是蓝牙技术的一种变体，特别适用于对电池寿命和功耗有严格要求的应用。

**BLE 的特点**

- 低功耗：BLE 设备在待机状态下消耗极少的电量，适合长时间运行的设备。
- 短距离通信：典型的通信范围为 10 米至 100 米，具体取决于环境和设备。
- 快速连接：BLE 设备能够快速连接和断开，适合需要快速响应的应用。
- 多设备连接：支持多个设备同时连接，适合物联网（IoT）场景。

## 协议栈

BLE 蓝牙协议栈是蓝牙设备之间通信的基础架构。它定义了数据的传输方式、设备的发现机制、连接过程以及应用层的服务交互。蓝牙协议栈通常分为多个层级，每个层次实现不同的功能，确保设备之间的可靠通信。以下是BLE协议栈的一个大致分层架构：

**物理层（Physical Layer, PHY）**

物理层是 BLE 协议栈的最低层，定义了如何通过无线电波传输数据。BLE 使用的频段为 2.4 GHz ISM（工业、科学和医学）频段，具体划分为 40 个通道（其中 3 个是用于广告数据的）。每个通道的带宽为1 MHz，传输速度最高为 1 Mbps。

- 传输模式：基于频率跳跃的扩频技术（FHSS），避免干扰。
- 信号编码：采用GFSK（Gaussian Frequency Shift Keying）调制技术，支持较低的功耗。

**链路层（Link Layer）**

链路层负责数据包的格式、信道管理、设备发现、连接建立和连接维持等任务。它处理 BLE 设备之间的基础通信，负责信号的发送、接收和处理低层数据。

- 设备发现：BLE 设备会广播自己的存在，其他设备可以扫描这些广告包来发现目标设备。
- 连接管理：链路层还负责连接的建立、断开、连接参数的协商和维护。
- 分包与重传：链路层实现了数据包的分割和重传机制，确保数据的完整传输。

**控制器层（Controller Layer）**

控制器层通常包括物理层和链路层，它负责所有与硬件直接交互的功能。在一些高端应用中，控制器层可能是单独的硬件模块，负责处理BLE 信号的传输和接收。控制器层和主机层通过 HCI（Host Controller Interface）进行通信。

**主机层（Host Layer）**⭐

主机层位于协议栈的上方，负责处理高层应用和管理设备的连接、服务发现、数据传输等任务。主机层实现了以下几个关键协议：

- GAP（Generic Access Profile）：负责设备间的发现、连接管理以及角色定义。GAP 协议定义了设备如何通过广播进行自我公开、如何扫描并建立连接，以及如何处理设备的连接和断开。它是 BLE 协议栈中管理设备访问行为的关键协议，确保设备能够在蓝牙网络中进行有效的通信。

- L2CAP（Logical Link Control and Adaptation Protocol）：提供数据分段、组装和传输的功能，允许多个应用层协议共享底层链路。
- ATT（Attribute Protocol）：用于设备间的属性数据交换。ATT 定义了如何存取蓝牙设备的各种数据（例如，传感器数据），通过服务和特性进行访问。
- GATT（Generic Attribute Profile）：基于 ATT 协议之上的高层协议，定义了 BLE 设备之间数据交互的标准化方式。GATT 使用“服务”和“特性”模型，设备通过特性（characteristic）交换数据。

**应用层（Application Layer）**

应用层是 BLE 协议栈的最高层，负责根据特定的应用需求处理业务逻辑。开发者可以在这一层设计 BLE 通信的具体行为，如控制设备、传输数据或处理用户输入等。应用层主要通过 GATT 协议进行操作，可以定义特定的服务和特性。例如：

- 心率监测器服务（Heart Rate Service）
- 电池服务（Battery Service）
- 自定义服务（例如，温度传感器）

此外，应用层也可以使用以下协议进行更高级的功能：

- SM（Security Manager）：用于加密和认证设备间的通信，确保数据的安全性。
- L2CAP：提供面向连接的传输服务。

6. 安全管理层（Security Manager, SM）

安全管理层负责对设备进行配对、身份验证和加密。BLE 设备可以通过配对建立加密连接，以确保数据在传输过程中的机密性和完整性。常见的安全机制包括：

- 身份验证：通过 PIN 码、键盘、显示器或其他方式进行身份认证。
- 加密：采用 AES 加密算法保证数据的安全。
- 隐私：BLE 设备支持地址隐私，以避免设备被追踪。

**在蓝牙通信过程中，GAP（Generic Access Profile）和 GATT（Generic Attribute Profile）是两个核心协议。二者又分别依赖于 ATT（Attribute Protocol）和 L2CAP（Logical Link Control and Adaptation Protocol）协议。**

- **GAP**：管理设备的广播、扫描、连接建立和断开，决定设备的通信角色（中心、外围、广播者、观察者）。它是所有蓝牙连接的基础。
- **GATT**：依赖 GAP 协议建立的连接，负责数据的标准化交互。它基于 ATT 协议，提供数据读取、写入等功能，使用“服务”和“特性”的模型进行数据传输。GATT 将 ATT 中 attribute 的内容分为服务、特征、值或描述。

![GPAxieyijiagou](/public/image/嵌入式/MCU/ESP32/ESP32S3/GPAxieyijiagou.png)

> **这张图很好地展示了两者的关系：**
>
> 1. **GAP** 位于较高的逻辑层，主要处理设备发现、连接和访问角色管理。它依赖于下层协议（如L2CAP）来完成数据传输的支持。
> 2. **GATT** 基于 ATT 协议，用于具体的数据交互。它在 GAP 建立的连接基础上工作，服务于应用数据的传输。

### GAP 协议

GAP（Generic Access Profile）是蓝牙低功耗（BLE）协议栈的一部分，属于主机层（Host Layer）。它定义了**设备如何发现彼此、如何连接、如何广播信息以及如何设置通信角色。**

**主要特点：**

GAP 协议定义了 BLE 设备的四种角色，角色的选择决定了设备在通信中的行为。

- 设备角色：GAP定义了设备在BLE通信中的角色，包括：

  - 广播设备（Advertiser）：发送广播包以进行设备发现。

  - 扫描设备（Scanner）：监听并响应广播包，发现其他设备。

  - 主设备（Central）：发起连接的设备（通常是手机或主控制器）。

  - 从设备（Peripheral）：被连接的设备（如传感器、穿戴设备）。

- 设备发现：GAP定义了设备如何通过广播和扫描发现其他设备。设备可以处于可发现模式或不可发现模式。
- 连接管理：GAP负责建立、管理和终止连接。它处理连接间隔、超时和其他连接参数。

**应用场景：**

GAP 广泛应用于各种 BLE 设备之间的发现和连接过程，确保设备能够有效地找到并连接到彼此。

### GATT协议

GATT（Generic Attribute Profile）位于蓝牙低功耗（BLE）协议栈中的主机层（Host Layer），位于ATT（Attribute Protocol）之上。
它定义了设备之间如何交换数据和互动的规则，并使用“服务”和“特性”这种方式来规范通信，让不同设备可以按照统一的标准进行数据传输和操作。

GATT 定义了一个框架，将 ATT 的属性结构组织起来，分为服务、特征、值或描述。

**主要特点：**

- 服务和特性：GATT 基于服务和特性构建，服务是一组相关的特性。每个服务和特性都有唯一的UUID标识符。

- 数据模型：GATT 使用层次结构的数据模型，设备通过服务和特性与其他设备进行交互。

- 数据传输方式：GATT 支持读取、写入、通知和指示等操作：

  - 读取（Read）：客户端可以请求服务器读取特性值。

  - 写入（Write）：客户端可以向服务器写入特性值。

  - 通知（Notify）：服务器可以主动向客户端发送特性值的变化。

  - 指示（Indicate）：服务器向客户端发送特性值的变化，并等待客户端的确认。

**应用场景：**

GATT通常用于传感器、健康设备、智能家居设备等场景，如心率监测器、温度传感器等。这些设备通过GATT服务和特性与其他BLE设备（如手机应用）进行数据交换。

### ATT协议

**ATT**（**Attribute Protocol**）其实就是一种**蓝牙设备间交换数据的规则**。在蓝牙低能耗（BLE）设备之间通信时，数据通常是以“属性”（Attribute）的形式进行组织和交换的。ATT 协议就是管理这些数据的“规则本”，它规定了如何读取、写入或者修改这些属性。

ATT 协议中的**属性**就是数据的基本单位。每个属性都有一个“名字”（也就是**UUID**，可以理解为“标签”）和一些**数据**。

**属性的构成**

![bleattxy](/public/image/嵌入式/MCU/ESP32/ESP32S3/bleattxy.png)

- **属性句柄（Attribute Handle）**：唯一标识属性的编号，用于访问属性。

- **属性类型（Attribute Type）**：标识属性的种类（如电池电量、心率等也就是我们说的 UUID）。

- **属性值（Attribute Value）**：存储实际的数据（如电池百分比、温度值等）。

- **属性权限（Attribute Permissions）**：定义对属性的访问权限（如可读、可写、通知等）。



[UUID 查询表](http://www.corenchip.com/2280.html)

### 总结🙃🙃🙃

- **GAP**：负责设备发现和连接管理，定义设备的角色和连接过程。
- **GATT**：负责设备之间的数据交换和数据组织，通过服务和特性来定义如何读取、写入和通知数据。
- **ATT**：负责在设备之间传输**具体的数据单元（属性）**，通过定义**属性（Attribute）**来组织和传输数据。每个属性都有一个唯一的标识符（UUID）和数据值。
- **L2CAP**：负责**数据封装和分段**，实现应用层和传输层之间的可靠数据传输。

这两个协议相辅相成，构成了蓝牙低功耗通信的基础，使得不同设备能够高效地进行数据交换和管理。

## 广播数据包格式

在 BLE 中，GAP 协议通过广播数据包的形式与其他设备进行交互。广播数据包的主要作用是：

- **设备发现**：向周围设备提供设备的身份信息（如设备名称、UUID 等），使其他设备能够快速发现并识别蓝牙设备。
- **服务信息传递**：传输设备支持的服务类型和功能。
- **连接准备**：当其他设备响应广播并发起连接请求后，可建立 BLE 连接。

广播数据包的有效负载（Payload）最大为 **37 字节**，前 6 个字节为 MAC 地址，后 **31 字节** 可供开发者使用。这 31 字节被分为一个或多个 **AD Structure**（广播数据结构），每个 AD Structure 包含以下部分：

1. **长度字段（Length）**：1 字节，表示该结构的总长度。
2. **类型字段（Type）**：1 字节，定义数据的类型（如设备名称、服务 UUID、设备类别等）。
3. **值字段（Value）**：可变长度，包含具体的数据内容。

广播数据包不直接发起连接请求，而是通过提供设备信息，等待中心设备（Scanner）发起连接请求。一旦中心设备响应广播包并发起连接请求，双方可以建立 BLE 连接。

值得注意的是，设备地址（6 字节）用于标识设备，但不占用广播数据包的 31 字节有效负载空间。通过合理设计广播数据内容，可以实现快速设备发现和低功耗的连接准备。

![ADStructuregeshi](/public/image/嵌入式/MCU/ESP32/ESP32S3/ADStructuregeshi.png)

> 每个AD Structure包含3部分内容，分别是：
>
> 1. Length(1字节):        广播数据包的长度
> 2. AD Type(1字节)：   广播的类型
> 3. AD Data（n字节）: 数据

**常见的广播类型如下**

![blegblx](/public/image/嵌入式/MCU/ESP32/ESP32S3/blegblx.png)

假设我此时有如下一组广播数据：

```c
// 假设这是一组广播数据
02 01 06 09 08 54 6F 6F 55 70 70 65 72 03 19 C1 03
// 我们将它按照 AD structure 拆分后为：
02 01 06	// 02 表示数据长度，01 表示广播类型为设备标识 06是具体的标识内容   
09 08 54 6F 6F 55 70 70 65 72 // 同理 09 表示长度，08 表示广播类型，后面就是类型的具体数据
03 19 C1 03 // 03 表示长度，19 表示类型，后面表示具体的数据    
```

> 注意：
>
> 在设置广播的设备外观时候，要采用低位先行的方式，比如我们要设备外观为键盘，键盘对应的数据位 0x03C1，那么我们在广播数据中填写的值就要是 03 19 C1 03（这里要采用低位先行的策略）

其他的设备外观数据我们可以在蓝牙官方的[已分配编号](https://www.bluetooth.com/wp-content/uploads/Files/Specification/HTML/Assigned_Numbers/out/en/Assigned_Numbers.pdf)表中进行查找

## 实现

### 蓝牙通信中的角色

主机（客户端）：可以发起扫描并主动连接从机设备的角色（发起连接），比如手机；

从机（服务端）：发起广播，等待主设备的连接（也可以被别人连接），比如蓝牙灯、手环；

> 同一个 BLE 设备既可以作为主机也可以作为从机

### BLE通信信道

BLE 蓝牙使用使用 2.4GHz 频道，一共有 40 个信道，37、38、39是蓝牙广播信道，剩余 37 个是数据信道。

![bletxxd](/public/image/嵌入式/MCU/ESP32/ESP32S3/bletxxd.png)

### 广播间隔和广播事件

**广播间隔**：从机每经过一个时间间隔发送一次广播数据，这个时间间隔称为广播间隔(20ms 到 10.24s)

**广播事件**：一次广播的动作，每次广播事件会在37，38，39信道上依次广播

![blelygbjg](/public/image/嵌入式/MCU/ESP32/ESP32S3/blelygbjg.png)

### BLE连接后

主机扫描到从机设备后，向从机设备发起连接请求；并且得到回应后双方才正式开始通信。此时有几个概念要注意下

**连接事件**：在 BLE 连接中使用跳频方案，两个设备在特定时间、频道上彼此发送和接收数据。这些设备稍后在新的通道上通过约定的时间相遇，这次用于收发数据的相遇称为连接事件。

**连接间隔**：两次连接事件之间的时间间隔称为连接间隔。1.25ms 为单位，范围从最小值 7.5ms 到最大值 4.0s.

**从机延迟**：可以跳过的最大事件数。从设备（比如一个心率监测器）在连接事件中跳过的最大次数。如果从设备的任务没有及时完成，它可以选择跳过某些连接事件，以节省电池。这个延迟值通常是通过连接参数来配置的，最大值通常可以达到 500。

**监控超时**：两次成功连接事件之间的最长时间。如果在此时间内没有成功的连接事件，设备将终止连接并返回到未连接状态。(100ms-32秒)。

**有效连接间隔**：实际有效的交互通信间隔，有效连接间隔 = 连接间隔 * (1+从机延迟)。

### 代码

ESP32 广播蓝牙，手机可以使用APP进行连接并且发送和接收数据，可以使用”BLE调试助手“进行测试。

```c
// ble.h
#ifndef _BLE_H_
#define _BLE_H_
#include "esp_err.h"

/**
 * 初始化并启动蓝牙BLE
 * @param 无
 * @return 是否成功
 */
esp_err_t ble_cfg_net_init(void);

/**
 * 设置特征1的值
 * @param value 值
 * @return 无
 */
void ble_set_ch1_value(uint16_t value);

/**
 * 设置特征2的值
 * @param value 值
 * @return 无
 */
void ble_set_ch2_value(uint16_t value);

#endif
=============================================================
// ble.c    
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/event_groups.h"
#include "esp_system.h"
#include "esp_log.h"
#include "esp_bt.h"

#include "esp_gap_ble_api.h"
#include "esp_gatts_api.h"
#include "esp_bt_defs.h"
#include "esp_bt_main.h"
#include "esp_gatt_common_api.h"
#include "nvs_flash.h"

#define TAG "BLE_CFG"

//设备名称
#define BLE_DEVICE_NAME "ESP32-HOME"

#define ESP_APP_ID 0x55

#define SVC_IND_ID1 0
#define SVC_IND_ID2 1

//蓝牙模块
enum{
    //服务1
    SV1_IDX_SVC,

    //特征1
    SV1_CH1_IDX_CHAR,
    SV1_CH1_IDX_CHAR_VAL,
    SV1_CH1_IDX_CHAR_CFG,

    //特征2
    SV1_CH2_IDX_CHAR,
    SV1_CH2_IDX_CHAR_VAL,
    SV1_CH2_IDX_CHAR_CFG,

    SV1_IDX_NB,
};

enum{
    //服务2
    SV2_IDX_SVC,

    //特征1
    SV2_CH1_IDX_CHAR,
    SV2_CH1_IDX_CHAR_VAL,
    SV2_CH1_IDX_CHAR_CFG,

    //特征2
    SV2_CH2_IDX_CHAR,
    SV2_CH2_IDX_CHAR_VAL,
    SV2_CH2_IDX_CHAR_CFG,

    SV2_IDX_NB,
};

static const uint16_t GATTS_SERVICE_UUID_TEST       = 0x18FF;   //自定义服务1
static const uint16_t GATTS_CHAR_UUID_CH1           = 0x2AFE;   //特征1 UUID
static const uint16_t GATTS_CHAR_UUID_CH2           = 0x2AFF;   //特征2 UUID

static const uint16_t GATTS_SERVICE2_UUID_TEST       = 0x18F3;   //自定义服务2
static const uint16_t GATTS_CHAR2_UUID_CH1           = 0x2AF1;   //特征1 UUID
static const uint16_t GATTS_CHAR2_UUID_CH2           = 0x2AF2;   //特征2 UUID

//主要服务声明UUID 0x2800
static const uint16_t primary_service_uuid         = ESP_GATT_UUID_PRI_SERVICE;

//次要服务声明UUID 0x2801
//static const uint16_t second_service_uuid          = ESP_GATT_UUID_SEC_SERVICE;

//特征声明UUID 0x2803
static const uint16_t character_declaration_uuid   = ESP_GATT_UUID_CHAR_DECLARE;

//特征描述UUID 0x2902
static const uint16_t character_client_config_uuid = ESP_GATT_UUID_CHAR_CLIENT_CONFIG;
//读权限
//static const uint8_t char_prop_read                =  ESP_GATT_CHAR_PROP_BIT_READ;
//写权限
//static const uint8_t char_prop_write               = ESP_GATT_CHAR_PROP_BIT_WRITE;
//读写、通知权限
static const uint8_t char_prop_read_write_notify   = ESP_GATT_CHAR_PROP_BIT_WRITE | ESP_GATT_CHAR_PROP_BIT_READ | ESP_GATT_CHAR_PROP_BIT_NOTIFY;
//读、通知权限
static const uint8_t char_prop_read_notify = ESP_GATT_CHAR_PROP_BIT_READ | ESP_GATT_CHAR_PROP_BIT_NOTIFY;
//特征1客户端特征配置
static uint8_t sv1_ch1_client_cfg[2] = {0x00, 0x00};
//特征2客户端特征配置
static uint8_t sv1_ch2_client_cfg[2]  = {0x00,0x00};
//特征1客户端特征配置
static uint8_t sv2_ch1_client_cfg[2] = {0x00, 0x00};
//特征2客户端特征配置
static uint8_t sv2_ch2_client_cfg[2]  = {0x00,0x00};

//gatt的访问接口，一个Profile（APP）对应1个
static uint16_t gl_gatts_if = ESP_GATT_IF_NONE;
//连接ID，连接成功后
static uint16_t gl_conn_id = 0xFFFF;

//char1的值
static char sv1_char1_value[2] = {0x00,0x16};
//char2的值
static char sv1_char2_value[2] = {0x00,0x25};

//char1的值
static char sv2_char1_value[2] = {0x00,0xAA};
//char2的值
static char sv2_char2_value[2] = {0x00,0xEE};

//att的handle表
uint16_t sv1_handle_table[SV1_IDX_NB];

//att的handle表
uint16_t sv2_handle_table[SV2_IDX_NB];

// 该标志位用于跟踪广播配置状态，
static uint8_t adv_config_done       = 0;

// ADV_CONFIG_FLAG 设置广播名称成功标志位
// SCAN_RSP_CONFIG_FLAG 设置扫描回复数据成功标志位
#define ADV_CONFIG_FLAG             (1 << 0)
#define SCAN_RSP_CONFIG_FLAG        (1 << 1)

//广播参数
static esp_ble_adv_params_t adv_params = {
    .adv_int_min       = 0x20,                //最小广播间隔，单位:0.625ms
    .adv_int_max       = 0x40,                //最大广播间隔，单位:0.625ms
    .adv_type          = ADV_TYPE_IND,        //广播类型(可连接的非定向广播)
    .own_addr_type     = BLE_ADDR_TYPE_PUBLIC,//使用固定地址广播
    .channel_map       = ADV_CHNL_ALL,        //在37、38、39信道进行广播
    .adv_filter_policy = ADV_FILTER_ALLOW_SCAN_ANY_CON_ANY,   //广播过滤策略（接收任何scan和任何连接）
};

//gatt描述表
// 在ESP32中一张表只能对应一个服务，可以有多个特征
static const esp_gatts_attr_db_t gatt1_db[SV1_IDX_NB] =
{
    //服务声明
    // ESP_GATT_AUTO_RSP：表示对这个属性的响应由 ESP32 自动处理，不需要应用层干预。
    // 权限（ESP_GATT_PERM_READ）：表示该服务的声明可以被读取。
    // ESP_UUID_LEN_16 UUID 长度
    // (uint8_t *)&primary_service_uuid:将服务标识为主要服务的 UUID (0x2800)
    // 大小：第一个 sizeof(uint16_t) 表示 UUID 的最大程度
    // 第二个 sizeof(GATTS_SERVICE_UUID_TEST) 表示服务 UUID 的实际大小。
    // GATTS_SERVICE_UUID_TEST 是服务的实际 UUID。
    [SV1_IDX_SVC]        =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&primary_service_uuid, ESP_GATT_PERM_READ,
      sizeof(uint16_t), sizeof(GATTS_SERVICE_UUID_TEST), (uint8_t *)&GATTS_SERVICE_UUID_TEST}},

    //特征1
    //特征声明
    // ESP_GATT_PERM_READ 这是特征声明的权限，表示特征声明本身是只读的。
    // char_prop_read_write_notify 定义了该特征的属性（即允许读、写、通知）。
    // 这是特征的属性（Properties），表示特征值支持的操作类型（读、写、通知等）。
    [SV1_CH1_IDX_CHAR]     =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&character_declaration_uuid, ESP_GATT_PERM_READ,
      sizeof(uint8_t), sizeof(uint8_t), (uint8_t *)&char_prop_read_write_notify}},   //特征值1允许读写和通知
    //特征值
    // ESP_GATT_RSP_BY_APP 对写/读操作的响应应由应用程序处理。
    // 也就是我们需要自己写函数进行调用回复
    // GATTS_CHAR_UUID_CH1 自定义特征1的UUID
    // 该特征值可以被读取和写入。
    // 大小：sizeof(sv1_char1_value) 表示该特征值的长度
    // 数据内容存储在 sv1_char1_value 变量中。
    [SV1_CH1_IDX_CHAR_VAL] =
    {{ESP_GATT_RSP_BY_APP}, {ESP_UUID_LEN_16, (uint8_t *)&GATTS_CHAR_UUID_CH1, ESP_GATT_PERM_READ | ESP_GATT_PERM_WRITE, 
      sizeof(sv1_char1_value), sizeof(sv1_char1_value), (uint8_t *)sv1_char1_value}},
    //特征描述->客户端特征配置
    // 数据内容存储在 sv1_ch1_client_cfg 变量中。
    [SV1_CH1_IDX_CHAR_CFG]  =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&character_client_config_uuid, ESP_GATT_PERM_READ|ESP_GATT_PERM_WRITE,
      sizeof(uint16_t), sizeof(sv1_ch1_client_cfg), (uint8_t *)sv1_ch1_client_cfg}},

    //特征2
    //特征声明
    [SV1_CH2_IDX_CHAR]      =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&character_declaration_uuid, ESP_GATT_PERM_READ,
      sizeof(uint8_t), sizeof(uint8_t), (uint8_t *)&char_prop_read_notify}},  //特征值2只允许读和通知

    //特征值
    [SV1_CH2_IDX_CHAR_VAL]  =
    {{ESP_GATT_RSP_BY_APP}, {ESP_UUID_LEN_16, (uint8_t *)&GATTS_CHAR_UUID_CH2, ESP_GATT_PERM_READ,
      sizeof(sv1_char2_value), sizeof(sv1_char2_value), (uint8_t *)sv1_char2_value}},

    //特征描述->客户端特征配置
    [SV1_CH2_IDX_CHAR_CFG]      =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&character_client_config_uuid, ESP_GATT_PERM_READ|ESP_GATT_PERM_WRITE,
      sizeof(uint16_t), sizeof(sv1_ch2_client_cfg), (uint8_t *)&sv1_ch2_client_cfg}},
};

//gatt描述表
static const esp_gatts_attr_db_t gatt2_db[SV1_IDX_NB] = 
{
    //服务2声明
    [SV2_IDX_SVC]        =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&primary_service_uuid, ESP_GATT_PERM_READ,
      sizeof(uint16_t), sizeof(GATTS_SERVICE2_UUID_TEST), (uint8_t *)&GATTS_SERVICE2_UUID_TEST}},

    //特征1
    //特征声明
    [SV2_CH1_IDX_CHAR]     =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&character_declaration_uuid, ESP_GATT_PERM_READ,
      sizeof(uint8_t), sizeof(uint8_t), (uint8_t *)&char_prop_read_notify}},
    //特征值
    [SV2_CH1_IDX_CHAR_VAL] =
    {{ESP_GATT_RSP_BY_APP}, {ESP_UUID_LEN_16, (uint8_t *)&GATTS_CHAR2_UUID_CH1, ESP_GATT_PERM_READ, 
      sizeof(sv2_char1_value), sizeof(sv2_char1_value), (uint8_t *)sv2_char1_value}},
    //特征描述->客户端特征配置
    [SV2_CH1_IDX_CHAR_CFG]  =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&character_client_config_uuid, ESP_GATT_PERM_READ|ESP_GATT_PERM_WRITE,
      sizeof(uint16_t), sizeof(sv2_ch1_client_cfg), (uint8_t *)sv2_ch1_client_cfg}},

    //特征2
    //特征声明
    [SV2_CH2_IDX_CHAR]      =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&character_declaration_uuid, ESP_GATT_PERM_READ,
      sizeof(uint8_t), sizeof(uint8_t), (uint8_t *)&char_prop_read_write_notify}},

    //特征值
    [SV2_CH2_IDX_CHAR_VAL]  =
    {{ESP_GATT_RSP_BY_APP}, {ESP_UUID_LEN_16, (uint8_t *)&GATTS_CHAR2_UUID_CH2, ESP_GATT_PERM_READ | ESP_GATT_PERM_WRITE,
      sizeof(sv2_char2_value), sizeof(sv2_char2_value), (uint8_t *)sv2_char2_value}},

    //特征描述->客户端特征配置
    [SV2_CH2_IDX_CHAR_CFG]      =
    {{ESP_GATT_AUTO_RSP}, {ESP_UUID_LEN_16, (uint8_t *)&character_client_config_uuid, ESP_GATT_PERM_READ|ESP_GATT_PERM_WRITE,
      sizeof(uint16_t), sizeof(sv2_ch2_client_cfg), (uint8_t *)&sv2_ch2_client_cfg}},
};

// 该数组的目的是存储一个 128 位的 UUID，
// 但由于 BLE 广播数据通常使用 16 位或 32 位的 UUID 来节省空间，我们只在数组中包括了实际需要的部分。
// 它实际上包含了一个16位（2字节）服务 UUID 和其它一些与设备相关的数据。0x000018FF
// GATTS_SERVICE_UUID_TEST 是自定义的UUID
// GATTS_SERVICE_UUID_TEST&0xff 获取 UUID 的低字节。
// (GATTS_SERVICE_UUID_TEST>>8)&0xff 获取 UUID 的高字节。
static uint8_t adv_service_uuid16[] = {
    /* LSB <------------------------------> MSB */
    0xfb, 0x34, 0x9b, 0x5f, 0x80, 0x00, 0x00, 0x80, 0x00, 0x10, 0x00, 0x00, 
    GATTS_SERVICE_UUID_TEST&0xff,(GATTS_SERVICE_UUID_TEST>>8)&0xff, 0x00, 0x00,
};

// The length of adv data must be less than 31 bytes
//static uint8_t test_manufacturer[TEST_MANUFACTURER_DATA_LEN] =  {0x12, 0x23, 0x45, 0x56};
//adv data广播数据
static esp_ble_adv_data_t adv_data = {
    // 如果设置为 true，设备会在接收到扫描请求后，回复包含更多信息的数据；
    // 如果设置为 false，则不返回扫描响应。
    .set_scan_rsp = false, //此广播数据是否启用扫描响应，
    // 如果设置为 true，设备广播时会包含其设备名称，这样其他设备可以看到设备的名字；
    // 如果设置为 false，则不包含设备名称。
    .include_name = true, //是否包含名字
    // 发射功率表示设备的信号强度，其他设备可以使用此信息来估算与该设备的距离。
    // 如果设置为 true，则会广播发射功率；如果设置为 false，则不广播。
    .include_txpower = false, //是否包含发射功率
    // 控制设备在连接时，设备之间交换数据的频率。
    .min_interval = 0x0006, //最小连接间隔 单位1.25ms
    .max_interval = 0x0010, //最大连接间隔 单位1.25ms
    // 设置设备的外观，这是一个标准化的值，设备可以根据此值告诉其他设备它是什么类型。
    .appearance = 0x00,     //apperance
    .manufacturer_len = 0, //厂商信息长度
    .p_manufacturer_data =  NULL, //厂商信息
    // 这个指针指向具体的服务数据内容。
    .service_data_len = 0,      //服务数据长度
    .p_service_data = NULL,     //服务数据
    // 服务 UUID（统一唯一标识符）用于标识 BLE 服务的类型。
    .service_uuid_len = sizeof(adv_service_uuid16),    //服务UUID长度
    .p_service_uuid = adv_service_uuid16,              //服务UUID
    .flag = (ESP_BLE_ADV_FLAG_GEN_DISC | ESP_BLE_ADV_FLAG_BREDR_NOT_SPT),//普通发现模式|不支持EDR经典蓝牙
};

//扫描回复数据，用于主机在主动扫描时，向设备发起扫描请求，设备需要回复的内容
static esp_ble_adv_data_t scan_rsp_data = {
    // true 表示开启扫描回复数据，设备将回复扫描请求的主机。
    .set_scan_rsp = true,
    // 是否在扫描回复数据中包含设备的名字。
    .include_name = true,
    // 是否在扫描回复数据中包含设备的发射功率。
    .include_txpower = true,
    // 设备的外观
    .appearance = 0x00,
    // 厂商数据的长度与内容
    .manufacturer_len = 0, //TEST_MANUFACTURER_DATA_LEN,
    .p_manufacturer_data =  NULL, //&test_manufacturer[0],
    // 指定服务数据的长度与内容
    .service_data_len = 0,
    .p_service_data = NULL,
    // 指定服务 UUID 的长度与内容
    .service_uuid_len = sizeof(adv_service_uuid16),
    .p_service_uuid = adv_service_uuid16,
    // 设置设备的广播标志。
    // ESP_BLE_ADV_FLAG_GEN_DISC：设备支持通用发现（General Discoverable Mode），可以被扫描到。
    // ESP_BLE_ADV_FLAG_BREDR_NOT_SPT：设备不支持经典蓝牙（BR/EDR），只支持 BLE。
    .flag = (ESP_BLE_ADV_FLAG_GEN_DISC | ESP_BLE_ADV_FLAG_BREDR_NOT_SPT),
};

/**
 * gatt事件回调函数
 * @param event 事件ID
 * @param gatts_if gatt接口，一个profile对应一个
 * @param 
 *		event: 事件类型，指示发生了什么操作，如注册、读取、写入等。
 *		gatts_if: 表示 GATT 服务器接口。用于在 GATT 服务器和应用程序之间进行交互。
 * 			他是与 profile 对应的我们这里只设置了一个 profile 所以 gatts_if 可以不用管。
 *		param: 事件参数，包含具体事件的信息。它是一个结构体，类型为 esp_ble_gatts_cb_param_t
 * @return 无
 */
static void gatts_profile_event_handler(esp_gatts_cb_event_t event, esp_gatt_if_t gatts_if, esp_ble_gatts_cb_param_t *param){
    switch (event) {
        // 当 GATT profile 注册成功时触发的事件，表示设备的 GATT 服务已成功启动。    
        case ESP_GATTS_REG_EVT:
        {
            //设置 BLE 广播时的设备名称
            esp_err_t set_dev_name_ret = esp_ble_gap_set_device_name(BLE_DEVICE_NAME);
            if (set_dev_name_ret){
                ESP_LOGE(TAG, "set device name failed, error code = %x", set_dev_name_ret);
            }
            //设置广播数据，完成广播的初始化。
            esp_err_t ret = esp_ble_gap_config_adv_data(&adv_data);
            if (ret){
                ESP_LOGE(TAG, "config adv data failed, error code = %x", ret);
            }
            adv_config_done |= ADV_CONFIG_FLAG;
            //设置扫描回复参数
            // 即在设备广播过程中，其他设备扫描到该设备时，设备可以返回给扫描者更多的详细信息。
            // 也可以理解为，主机扫描到设备后，设备需要回复给主机的内容。
            ret = esp_ble_gap_config_adv_data(&scan_rsp_data);
            if (ret){
                ESP_LOGE(TAG, "config scan response data failed, error code = %x", ret);
            }
            adv_config_done |= SCAN_RSP_CONFIG_FLAG;      
            // 注册两个服务的属性表（gatt1_db 和 gatt2_db）。（每个属性表包含符合与特征）
            // 每个属性表都与一个 svc_ind_id 关联，并根据 gatts_if 注册到 GATT 服务器中。
            // 注册attr表1 第一个服务
            // gatt1_db 这是一个指向服务属性数据库的指针，定义了服务的属性表，包括服务的特征和描述符等。
            // gatts_if GATT服务器的接口标识符。
            // SV1_IDX_NB 指定了要添加到服务数据库中的最大属性数量。每个服务、特征和描述符都是一个属性。
            // 它应该与 gatt1_db 中定义的属性数量相匹配。
            // SVC_IND_ID1 这是服务的实例ID。
            // 在一个GATT服务器中，可以有多个服务实例，每个实例都有一个唯一的实例ID。
            esp_err_t create_attr_ret = esp_ble_gatts_create_attr_tab(gatt1_db, gatts_if, SV1_IDX_NB, SVC_IND_ID1);
            if (create_attr_ret){
                ESP_LOGE(TAG, "create attr1 table failed, error code = %x", create_attr_ret);
            }
            //注册attr表2 第二个服务
            create_attr_ret = esp_ble_gatts_create_attr_tab(gatt2_db, gatts_if, SV2_IDX_NB, SVC_IND_ID2);
            if (create_attr_ret){
                ESP_LOGE(TAG, "create attr2 table failed, error code = %x", create_attr_ret);
            }
            // param 事件参数，
            // param->reg.status：这是 ESP_GATTS_REG_EVT 事件中的字段，表示注册结果。
            // 保存到全局变量，因为发送数据的时候会用到这个接口
            if(param->reg.status == ESP_GATT_OK){
                gl_gatts_if = gatts_if;
                //gl_conn_id = param->connect.conn_id;
            }
        }
       	    break;
        // 收到客户端请求读取某个特征的值时触发此事件。
        case ESP_GATTS_READ_EVT:
        {
            ESP_LOGI(TAG, "ESP_GATTS_READ_EVT");
            // 定义一个响应结构体，也就是要返回的数据
            esp_gatt_rsp_t rsp;
            // 将响应结构体清零，确保没有残留数据。
            memset(&rsp, 0, sizeof(esp_gatt_rsp_t));
            // 设置响应结构体中的句柄，表示要返回的特征值的句柄。
            rsp.attr_value.handle = param->read.handle;
            // 服务1中特征1的特征值句柄。
            // 在GATT协议中，每个特征值或描述符的句柄是全局唯一的，即使它们属于不同的服务。
            // 所以可以不用去判断属于哪一个服务；主机只在乎特征值，其他都是配置自动回复的
            // 所以我们只用判断是哪一个特征值就可以了
            if(sv1_handle_table[SV1_CH1_IDX_CHAR_VAL] == param->read.handle){
                // 将 sv1_char2_value 的数据复制到响应结构体中。
                rsp.attr_value.len = sizeof(sv1_char2_value);
                memcpy(rsp.attr_value.value,sv1_char2_value,sizeof(sv1_char2_value));
            }
            // 服务1中特征2的特征值句柄。
            if(sv1_handle_table[SV1_CH2_IDX_CHAR_VAL] == param->read.handle){
                rsp.attr_value.len = sizeof(sv2_char2_value);
                memcpy(rsp.attr_value.value,sv2_char2_value,sizeof(sv2_char2_value));
            }
            // 将准备好的响应数据发送给客户端。
            // gatts_if：GATT服务器接口。
            // param->read.conn_id：连接的ID，表示是哪个客户端发起的请求。
            // param->read.trans_id：事务ID，用于标识本次读取请求。
            // ESP_GATT_OK：表示响应成功。
            // &rsp：指向响应结构体的指针。
            esp_ble_gatts_send_response(gatts_if, param->read.conn_id, param->read.trans_id,ESP_GATT_OK, &rsp);
       	    break;
        }
        // 当客户端发送写入请求时，BLE栈会触发该事件。
        case ESP_GATTS_WRITE_EVT:
            // is_prep：表示是否是准备写入（长写入）。
            // 如果是 true，设备首先会将这些数据保存下来，等主机确认了最后一包这个预写的数据发完后
            // 我们的设备才一次性的把这些数据更新到内存中去
            if (!param->write.is_prep){
                // the data length of gattc write  must be less than GATTS_DEMO_CHAR_VAL_LEN_MAX.
                ESP_LOGI(TAG, "GATT_WRITE_EVT, handle = %d, value len = %d, value :", param->write.handle, param->write.len);
                // 打印写入请求的详细信息，包括句柄、数据长度和数据内容。
                // 以十六进制格式打印数据内容。
                esp_log_buffer_hex(TAG, param->write.value, param->write.len);
                // 分别保存特征1客户端写入数据的第一个和第二个字节.
                if(param->write.handle == sv1_handle_table[SV1_CH1_IDX_CHAR_CFG]){ 
                    sv1_ch1_client_cfg[0] = param->write.value[0];
                    sv1_ch1_client_cfg[1] = param->write.value[1];
                }
                // 分别保存特征2客户端写入数据的第一个和第二个字节.
                else if(param->write.handle == sv1_handle_table[SV1_CH2_IDX_CHAR_CFG]){
                    sv1_ch2_client_cfg[0] = param->write.value[0];
                    sv1_ch2_client_cfg[1] = param->write.value[1];
                }
                // 分别保存特征1特征值写入数据的第一个和第二个字节.
                // 特征2只允许读和通知所以不考虑写入
                else if(param->write.handle == sv1_handle_table[SV1_CH1_IDX_CHAR_VAL]){
                    sv1_char2_value[0] = param->write.value[0];
                    sv1_char2_value[1] = param->write.value[1];
                }
				// 如果客户端需要响应，则发送响应。
                // gatts_if：GATT服务器接口。
				//param->write.conn_id：连接的ID，表示是哪个客户端发起的请求。
				//param->write.trans_id：事务ID，用于标识本次写入请求。
				//ESP_GATT_OK：表示响应成功。
				//NULL：表示没有额外的响应数据。
                if (param->write.need_rsp){
                    esp_ble_gatts_send_response(gatts_if, param->write.conn_id, param->write.trans_id, ESP_GATT_OK, NULL);
                }
            }
      	    break;
        case ESP_GATTS_EXEC_WRITE_EVT:
            // the length of gattc prepare write data must be less than GATTS_DEMO_CHAR_VAL_LEN_MAX.
            ESP_LOGI(TAG, "ESP_GATTS_EXEC_WRITE_EVT");
            //example_exec_write_event_env(&prepare_write_env, param);
            break;
        case ESP_GATTS_MTU_EVT:
            ESP_LOGI(TAG, "ESP_GATTS_MTU_EVT, MTU %d", param->mtu.mtu);
            break;
        case ESP_GATTS_CONF_EVT:
            ESP_LOGI(TAG, "ESP_GATTS_CONF_EVT, status = %d, attr_handle %d", param->conf.status, param->conf.handle);
            break;
        case ESP_GATTS_START_EVT:
            ESP_LOGI(TAG, "SERVICE_START_EVT, status %d, service_handle %d", param->start.status, param->start.service_handle);
            break;
        // 当 BLE 设备与客户端（例如手机或其他 BLE 设备）建立连接后，会触发这个事件。    
        case ESP_GATTS_CONNECT_EVT:
            // 印连接事件的相关信息。
            ESP_LOGI(TAG, "ESP_GATTS_CONNECT_EVT, conn_id = %d", param->connect.conn_id);
            esp_log_buffer_hex(TAG, param->connect.remote_bda, 6);
            // 初始化连接参数结构体 esp_ble_conn_update_params_t
            esp_ble_conn_update_params_t conn_params = {0};
            // 将客户端的蓝牙地址复制到结构体中。
            memcpy(conn_params.bda, param->connect.remote_bda, sizeof(esp_bd_addr_t));
            // 设置连接参数
            // iOS 对 BLE 连接参数有严格的限制，开发者需要遵循 Apple 的官方文档，否则可能导致连接失败或不稳定。
            // 表示从机可以忽略多少个连接事件。这里设置为 0，表示从机不会忽略任何事件。
            conn_params.latency = 0;    //从机延迟
            conn_params.max_int = 0x20; // 最大连接间隔 = 0x20*1.25ms = 40ms
            conn_params.min_int = 0x10; // 最小连接间隔 = 0x10*1.25ms = 20ms
            // 如果在这个时间内没有通信，连接会被认为断开。
            conn_params.timeout = 400;  // 监控超时 = 400*10ms = 4000ms
            // 向客户端发送更新连接参数的请求。客户端可以选择接受或拒绝这些参数。
            esp_ble_gap_update_conn_params(&conn_params);
            // 将连接句柄保存到全局变量 gl_conn_id 中，以便后续使用
            gl_conn_id = param->connect.conn_id;
            break;
        case ESP_GATTS_DISCONNECT_EVT:  //收到断开连接事件
            ESP_LOGI(TAG, "ESP_GATTS_DISCONNECT_EVT, reason = 0x%x", param->disconnect.reason);
            // 注意断开后要从新设置连接，并将连接id设为一个无效值
            esp_ble_gap_start_advertising(&adv_params);
            gl_conn_id = 0xFFFF;
            break;
        // 当 GATT 服务的属性表创建成功时触发  
        case ESP_GATTS_CREAT_ATTR_TAB_EVT: 
            // 检查属性表是否创建成功。
            if (param->add_attr_tab.status != ESP_GATT_OK){
                    ESP_LOGE(TAG, "create attribute table failed, svc id = %d,error code=0x%x",param->add_attr_tab.svc_inst_id, param->add_attr_tab.status);
                }
            // 处理服务属性表1的结果。SVC_IND_ID1 服务实例 1 的 ID
            if(param->add_attr_tab.svc_inst_id == SVC_IND_ID1)
            {
                // 如果 num_handle 不等于 SV1_IDX_NB，说明属性表创建异常，记录错误日志。
                // SV1_IDX_NB：服务实例 1 的属性表预期包含的属性数量。
                if (param->add_attr_tab.num_handle != SV1_IDX_NB){
                    ESP_LOGE(TAG, "create attribute table abnormally, num_handle (%d) \
                            doesn't equal to NETCFG_IDX_NB(%d)", param->add_attr_tab.num_handle, SV1_IDX_NB);
                }
                else {
                    ESP_LOGI(TAG, "create attribute table successfully, the number handle = %d\n",param->add_attr_tab.num_handle);
                    // 将创建的属性句柄数组复制到 sv1_handle_table 中，以便后续使用。
                    memcpy(sv1_handle_table, param->add_attr_tab.handles, sizeof(sv1_handle_table));
                    // 启动服务实例 1
                    esp_ble_gatts_start_service(sv1_handle_table[SV1_IDX_SVC]);
                }         
            }
            else if(param->add_attr_tab.svc_inst_id == SVC_IND_ID2)
            {
                if (param->add_attr_tab.num_handle != SV2_IDX_NB){
                    ESP_LOGE(TAG, "create attribute table abnormally, num_handle (%d) \
                            doesn't equal to NETCFG_IDX_NB(%d)", param->add_attr_tab.num_handle, SV2_IDX_NB);
                }
                else {
                    ESP_LOGI(TAG, "create attribute table successfully, the number handle = %d\n",param->add_attr_tab.num_handle);
                    memcpy(sv2_handle_table, param->add_attr_tab.handles, sizeof(sv2_handle_table));
                    esp_ble_gatts_start_service(sv2_handle_table[SV2_IDX_SVC]);
                }
            }
            break;
        case ESP_GATTS_STOP_EVT:
        case ESP_GATTS_OPEN_EVT:
        case ESP_GATTS_CANCEL_OPEN_EVT:
        case ESP_GATTS_CLOSE_EVT:
        case ESP_GATTS_LISTEN_EVT:
        case ESP_GATTS_CONGEST_EVT:
        case ESP_GATTS_UNREG_EVT:
        case ESP_GATTS_DELETE_EVT:
        default:
            break;
    }
}

/**
 * GAP事件回调函数
 * GAP是蓝牙协议栈中定义设备如何发现、连接和与其他设备交互的部分。
 * @param event BLE GAP 事件类型，枚举类型
 * @param param 一个指向 esp_ble_gap_cb_param_t 的指针，包含与当前事件相关的参数。
 * @return 无
 */
static void gap_event_handler(esp_gap_ble_cb_event_t event, esp_ble_gap_cb_param_t *param){
    switch (event) {
        // 该事件触发时，表示广播数据已经成功设置    
        case ESP_GAP_BLE_ADV_DATA_SET_COMPLETE_EVT:
            // 将标志位置为0
            adv_config_done &= (~ADV_CONFIG_FLAG);
            if (adv_config_done == 0){
                // 开始广播，adv_params中是配置的广播参数
                esp_ble_gap_start_advertising(&adv_params);
            }
            break;
        // 该事件触发时，表示扫描响应数据已经成功设置    
        case ESP_GAP_BLE_SCAN_RSP_DATA_SET_COMPLETE_EVT:
            // 将标志位置为0
            adv_config_done &= (~SCAN_RSP_CONFIG_FLAG);
            if (adv_config_done == 0){
                // 开始广播
                esp_ble_gap_start_advertising(&adv_params);
            }
            break;
        // 此事件表示广播成功启动    
        case ESP_GAP_BLE_ADV_START_COMPLETE_EVT:
            /* advertising start complete event to indicate advertising start successfully or failed */
            if (param->adv_start_cmpl.status != ESP_BT_STATUS_SUCCESS) {
                ESP_LOGE(TAG, "advertising start failed");
            }else{
                ESP_LOGI(TAG, "advertising start successfully");
            }
            break;
        case ESP_GAP_BLE_ADV_STOP_COMPLETE_EVT:             //停止广播成功事件
            if (param->adv_stop_cmpl.status != ESP_BT_STATUS_SUCCESS) {
                ESP_LOGE(TAG, "Advertising stop failed");
            }
            else {
                ESP_LOGI(TAG, "Stop adv successfully\n");
            }
            break;
        case ESP_GAP_BLE_UPDATE_CONN_PARAMS_EVT:        //更新连接参数成功事件
            ESP_LOGI(TAG, "update connection params status = %d, min_int = %d, max_int = %d,conn_int = %d,latency = %d, timeout = %d",
                  param->update_conn_params.status,
                  param->update_conn_params.min_int,
                  param->update_conn_params.max_int,
                  param->update_conn_params.conn_int,
                  param->update_conn_params.latency,
                  param->update_conn_params.timeout);
            break;
        default:
            break;
    }
}

/**
 * 初始化并启动蓝牙BLE
 * @param 无
 * @return 是否成功
 */
esp_err_t ble_cfg_net_init(void){
    // esp_err_t ret;
    // 释放 Classic Bluetooth（经典蓝牙）模式的内存。
    // ESP32 支持多种蓝牙模式，ESP_BT_MODE_CLASSIC_BT 是经典蓝牙模式，
    // 调用该函数会释放其相关的内存资源，因为这里我们要使用的是 BLE 模式，经典蓝牙模式不再需要。
    ESP_ERROR_CHECK(esp_bt_controller_mem_release(ESP_BT_MODE_CLASSIC_BT));
	// 引用提供的蓝牙默认配置
    esp_bt_controller_config_t bt_cfg = BT_CONTROLLER_INIT_CONFIG_DEFAULT();
    // 使用默认配置初始化蓝牙控制器
    ESP_ERROR_CHECK(esp_bt_controller_init(&bt_cfg));
    // 启用蓝牙控制器并指定使用蓝牙低能耗 (BLE) 模式
    ESP_ERROR_CHECK(esp_bt_controller_enable(ESP_BT_MODE_BLE));
    // 定义并初始化蓝牙堆栈（Bluedroid）的配置结构体
    // Bluedroid 是 ESP32 官方封装好的蓝牙协议栈支持经典蓝牙和低功耗蓝牙。
    esp_bluedroid_config_t bluedroid_cfg = BT_BLUEDROID_INIT_CONFIG_DEFAULT();
    // 使用指定的配置初始化蓝牙堆栈
    ESP_ERROR_CHECK(esp_bluedroid_init_with_cfg(&bluedroid_cfg));
    // 启用蓝牙堆栈
    ESP_ERROR_CHECK(esp_bluedroid_enable());
    // 注册 GATT 事件用于注册处理 GATT 服务器事件的回调函数
    ESP_ERROR_CHECK(esp_ble_gatts_register_callback(gatts_profile_event_handler));
    // 注册 GAP（通用访问配置文件）事件回调函数处理与 BLE 连接相关的事件，例如设备扫描、连接、断开连接等。
    ESP_ERROR_CHECK(esp_ble_gap_register_callback(gap_event_handler));
    // 注册一个 GATT 应用,通常来说一个应用注册一个 ID 即可。
    // 例如你的设备需要同时支持心率服务和温度服务，那么你需要为这两个服务分别定义不同的应用 ID。
    ESP_ERROR_CHECK(esp_ble_gatts_app_register(ESP_APP_ID));
    // 设置本地的 GATT MTU（最大传输单元）。指在网络协议中一次传输的数据包的最大字节数。
    ESP_ERROR_CHECK(esp_ble_gatt_set_local_mtu(500));
    return ESP_OK;
}

// 这两个函数是暴露给外部去使用的
/**
 * 设置特征1的值
 * @param value 值
 * @return 无
 */
void ble_set_ch1_value(uint16_t value){
    sv1_char2_value[0] = value&0xff;
    sv1_char2_value[1] = value>>8;
    //判断连接是否有效，以及客户端特征配置是否不为0
    // 因为 0 表示不能上报
    if(gl_conn_id != 0xFFFF && (sv1_ch1_client_cfg[0] | sv1_ch1_client_cfg[1])){
        // 更新 GATT 服务器中特征的值。
        // v1_handle_table[SV1_CH1_IDX_CHAR_VAL]：特征的句柄（Handle）
        // 2：特征值的长度（2 字节）。
        // (const uint8_t*)&sv1_char2_value：特征值的数据。
        esp_ble_gatts_set_attr_value(sv1_handle_table[SV1_CH1_IDX_CHAR_VAL], 2, (const uint8_t*)&sv1_char2_value);
        // 向客户端发送通知（Indication）。
        // gl_gatts_if：GATT 服务器的接口 ID。
        // gl_conn_id：连接句柄，表示当前连接。
        // sv1_handle_table[SV1_CH1_IDX_CHAR_VAL]：特征的句柄。
        // 2：特征值的长度。m
        // (uint8_t*)&sv1_char2_value：特征值的数据。
        // false：表示发送的是通知（Notification），而不是指示（Indication）。
        // 如果是 true，则表示发送的是指示（Indication），客户端需要回复确认。
        // 注意这里发送的通知或指示，要看特征中配置的是通知还是指示
        esp_ble_gatts_send_indicate(gl_gatts_if, gl_conn_id,sv1_handle_table[SV1_CH1_IDX_CHAR_VAL], 2, (uint8_t*)&sv1_char2_value, false);
    }
}

/**
 * 设置特征2的值
 * @param value 值
 * @return 无
 */
void ble_set_ch2_value(uint16_t value){
    sv2_char2_value[0] = value&0xff;
    sv2_char2_value[1] = value>>8;
    //判断连接是否有效，以及客户端特征配置是否不为0
    if(gl_conn_id != 0xFFFF && (sv1_ch2_client_cfg[0] | sv1_ch2_client_cfg[1])){
        esp_ble_gatts_set_attr_value(sv1_handle_table[SV1_CH2_IDX_CHAR_VAL], 2, (const uint8_t*)&sv2_char2_value);
        esp_ble_gatts_send_indicate(gl_gatts_if, gl_conn_id,sv1_handle_table[SV1_CH2_IDX_CHAR_VAL], 2, (uint8_t*)&sv2_char2_value, false);
    }
}    
=============================================================
// main.c    
#include <stdio.h>
#include "esp_err.h"
#include "ble.h"
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "nvs_flash.h"

void mytask1(void* param){
    uint16_t count1 = 1;
    uint16_t count2 = 100;
    while(1){
        vTaskDelay(pdMS_TO_TICKS(3000));
        ble_set_ch1_value(count1++);
        vTaskDelay(pdMS_TO_TICKS(700));
        ble_set_ch2_value(count2++);
    }
}

void app_main(void){
    // 初始化 NVS (Non-Volatile Storage)，它是一个用于存储配置、状态信息等的非易失性存储
    ESP_ERROR_CHECK(nvs_flash_init());
    // 初始化蓝牙配置
    ble_cfg_net_init();
    // 创建一个名为 mytask1 的 FreeRTOS 任务并将其固定到指定的 CPU 核心。
    // 这个任务我们拿来做自己的事情
    xTaskCreatePinnedToCore(mytask1,"mytask",4096,NULL,3,NULL,1);
}
```

代码运行后可以通过蓝牙检索到名为"ESP32-HOME"的设备，通过上述的微信小程序可成功连接到蓝牙；

## 问题

一、**头文件报错**

例如：fatal error: esp_bt.h: No such file or directory

![image-20250120211959363](C:\Users\kay\AppData\Roaming\Typora\typora-user-images\image-20250120211959363.png)

![image-20250120213226853](C:\Users\kay\AppData\Roaming\Typora\typora-user-images\image-20250120213226853.png)
