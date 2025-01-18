---
title: "MQTT协议"
date: 2025-1-17 18:18:00
categories: "ESP32"
tags: 
- "ESP32-S3"
---

MQTT(Message Queuing Telemetry Transport，消息队列遥测传输协议)，是一种基于发布/订阅(publish/subscribe)模式的"轻量级"通讯协议，该协议构建于 TCP/IP 协议上，由 IBM 在 1999 年发布。MQTT 最大优点在于，可以以极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务。

**协议类型**：发布/订阅（Pub/Sub）模式。

**传输层协议**：基于 TCP/IP 协议栈，默认使用 **TCP** 作为传输协议，但也有针对 **WebSocket** 和 **TLS** 等协议的扩展。

**轻量级**：头部非常小，适合在带宽受限、计算能力有限的嵌入式设备中使用。

## MQTT协议特性

**发布/订阅模型**

- **发布者（Publisher）**：向特定的“主题（Topic）”发布消息。
- **订阅者（Subscriber）**：对一个或多个主题进行订阅，接收相关的消息。
- **代理（Broker）**：充当消息的中转站，管理客户端的连接、消息路由和交付。

这种模型使得通信双方不需要直接互相了解或联系，而是通过代理来传递消息，提升了系统的灵活性和扩展性。

**消息传递**

- **主题（Topic）**：消息的标识符，通常是一个层级结构的字符串。例如，home/livingroom/light。

- QoS（Quality of Service）级别

  ：MQTT 提供了三种消息传递服务质量级别：

  - **QoS 0**：最多一次传送（At most once），消息可能丢失，不会重发。
  - **QoS 1**：至少一次传送（At least once），消息会重发，直到确认收到。
  - **QoS 2**：只有一次传送（Exactly once），确保消息只发送一次。

**持久化会话（Session Persistence）**

- MQTT 允许在断开连接后保留客户端的会话状态，包括订阅信息和未接收的消息。连接恢复后，可以继续接收消息，而无需重新订阅。

**遗嘱消息（Last Will and Testament, LWT）**

- MQTT 允许在客户端异常断开时，代理发送一个遗嘱消息。这有助于检测客户端的状态，并通知其他订阅者。

**低带宽和高效性**

- MQTT 协议的消息头非常小，最大只有 2 字节，适合带宽受限或不稳定的网络环境。

## MQTT协议包结构

MQTT（**Message Queuing Telemetry Transport**）协议结构主要由 **固定报头（Fixed Header）** 和 **可变报头（Variable Header）** 以及 **有效载荷（Payload）** 构成。可变头部与有效荷载不一定每个报文都有，内容根据报文类型不同而不同。

|              |             |                                |
| ------------ | ----------- | ------------------------------ |
| **固定头部** | 1-2 字节    | 包括消息类型、标志和剩余长度。 |
| **可变头部** | 0 到 N 字节 | 包含主题名称、消息标识符等。   |
| **有效载荷** | N 字节      | 包含消息的实际数据。           |

### 固定报头

**第一字节**：

- **0-3 位**：用于指定报文类型的标志位。
- **4-7 位**：用于表示报文类型。

**第二字节及后续字节**：

- **剩余长度（Remaining Length）**：从第二字节开始，表示后续可变报头和消息负载的总长度。
- 剩余长度字段最多可以使用**四个字节**。
- 单个字节最大值为 **0x7F**（十六进制），即 127 字节。
- 如果剩余长度字段的最高位（第 8 位）为 **1**，则表示后续还有更多字节存在，这种机制被称为“延续位”。
- 因此，最后一字节的最大值只能是 **0x7F**。

**固定报头第一个字节表**

![gdbzdygzj](/public/image/嵌入式/MCU/ESP32/ESP32S3/gdbzdygzj.png)

