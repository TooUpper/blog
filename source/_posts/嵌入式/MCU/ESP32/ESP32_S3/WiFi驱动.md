---
title: "WiFi驱动"
date: 2025-01-09 10:18:00
categories: "ESP32"
tags: 
- "ESP32-S3"
---

Wi-Fi（Wireless Fidelity）是基于IEEE 802.11标准的无线网络通信技术，允许设备**通过无线电波在局域网内进行数据传输和联网操作**。它是目前最常见的**无线局域网（WLAN）技术**，广泛用于家庭、办公以及公共场所。Wi-Fi 网络**通常采用 TCP/IP 协议进行通信**。

## **Wi-Fi的工作原理**

Wi-Fi通过在 2.4GHz 和 5GHz（部分设备支持6GHz）频段内的无线电波来传输数据：

1. **接入点（AP）** 作为中心设备，将有线网络（如光纤或以太网）转换为无线信号。
2. **终端设备（STA）** 接收无线信号，并通过网络协议与其他设备通信。
3. 数据通过无线信道在 AP 和 STA 之间传输，网络协议负责保障数据的可靠性和安全性。

## **Wi-Fi网络中常见的两种工作模式**

**Access Point (AP) 模式：无线接入点模式**

- **功能**：负责发射无线信号，允许终端设备（如手机和笔记本电脑）接入。
- **应用场景**：常见于家庭路由器、企业办公路由器等，起到网络中枢的作用。
- **特点**：支持多个设备接入、通常配备密码和加密机制（如WPA2或WPA3）以保障安全、AP 和 AP 可以互相连接。

**Station (STA) 模式：终端模式**

- **功能**：STA模式的设备连接到AP，作为网络终端使用，获取网络访问权限。
- **应用场景**：手机、平板电脑、智能家居设备等通常以STA模式连接至网络。
- **特点**：不具备发射无线信号的功能、设备仅作为终端使用，**不允许其他设备通过其转发连接或者接入**。

## ESP32中的WiFi