> DUP<sup>1</sup>：用于标识消息是否是重复的。0：表示该消息是新的，1：表示该消息是重复的。
>
> QoS<sup>2</sup>：定义消息传递的可靠性。MQTT协议提供了三个级别的QoS，分别为：
>
> - **QoS 0**：至多一次（At most once）——消息最多被传输一次，不做重传保证。适用于不需要严格可靠性的场景。
> - **QoS 1**：至少一次（At least once）——消息至少传输一次，保证消息到达。会进行重试以确保消息被接收。
> - **QoS 2**：只有一次（Exactly once）——消息只会传输一次，并且通过四次握手保证消息的唯一性。适用于高可靠性需求的场景。
> - **值**：00：QoS 0、01：QoS 1、10：QoS 2
>
> RETAIN<sup>3</sup>：指示消息是否应该被保留在代理服务器上，以便后续的新订阅者能够立即接收到这条消息。
>
> - 如果设置了 RETAIN，则该消息会在代理中保留，直到有新的相同主题的消息发布。此时，新的订阅者一旦订阅该主题，会立即收到这条保留的消息。
>
> - **值**：0：表示消息不会被保留。1：表示消息将被保留。

### 可变报头与有效荷载

不同类型的报文可变报头与有效荷载均不相同，这里以 CONNECT 与 CONNACK 为例：

CONNECT 报文是用于客户端向服务器发起连接请求的报文，服务器会验证其中的信息并且返回 CONNACK 告知客户端结果。CONNECT 包含固定包头、可变包头、消息载荷。CONNACK 包含固定包头、可变包头。

**CONNECT 报文格式介绍**
可变包头由如下构成协议名(Protocol Name)、协议等级(Protocol Level)、连接标志(Connect Flags)、保持连接(Keep Alive)

![connectkbbt](/public/image/嵌入式/MCU/ESP32/ESP32S3/connectkbbt.png)

> 协议名：一般 3.1.1 以后都用 MQTT
>
> 协议等级：4 代表 3.1.1，3 代表 3.1.1 之前的版本，5 代表 5.0
>
> 保持连接时间：**客户端传输完成一个控制报文的时刻，到发送下一个报文的时刻，两者之间允许空闲的最大时间间隔**
>
> - 如果保持连接时间设置为`60`秒，那么每60秒客户端必须至少向服务器发送一次消息（或进行心跳检查），如果没有，则服务器可能会断开连接。

#### 连接标志

连接标志，主要用于指示 payload(有效荷载) 域存在哪些内容

![ljbz](/public/image/嵌入式/MCU/ESP32/ESP32S3/ljbz.png)

> Clean Session：标识客户端是(0)否(1)建立一个持久化的会话，当 Clean Session 的标识设为 0 时，代表客户端希望建立一个持久会话的连接，代理服务器将存储该客户端订阅的主题和未接受的消息,否则(设置为1)代理服务器不会存储这些数据，同时在建立连接时清除这个客户端之前存在的持久化会话所保存的数据。

如果连接标志包含所有内容，必须按如下这个顺序出现:客户端标识符，遗嘱主题，遗嘱消息，用户名，密 码，阿里云 IOT 不支持 will，因此 payload(有效荷载) 简化成如下：

![payloadcd](/public/image/嵌入式/MCU/ESP32/ESP32S3/payloadcd.png)

> 所谓遗嘱功能，就是当服务器检测到客户端非正常断开连接时，就会向客户端遗嘱主题中发送相应的遗嘱消息；上图中就是不使用遗嘱功能的 payload 图。

**CONNACK 报文格式介绍**

可变包头由如下构成:连接确认标志，返回码

![connackkbbt](/public/image/嵌入式/MCU/ESP32/ESP32S3/connackkbbt.png)

当 CONNECT 报文中的 Clean Session 标志设置为 1 时，当前会话标志为。

CONNACK 没有 payload 

## PUBLISH(发布)与SUBSCIRBE(订阅)报文

- **发布者（Publisher）**

  负责将消息发布到主题上，发布者一次只能向一个主题发送数据，发布者发布消息时也无需关心订阅者是否在线。

- **订阅者（Subscriber）**

  订阅者通过订阅主题接收消息，且可一次订阅多个主题。MQTT 还支持通过[共享订阅](https://www.emqx.com/zh/blog/introduction-to-mqtt5-protocol-shared-subscription)的方式在多个订阅者之间实现订阅的负载均衡。

- **代理（Broker）**

  负责接收发布者的消息，并将消息转发至符合条件的订阅者。另外，代理也需要负责处理客户端发起的连接、断开连接、订阅、取消订阅等请求。

- **主题（Topic）**

  主题是 MQTT 进行消息路由的基础，它类似 URL 路径，使用斜杠 / 进行分层，比如 sensor/1/temperature。一个主题可以有多个订阅者，代理会将该主题下的消息转发给所有订阅者；一个主题也可以有多个发布者，代理将按照消息到达的顺序转发。

**MQTT 发布与订阅流程**

![mqttlc](/public/image/嵌入式/MCU/ESP32/ESP32S3/mqttlc.png)

> MQTT 是基于主题来进行消息流向的；
>
> topic(主题)：topic 在 MQTT 里面是消息传递的基础，代表了消息的流向，从本质上看，topic 是一串字符串，可以使用正斜杠对topic 进行分级。
>
> 例子：比如我向 A 代理服务器订阅了如下主题消息"/编程知识/嵌入式/C语言”，当有其他人向 A 服务器推送了主题“/编程知识/嵌入式/C语言”的消息时候，那么代理服务器就会把这条消息推送给我；如果别人推送的是“/编程知识/嵌入式/C++”，由于主题不匹配，那么代理服务器就不会把这条消息推送给我们。

### 订阅协议格式

图片展示了与订阅和响应相关的数据格式，主要分为两部分：**订阅请求报文**和**订阅响应报文**。

![dyqq](/public/image/嵌入式/MCU/ESP32/ESP32S3/dyqq.png)

> **固定报头**：
>
> - 包含高 4 位的订阅报文类型（byte1），以及底 4 位的固定QOS（Quality of Service）值（byte1）。
> - 从第二字节开始，表示后续可变报头和消息负载的总长度。
>
> **可变报头**：
>
> - PacketID：标识报文的唯一ID。PacketID 的作用是为报文的发送和接收提供唯一性，确保发送方和接收方在处理时能够正确地匹配请求和响应报文；也就是说发送与回应中的 PacketID 要保持一致。
>
> **帧载荷**：
>
> - 2 byte 主题长度，表示主题的字节数。
> - 主题内容（byte3到byteN），即具体的订阅内容。
>
> - 服务质量要求QoS：比如有个客户端要向这个主题发布一条消息，那么客户端发布报文的 QoS 等级就不能高于此处的 QoS 等级。

### 发布协议格式

![dyxy](/public/image/嵌入式/MCU/ESP32/ESP32S3/dyxy.png)

## 实现ESP32连接MQTT

### MQTTX客户端工具

这里要使用一个 MQTTX 这个工具。

[MQTTX 官网下载](https://mqttx.app/)

我们要利用这个工具去连接到一个免费的给我们测试用的 MQTT 服务器中，操作如下：

![mqttlj](/public/image/嵌入式/MCU/ESP32/ESP32S3/mqttlj.png)

点击连接后就可以了。

### ESP32实现

**实现分为两个部分，首先要让 ESP32 连接到互联网，之后才可以进行 MQTT 操作**

```c
// sta.c
#include <stdio.h>
#include "nvs_flash.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "freertos/semphr.h"

// 配置一个认证模式
#define ESP_WIFI_SCAN_AUTH_MODE_THRESHOLD WIFI_AUTH_WPA2_PSK

// SSID、密码、最大连接次数
#define EXAMPLE_ESP_WIFI_SSID      "Redmi Note 14 5G"
#define EXAMPLE_ESP_WIFI_PASS      "wang123456"
#define EXAMPLE_ESP_MAXIMUM_RETRY  100

// FreeRTOS 事件组，当我们连接时发出信号
static EventGroupHandle_t s_wifi_event_group;

// 事件组连接和失败标志位
#define WIFI_CONNECTED_BIT BIT0
#define WIFI_FAIL_BIT      BIT1

// 打印日志用的标识符
static char *TAG = "wifi station";

// 当前连接次数
static int s_retry_num = 0;

// 连接成功标志,使用二进制信号量
SemaphoreHandle_t  wifi_is_connected = NULL;

// 事件回调函数
static void event_handler(void* arg, esp_event_base_t event_base,
                                int32_t event_id, void* event_data) {
    // 先判断事件的类型：WIFI_EVENT 表示wifi事件，WIFI_EVENT_STA_START 表示模式为 STA 启动模式
    if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_START) {
        // 连接到这个 WiFi
        esp_wifi_connect();
    // 表示 WiFi 断开事件    
    } else if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_DISCONNECTED) {
        // 判断当前重复连接次数是否小于最大重复次数
        if (s_retry_num < EXAMPLE_ESP_MAXIMUM_RETRY) {
            esp_wifi_connect();
            s_retry_num++;
            ESP_LOGI(TAG, "retry to connect to the AP");
        // 如果达到最大重试次数，设置 WIFI_FAIL_BIT 标志，通知连接失败。
        } else {
            xEventGroupSetBits(s_wifi_event_group, WIFI_FAIL_BIT);
        }
        // 打印失败日志
        ESP_LOGI(TAG,"connect to the AP fail");
    // 获得 IP 地址事件    
    // IP_EVENT_STA_GOT_IP 表示 Wi-Fi 站点 (STA) 成功通过 DHCP（动态主机配置协议）获取到了 IP 地址。  
    } else if (event_base == IP_EVENT && event_id == IP_EVENT_STA_GOT_IP) {
        ip_event_got_ip_t* event = (ip_event_got_ip_t*) event_data;
        ESP_LOGI(TAG, "got ip:" IPSTR, IP2STR(&event->ip_info.ip));
        s_retry_num = 0;
        // 设置 WIFI_CONNECTED_BIT 标志，表示连接成功。
        xEventGroupSetBits(s_wifi_event_group, WIFI_CONNECTED_BIT);
    // 表示 Wi-Fi 站点 (STA) 成功连接到 Wi-Fi 网络的 接入点 (AP)。  
    } else if(event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_CONNECTED) {
        ESP_LOGI(TAG, "esp32 connected to AP!");
        xSemaphoreGive(wifi_is_connected);
    }
}

void wifi_init_sta(void) {
    // 创建一个事件组，用于任务间通信和同步。
    s_wifi_event_group = xEventGroupCreate();
	// 初始化底层的网络接口模块，为后续的网络通信做好准备。
    ESP_ERROR_CHECK(esp_netif_init());
    // 创建默认的事件循环，用于处理系统事件（如 Wi-Fi 的连接、断开、获取 IP 地址等事件）。
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    // 创建默认的 Wi-Fi STA 接口（Station），这是 ESP32 的网络接口之一，用于连接到路由器。
    // 该函数会返回一个网卡对象，但是我们一般用不到
    esp_netif_create_default_wifi_sta();
    /*初始化 WiFi 驱动*/
    // 通过 WIFI_INIT_CONFIG_DEFAULT() 宏生成默认配置结构体。
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    // 初始化 Wi-Fi 驱动
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
    // 创建二进制信号量，用于后续 MQTT 判断
    wifi_is_connected = xSemaphoreCreateBinary();
	// 注册 Wi-Fi 和 IP 事件处理程序
    esp_event_handler_instance_t instance_any_id;
    esp_event_handler_instance_t instance_got_ip;
    // WIFI_EVENT 表示事件类型
    // ESP_EVENT_ANY_ID 表示监听所有的 Wi-Fi 事件，而不是单一特定事件。
    // event_handler 表示处理函数
    // 自定义参数填 NULL
    ESP_ERROR_CHECK(esp_event_handler_instance_register(WIFI_EVENT,
                                                        ESP_EVENT_ANY_ID,
                                                        &event_handler,
                                                        NULL,
                                                        &instance_any_id));
    // 同上，这里表示注册一个 IP 事件
    // 当我们连接到路由器，获取到 IP 后就会触发这个事件
    ESP_ERROR_CHECK(esp_event_handler_instance_register(IP_EVENT,
                                                        IP_EVENT_STA_GOT_IP,
                                                        &event_handler,
                                                        NULL,
                                                        &instance_got_ip));
	// 配置 WiFi STA 模式参数
    wifi_config_t wifi_config = {
        .sta = {
            // 要连接的 Wi-Fi 路由器的 SSID。
            .ssid = EXAMPLE_ESP_WIFI_SSID,
            // Wi-Fi 路由器的密码。
            .password = EXAMPLE_ESP_WIFI_PASS,
            /*
               设置加密模式
               如果 Wi-Fi 密码符合 WPA2 标准，即密码长度为 8 字节或以上（这是 WPA2 的最低要求）
               则认证模式（Authmode）默认设置为 WPA2。
               
			   ESP32 会自动判断提供的密码是否满足 WPA2 标准，并相应地设置 Wi-Fi 的认证模式为 WPA2。
               如果用户未主动修改 authmode 的值，这个默认设置将会被使用。
               
               如果你需要连接到较旧（已被淘汰）的 Wi-Fi 网络（例如 WEP 或 WPA 加密方式的网络）
               需要手动将 authmode 的阈值设置为 WIFI_AUTH_WEP 或 WIFI_AUTH_WPA_PSK。
               当连接到 WEP 或 WPA 网络时，密码必须符合这些网络的要求：
					WEP：密码可以是 5 个字符（40 位）或 13 个字符（104 位）等格式。
					WPA：密码通常需要在 8 到 63 个字符之间。
             */       
            .threshold.authmode = ESP_WIFI_SCAN_AUTH_MODE_THRESHOLD,
            // 保护管理帧，提升网络安全性。
            // 表示设备支持 PMF 功能。
            // 设备不强制要求 PMF。如果接入点支持 PMF，则会使用；如果接入点不支持，也允许连接。
            .pmf_cfg.capable = true,
            .pmf_cfg.required = false,
        },
    };
    // 设置 Wi-Fi 模式为 WIFI_MODE_STA，即客户端模式。
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA) );
    // 配置 Wi-Fi STA 的参数（如 SSID 和密码）。
    ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_config) );
    // 启动 Wi-Fi 驱动，开始尝试连接到配置好的 Wi-Fi 网络。
    ESP_ERROR_CHECK(esp_wifi_start() );
    ESP_LOGI(TAG, "wifi_init_sta finished.");
    
	/* 等待连接结果 */
    /* Waiting until either the connection is established (WIFI_CONNECTED_BIT) or connection failed for the maximum
     * number of re-tries (WIFI_FAIL_BIT). The bits are set by event_handler() (see above) */
    // 等待事件组中的指定事件位（WIFI_CONNECTED_BIT | WIFI_FAIL_BIT）被设置。
    // pdFALSE、pdFALSE 表示不清除事件位、表示任意一个事件位设置即可返回。
    // portMAX_DELAY 超时时间，portMAX_DELAY 表示无限等待，直到事件发生。
    // 返回的是事件组中设置的事件位状态，即调用 xEventGroupWaitBits 时，事件组中的状态位。
    EventBits_t bits = xEventGroupWaitBits(s_wifi_event_group,
            WIFI_CONNECTED_BIT | WIFI_FAIL_BIT,
            pdFALSE,
            pdFALSE,
            portMAX_DELAY);

    /* xEventGroupWaitBits() returns the bits before the call returned, hence we can test which event actually
     * happened. */
    // 判断连接状态
    // WIFI_CONNECTED_BIT（Wi-Fi 连接成功的标志）
    if (bits & WIFI_CONNECTED_BIT) {
        ESP_LOGI(TAG, "connected to ap SSID:%s password:%s",
                 EXAMPLE_ESP_WIFI_SSID, EXAMPLE_ESP_WIFI_PASS);
    // WIFI_FAIL_BIT（Wi-Fi 连接失败的标志）    
    } else if (bits & WIFI_FAIL_BIT) {
        ESP_LOGI(TAG, "Failed to connect to SSID:%s, password:%s",
                 EXAMPLE_ESP_WIFI_SSID, EXAMPLE_ESP_WIFI_PASS);
    // 表示发生了意料之外的情况（理论上不应该进入）。    
    } else {
        ESP_LOGE(TAG, "UNEXPECTED EVENT");
    }
}

void sta_main(void) {
    // NVS是 ESP32 内部的一块非易失性存储空间，通常用于保存 Wi-Fi 配置、设备配置信息等需要掉电保持的数据。
    // 当我们使用 SSID 和 密码连接成功后，IDF 的底层会帮我们把这组 SSID 和密码保存到 NVS 中，下次
    // 系统启动的时候，启动 STA 模式连接后，就会使用这组 SSID 和密码继续连接；
    esp_err_t ret = nvs_flash_init();
    // 如果返回值为 ESP_ERR_NVS_NO_FREE_PAGES 或 ESP_ERR_NVS_NEW_VERSION_FOUND
    // 则表示当前 NVS 存储有问题（如存储空间不足或版本不兼容）。
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
      // nvs_flash_erase() 擦除 NVS 存储，清空之前的数据。
      // ESP_ERROR_CHECK 用于检查函数返回值。如果函数返回错误，会触发异常，打印错误日志并终止程序。
      ESP_ERROR_CHECK(nvs_flash_erase());
      // 重新初始化 NVS (nvs_flash_init())，确保可以正常使用  
      ret = nvs_flash_init();
    }
    ESP_ERROR_CHECK(ret);
	// 使用 ESP-IDF 提供的日志系统，打印一条信息日志，表明接下来将配置 Wi-Fi 工作模式为 STA
    ESP_LOGI(TAG, "ESP_WIFI_MODE_STA");
    // 配置并启动 Wi-Fi 的 STA 模式，使 ESP32 作为 Wi-Fi 客户端连接到指定的路由器。
    wifi_init_sta();
}
===================================================================
// main.c
#include <stdio.h>
#include "esp_log.h"
#include "mqtt_client.h"

// MQTTX 配置
#define MQTT_ADDRESS "mqtt://broker-cn.emqx.io"
#define MQTT_PORT 1883
#define MQTT_USER "kay"
#define MQTT_PWD "wang123456"
#define MQTT_CLIENTID "esp32_ClientID"

// 主题
// ESP32 向 MQTT_TOPIC_ESP32_SEND 发送信息
#define MQTT_TOPIC_ESP32_SEND "/topic/esp32_send"
// MQTTX 向 MQTT_TOPIC_ESP32_RECV 发送信息，ESP32 接收
#define MQTT_TOPIC_ESP32_RECV "/topic/esp32_recv"

static char *TAG = "mqtt5";

extern SemaphoreHandle_t  wifi_is_connected;
extern void sta_main();

/*
 * @brief 注册的事件处理程序，用于接收 MQTT 事件
 *
 *  该函数由 MQTT 客户端事件循环调用。
 *
 * @param handler_args 用户数据，注册到该事件的。
 * @param base 事件基类（在此示例中始终为 MQTT Base）。
 * @param event_id 接收到的事件的 ID。
 * @param event_data 事件数据，类型为 esp_mqtt_event_handle_t。
 */
static void mqtt5_event_handler(void *handler_args, esp_event_base_t base, int32_t event_id, void *event_data) {
    // 打印事件是从哪个事件循环基础（base）触发的，并且标明了事件的 ID，便于识别是哪种事件。
    ESP_LOGD(TAG, "Event dispatched from event loop base=%s, event_id=%" PRIi32, base, event_id);
    esp_mqtt_event_handle_t event = event_data;
    esp_mqtt_client_handle_t client = event->client;
    int msg_id;

    // 打印当前设备的堆内存状态，输出的信息包括当前的可用堆内存大小和最低可用堆内存大小，
    ESP_LOGD(TAG, "free heap size is %" PRIu32 ", minimum %" PRIu32, esp_get_free_heap_size(), esp_get_minimum_free_heap_size());
    switch ((esp_mqtt_event_id_t)event_id) {
    // MQTT 连接成功事件    
    case MQTT_EVENT_CONNECTED:
        // MQTT 连接成功
        ESP_LOGI(TAG, "MQTT connected");
        // 连接成功后就可以进行发布与订阅
        // 订阅主题
        msg_id = esp_mqtt_client_subscribe(client, MQTT_TOPIC_ESP32_RECV, 1);
        ESP_LOGI(TAG, "sent subscribe successful, msg_id=%d", msg_id);
        break;
    // MQTT 断开事件    
    case MQTT_EVENT_DISCONNECTED:
        ESP_LOGI(TAG, "MQTT_EVENT_DISCONNECTED");
        break;
    // 向服务器订阅一个主题，服务器返回给我们 ACK 后会触发这个事件 
    case MQTT_EVENT_SUBSCRIBED:
        ESP_LOGI(TAG, "MQTT_EVENT_SUBSCRIBED, msg_id=%d", event->msg_id);
        msg_id = esp_mqtt_client_publish(client, "/topic/qos0", "data", 0, 0, 0);
        ESP_LOGI(TAG, "sent publish successful, msg_id=%d", msg_id);
        break;
    // 当 MQTT 客户端请求取消订阅某个主题时，服务器会返回响应，确认取消订阅的请求已被处理并成功完成。该事件便在客户端收到响应时触发。    
    case MQTT_EVENT_UNSUBSCRIBED:
        ESP_LOGI(TAG, "MQTT_EVENT_UNSUBSCRIBED, msg_id=%d", event->msg_id);
        esp_mqtt_client_disconnect(client);
        break;
    // MQTT 发布事件,当我们发布一条消息服务端给我们成功返回ACK之后会触发这个事件    
    case MQTT_EVENT_PUBLISHED:
        ESP_LOGI(TAG, "MQTT_EVENT_PUBLISHED, msg_id=%d", event->msg_id);
        break;
    // ESP32 收到服务器发送过来的消息时候会触发这个事件    
    case MQTT_EVENT_DATA:
        ESP_LOGI(TAG, "MQTT_EVENT_DATA");
        // 打印主题，与数据内容
        ESP_LOGI(TAG, "TOPIC=%.*s", event->topic_len, event->topic);
        ESP_LOGI(TAG, "DATA=%.*s", event->data_len, event->data);
        break;
    // 在在连接过程中发生了错误，会触发这个事件    
    case MQTT_EVENT_ERROR:
        ESP_LOGI(TAG, "MQTT_EVENT_ERROR");
        ESP_LOGI(TAG, "MQTT5 return code is %d", event->error_handle->connect_return_code);
        break;
    default:
        ESP_LOGI(TAG, "Other event id:%d", event->event_id);
        break;
    }
}

void mqtt_start(void) {

    esp_mqtt_client_config_t mqtt_cfg = {
        .broker.address.uri = MQTT_ADDRESS,
        .broker.address.port = MQTT_PORT,
        .credentials.client_id = MQTT_CLIENTID,
        .credentials.username = MQTT_USER,
        .credentials.authentication.password = MQTT_PWD,
    };
    // mqtt 初始化
    esp_mqtt_client_handle_t client = esp_mqtt_client_init(&mqtt_cfg);

    // 注册了一个 MQTT 客户端事件处理程序,
    // 监听来自于 client 这个示例中所有的 MQTT 事件，在 mqtt5_event_handler 函数中进行处理
    // ESP_EVENT_ANY_ID 监听所有的事件
    esp_mqtt_client_register_event(client, ESP_EVENT_ANY_ID, mqtt5_event_handler, NULL);

    esp_mqtt_client_start(client);
}

void app_main(void) {

    sta_main();
    // 只有在连接到网络之后会进行 MQTT 的一些列操作   
    // 等待获取信号量 
    xSemaphoreTake(wifi_is_connected, portMAX_DELAY);
    mqtt_start();
    xSemaphoreGive(wifi_is_connected);
   
}
```

运行后即可通过 MQTTX 实现与 EPS32 之间的数据交互，如图所示：

![mqttjg](/public/image/嵌入式/MCU/ESP32/ESP32S3/mqttjg.png)

## 问题

一、**导入的头文件报错**

ctrl + shift + P 选择：ESP-IDF: Add vscode configuration Folder

即可加入 IDF 路径

二、**新建的多个文件报错**

Build 下就好了