[ESP32 官方指南](https://docs.espressif.com/projects/esp-idf/zh_CN/v5.4/esp32/index.html)

ESP32 内置WiFi模块，支持以下三种主要的工作模式，用于满足不同的网络需求：

**STA模式（Station Mode，终端模式）**

- ESP32 作为无线终端连接到路由器，获取 IP 地址，访问外部服务器或与局域网内其他设备通信。
- **应用场景**：上传数据到云端，与局域网设备通信。
- **典型特点**：与路由器通信，设备需要提供 WiFi 网络的 SSID 和密码。

**AP模式（Access Point Mode，无线接入点模式）**

- ESP32 作为无线热点，创建一个WiFi网络，允许其他设备（如手机、平板等）连接到它。
- **应用场景**：点对点通信，设备初始配网或本地控制。
- **典型特点**：无需路由器，ESP32 自己生成网络并分配 IP。

**AP+STA模式（双模式）**

- ESP32同时作为无线终端连接路由器（STA），又作为无线热点（AP）提供网络接入。
- **应用场景**：需要同时与云端通信和本地设备交互，例如智能家居设备配网和数据上传。
- **典型特点**：结合 STA 和 AP 功能，支持双重连接。

## 实现

### STA模式

```c
#include <stdio.h>
#include "nvs_flash.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"

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
static const char *TAG = "wifi station";

// 当前连接次数
static int s_retry_num = 0;

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
    // IP_EVENT_STA_GOT_IP 表示已经成功连接到 Wi-Fi 并从 DHCP 获取到 IP 地址。    
    } else if (event_base == IP_EVENT && event_id == IP_EVENT_STA_GOT_IP) {
        ip_event_got_ip_t* event = (ip_event_got_ip_t*) event_data;
        ESP_LOGI(TAG, "got ip:" IPSTR, IP2STR(&event->ip_info.ip));
        s_retry_num = 0;
        // 设置 WIFI_CONNECTED_BIT 标志，表示连接成功。
        xEventGroupSetBits(s_wifi_event_group, WIFI_CONNECTED_BIT);
    // 打印是否连接成功事件    
    } else if(event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_CONNECTED) {
        ESP_LOGI(TAG, "esp32 connected to AP!");
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

void app_main(void) {
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
```

### AP模式

```c
#include <string.h>
#include "esp_mac.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"

// 配置 WiFi SSID（热点名称）、密码、信道和最大连接数。
#define EXAMPLE_ESP_WIFI_SSID      "ESP32Test"
#define EXAMPLE_ESP_WIFI_PASS      "wang123456"
#define EXAMPLE_ESP_WIFI_CHANNEL   4
#define EXAMPLE_MAX_STA_CONN       6

static const char *TAG = "wifi softAP";

// 事件处理函数
static void wifi_event_handler(void* arg, esp_event_base_t event_base,
                                    int32_t event_id, void* event_data){
    // WIFI_EVENT_AP_STACONNECTED: 设备成功连接到热点。                                    
    if (event_id == WIFI_EVENT_AP_STACONNECTED) {
        wifi_event_ap_staconnected_t* event = (wifi_event_ap_staconnected_t*) event_data;
        ESP_LOGI(TAG, "station "MACSTR" join, AID=%d",
                 MAC2STR(event->mac), event->aid);
    // WIFI_EVENT_AP_STADISCONNECTED: 设备断开与热点的连接。             
    } else if (event_id == WIFI_EVENT_AP_STADISCONNECTED) {
        wifi_event_ap_stadisconnected_t* event = (wifi_event_ap_stadisconnected_t*) event_data;
        ESP_LOGI(TAG, "station "MACSTR" leave, AID=%d, reason=%d",
                 MAC2STR(event->mac), event->aid, event->reason);
    }
}

void wifi_init_softap(void){
    // WiFi 初始化
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    esp_netif_create_default_wifi_ap();

    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
	// 注册事件
    ESP_ERROR_CHECK(esp_event_handler_instance_register(WIFI_EVENT,
                                                        ESP_EVENT_ANY_ID,
                                                        &wifi_event_handler,
                                                        NULL,
                                                        NULL));
	// WiFi 配置
    wifi_config_t wifi_config = {
        .ap = {
            .ssid = EXAMPLE_ESP_WIFI_SSID,
            .ssid_len = strlen(EXAMPLE_ESP_WIFI_SSID),
            .channel = EXAMPLE_ESP_WIFI_CHANNEL,
            .password = EXAMPLE_ESP_WIFI_PASS,
            .max_connection = EXAMPLE_MAX_STA_CONN,
#ifdef CONFIG_ESP_WIFI_SOFTAP_SAE_SUPPORT
            .authmode = WIFI_AUTH_WPA3_PSK,
            .sae_pwe_h2e = WPA3_SAE_PWE_BOTH,
#else /* CONFIG_ESP_WIFI_SOFTAP_SAE_SUPPORT */
            .authmode = WIFI_AUTH_WPA2_PSK,
#endif
            .pmf_cfg = {
                    .required = true,
            },
        },
    };
    if (strlen(EXAMPLE_ESP_WIFI_PASS) == 0) {
        wifi_config.ap.authmode = WIFI_AUTH_OPEN;
    }

    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_AP));
    ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_AP, &wifi_config));
    ESP_ERROR_CHECK(esp_wifi_start());

    ESP_LOGI(TAG, "wifi_init_softap finished. SSID:%s password:%s channel:%d",
             EXAMPLE_ESP_WIFI_SSID, EXAMPLE_ESP_WIFI_PASS, EXAMPLE_ESP_WIFI_CHANNEL);
}

void app_main(void){
    //Initialize NVS
    esp_err_t ret = nvs_flash_init();
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
      ESP_ERROR_CHECK(nvs_flash_erase());
      ret = nvs_flash_init();
    }
    ESP_ERROR_CHECK(ret);

    ESP_LOGI(TAG, "ESP_WIFI_MODE_AP");
    wifi_init_softap();
}
```

手机打开搜索对应 WiFi 名称输入密码即可连接；如果搜不出来可以刷新下，他的信号有可能在中间位置，要往下翻一翻；

### AP+STA模式

```c

#include <string.h>
#include "esp_mac.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "esp_netif.h"
#include "nvs_flash.h"
#if IP_NAPT
#include "lwip/lwip_napt.h"
#endif

/* STA Configuration */
#define EXAMPLE_ESP_WIFI_STA_SSID           "Redmi Note 14 5G"
#define EXAMPLE_ESP_WIFI_STA_PASSWD         "wang123456"
#define EXAMPLE_ESP_MAXIMUM_RETRY           6

// STA 下的模式
#define ESP_WIFI_SCAN_AUTH_MODE_THRESHOLD   WIFI_AUTH_WPA2_PSK

/* AP Configuration */
#define EXAMPLE_ESP_WIFI_AP_SSID            "ESP_AP"
#define EXAMPLE_ESP_WIFI_AP_PASSWD          "wang123456"
#define EXAMPLE_ESP_WIFI_CHANNEL            4
#define EXAMPLE_MAX_STA_CONN                6


// WIFI_CONNECTED_BIT 和 WIFI_FAIL_BIT：用作FreeRTOS事件标志位。
#define WIFI_CONNECTED_BIT BIT0
#define WIFI_FAIL_BIT      BIT1

/*DHCP server option*/
#define DHCPS_OFFER_DNS             0x02

static const char *TAG_AP = "WiFi SoftAP";
static const char *TAG_STA = "WiFi Sta";

static int s_retry_num = 0;

/* FreeRTOS event group to signal when we are connected/disconnected */
static EventGroupHandle_t s_wifi_event_group;

static void wifi_event_handler(void *arg, esp_event_base_t event_base,
                               int32_t event_id, void *event_data){
    // 有设备连接到ESP32的AP模式。                            
    if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_AP_STACONNECTED) {
        wifi_event_ap_staconnected_t *event = (wifi_event_ap_staconnected_t *) event_data;
        ESP_LOGI(TAG_AP, "Station "MACSTR" joined, AID=%d",
                 MAC2STR(event->mac), event->aid);
    // 设备从ESP32的AP断开。             
    } else if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_AP_STADISCONNECTED) {
        wifi_event_ap_stadisconnected_t *event = (wifi_event_ap_stadisconnected_t *) event_data;
        ESP_LOGI(TAG_AP, "Station "MACSTR" left, AID=%d, reason:%d",
                 MAC2STR(event->mac), event->aid, event->reason);
    // STA模式启动后，尝试连接到外部WiFi。             
    } else if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_START) {
        esp_wifi_connect();
        ESP_LOGI(TAG_STA, "Station started");
    // 成功获取IP地址后，将事件标志位 WIFI_CONNECTED_BIT 置位。    
    } else if (event_base == IP_EVENT && event_id == IP_EVENT_STA_GOT_IP) {
        ip_event_got_ip_t *event = (ip_event_got_ip_t *) event_data;
        ESP_LOGI(TAG_STA, "Got IP:" IPSTR, IP2STR(&event->ip_info.ip));
        s_retry_num = 0;
        xEventGroupSetBits(s_wifi_event_group, WIFI_CONNECTED_BIT);
    }
}

/* Initialize soft AP */
esp_netif_t *wifi_init_softap(void){
    esp_netif_t *esp_netif_ap = esp_netif_create_default_wifi_ap();

    //  AP模式初始化配置
    wifi_config_t wifi_ap_config = {
        .ap = {
            .ssid = EXAMPLE_ESP_WIFI_AP_SSID,
            .ssid_len = strlen(EXAMPLE_ESP_WIFI_AP_SSID),
            .channel = EXAMPLE_ESP_WIFI_CHANNEL,
            .password = EXAMPLE_ESP_WIFI_AP_PASSWD,
            .max_connection = EXAMPLE_MAX_STA_CONN,
            .authmode = WIFI_AUTH_WPA2_PSK,
            .pmf_cfg = {
                .required = false,
            },
        },
    };

    if (strlen(EXAMPLE_ESP_WIFI_AP_PASSWD) == 0) {
        wifi_ap_config.ap.authmode = WIFI_AUTH_OPEN;
    }

    ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_AP, &wifi_ap_config));

    ESP_LOGI(TAG_AP, "wifi_init_softap finished. SSID:%s password:%s channel:%d",
             EXAMPLE_ESP_WIFI_AP_SSID, EXAMPLE_ESP_WIFI_AP_PASSWD, EXAMPLE_ESP_WIFI_CHANNEL);

    return esp_netif_ap;
}

/* Initialize wifi station */
esp_netif_t *wifi_init_sta(void){
    esp_netif_t *esp_netif_sta = esp_netif_create_default_wifi_sta();

    wifi_config_t wifi_sta_config = {
        .sta = {
            .ssid = EXAMPLE_ESP_WIFI_STA_SSID,
            .password = EXAMPLE_ESP_WIFI_STA_PASSWD,
            .scan_method = WIFI_ALL_CHANNEL_SCAN,
            .failure_retry_cnt = EXAMPLE_ESP_MAXIMUM_RETRY,
            /* Authmode threshold resets to WPA2 as default if password matches WPA2 standards (password len => 8).
             * If you want to connect the device to deprecated WEP/WPA networks, Please set the threshold value
             * to WIFI_AUTH_WEP/WIFI_AUTH_WPA_PSK and set the password with length and format matching to
            * WIFI_AUTH_WEP/WIFI_AUTH_WPA_PSK standards.
             */
            .threshold.authmode = ESP_WIFI_SCAN_AUTH_MODE_THRESHOLD,
            .sae_pwe_h2e = WPA3_SAE_PWE_BOTH,
        },
    };

    ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_sta_config) );

    ESP_LOGI(TAG_STA, "wifi_init_sta finished.");

    return esp_netif_sta;
}

// DNS和NAPT设置
void softap_set_dns_addr(esp_netif_t *esp_netif_ap,esp_netif_t *esp_netif_sta){
    esp_netif_dns_info_t dns;
    esp_netif_get_dns_info(esp_netif_sta,ESP_NETIF_DNS_MAIN,&dns);
    uint8_t dhcps_offer_option = DHCPS_OFFER_DNS;
    ESP_ERROR_CHECK_WITHOUT_ABORT(esp_netif_dhcps_stop(esp_netif_ap));
    ESP_ERROR_CHECK(esp_netif_dhcps_option(esp_netif_ap, ESP_NETIF_OP_SET, ESP_NETIF_DOMAIN_NAME_SERVER, &dhcps_offer_option, sizeof(dhcps_offer_option)));
    ESP_ERROR_CHECK(esp_netif_set_dns_info(esp_netif_ap, ESP_NETIF_DNS_MAIN, &dns));
    ESP_ERROR_CHECK_WITHOUT_ABORT(esp_netif_dhcps_start(esp_netif_ap));
}

void app_main(void){
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());

    //Initialize NVS
    esp_err_t ret = nvs_flash_init();
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
        ESP_ERROR_CHECK(nvs_flash_erase());
        ret = nvs_flash_init();
    }
    ESP_ERROR_CHECK(ret);

    /* Initialize event group */
    s_wifi_event_group = xEventGroupCreate();

    /* Register Event handler */
    ESP_ERROR_CHECK(esp_event_handler_instance_register(WIFI_EVENT,
                    ESP_EVENT_ANY_ID,
                    &wifi_event_handler,
                    NULL,
                    NULL));
    ESP_ERROR_CHECK(esp_event_handler_instance_register(IP_EVENT,
                    IP_EVENT_STA_GOT_IP,
                    &wifi_event_handler,
                    NULL,
                    NULL));

    /*Initialize WiFi */
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));

    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_APSTA));

    /* Initialize AP */
    ESP_LOGI(TAG_AP, "ESP_WIFI_MODE_AP");
    esp_netif_t *esp_netif_ap = wifi_init_softap();

    /* Initialize STA */
    ESP_LOGI(TAG_STA, "ESP_WIFI_MODE_STA");
    esp_netif_t *esp_netif_sta = wifi_init_sta();

    /* Start WiFi */
    ESP_ERROR_CHECK(esp_wifi_start() );

    /*
     * Wait until either the connection is established (WIFI_CONNECTED_BIT) or
     * connection failed for the maximum number of re-tries (WIFI_FAIL_BIT).
     * The bits are set by event_handler() (see above)
     */
    EventBits_t bits = xEventGroupWaitBits(s_wifi_event_group,
                                           WIFI_CONNECTED_BIT | WIFI_FAIL_BIT,
                                           pdFALSE,
                                           pdFALSE,
                                           portMAX_DELAY);

    /* xEventGroupWaitBits() returns the bits before the call returned,
     * hence we can test which event actually happened. */
    if (bits & WIFI_CONNECTED_BIT) {
        ESP_LOGI(TAG_STA, "connected to ap SSID:%s password:%s",
                 EXAMPLE_ESP_WIFI_STA_SSID, EXAMPLE_ESP_WIFI_STA_PASSWD);
        softap_set_dns_addr(esp_netif_ap,esp_netif_sta);
    } else if (bits & WIFI_FAIL_BIT) {
        ESP_LOGI(TAG_STA, "Failed to connect to SSID:%s, password:%s",
                 EXAMPLE_ESP_WIFI_STA_SSID, EXAMPLE_ESP_WIFI_STA_PASSWD);
    } else {
        ESP_LOGE(TAG_STA, "UNEXPECTED EVENT");
        return;
    }

    /* Set sta as the default interface */
    esp_netif_set_default_netif(esp_netif_sta);

    /* Enable napt on the AP netif */
    if (esp_netif_napt_enable(esp_netif_ap) != ESP_OK) {
        ESP_LOGE(TAG_STA, "NAPT not enabled on the netif: %p", esp_netif_ap);
    }
}
```
