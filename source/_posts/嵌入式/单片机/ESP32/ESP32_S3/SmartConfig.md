---
title: "SmartConfig"
date: 2025-1-14 19:18:00
categories: "ESP32"
tags: 
- "ESP32-S3"
---

SmartConfig 是一种用于 ESP32 系列设备的快速配置 Wi-Fi 网络的技术。它通过智能手机或其他智能设备将 Wi-Fi 配置信息（SSID、密码）无线传输到 ESP32 设备中，免去手动输入的麻烦，适用于没有显示屏或输入设备的物联网设备。

SmartConfig 主要用于设备首次连接 Wi-Fi 网络时，或在需要切换网络的情况下。**它通过广播 Wi-Fi 配置信息，将网络设置传输给ESP32 设备，后者接收到信息后自动连接到指定的 Wi-Fi 网络。**

## SmartConfig工作原理

**手机App生成配置信息**：用户通过手机 APP（如 ESP32 提供的 SmartConfig 工具或自定义的 App）选择 Wi-Fi 网络并输入密码。手机会将这些信息通过 UDP 广播发送到局域网中的 ESP32 设备。

**ESP32设备接收配置**：ESP32 设备在启动时，开启 SmartConfig 服务并监听特定的 UDP 广播端口。当它接收到从手机发送的 Wi-Fi 配置包时，便获取到 Wi-Fi SSID 和密码。

**连接Wi-Fi网络**：ESP32 设备使用接收到的 SSID 和密码连接 Wi-Fi，并通过设置完成后可以执行其他业务逻辑。

**反馈连接结果**：连接成功后，ESP32 设备可以通过 LED 灯或其他指示器向用户反馈状态（例如，通过蓝色 LED 表示连接成功）。

## ESP32实现

### ESP32设备代码

```c
#include <string.h>
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"
#include "esp_netif.h"
#include "esp_smartconfig.h"

// FreeRTOS 事件组，用于在我们连接并准备好发送请求时发出信号。
static EventGroupHandle_t s_wifi_event_group;

/* 事件组允许为每个事件使用多个位，但我们只关心一个事件——我们是否已连接到 AP 并获得 IP 地址 */
// CONNECTED_BIT：当 Wi-Fi 连接成功并获得 IP 地址时，会设置这个标志位。
// ESPTOUCH_DONE_BIT：当 SmartConfig 获取到 Wi-Fi SSID 和密码时，会设置这个标志位。
// *TAG：用于日志打印时作为标签标识。
static const int CONNECTED_BIT = BIT0;
static const int ESPTOUCH_DONE_BIT = BIT1;
static const char *TAG = "smartconfig_example";

// SmartConfig 任务
static void smartconfig_example_task(void * parm) {
    EventBits_t uxBits;
    // 设置 SmartConfig 模式的类型，注意这里要与我们 App 的模式一致，不然会不兼容
    ESP_ERROR_CHECK( esp_smartconfig_set_type(SC_TYPE_ESPTOUCH) );
    // 使用默认配置并启动 SmartConfig 配置
    // 启动后设备开始监听来自手机或其他设备的 SmartConfig 信号。
    smartconfig_start_config_t cfg = SMARTCONFIG_START_CONFIG_DEFAULT();
    ESP_ERROR_CHECK( esp_smartconfig_start(&cfg) );
    // 死循环等待事件信号
    while (1) {
        // xEventGroupWaitBits() 函数会阻塞当前任务，直到事件组中的某些标志位被设置。
        uxBits = xEventGroupWaitBits(s_wifi_event_group, CONNECTED_BIT | ESPTOUCH_DONE_BIT, true, false, portMAX_DELAY);
        if(uxBits & CONNECTED_BIT) {
            ESP_LOGI(TAG, "WiFi Connected to ap");
        }
        if(uxBits & ESPTOUCH_DONE_BIT) {
            ESP_LOGI(TAG, "smartconfig over");
            // 停止 SmartConfig 过程，释放相关资源。
            esp_smartconfig_stop();
            // 删除当前任务
            vTaskDelete(NULL);
        }
    }
}

static void event_handler(void* arg, esp_event_base_t event_base,
                                int32_t event_id, void* event_data){
									
	// 当Wi-Fi STA 模式启动时，启动SmartConfig任务。								
    if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_START) {
        xTaskCreate(smartconfig_example_task, "smartconfig_example_task", 4096, NULL, 3, NULL);
	// Wi-Fi连接丢失时，会重新连接Wi-Fi。		
    } else if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_DISCONNECTED) {
        esp_wifi_connect();
        xEventGroupClearBits(s_wifi_event_group, CONNECTED_BIT);
	// Wi-Fi连接成功并获得IP地址时，设置 CONNECTED_BIT 标志，表示设备已连接。	
    } else if (event_base == IP_EVENT && event_id == IP_EVENT_STA_GOT_IP) {
        xEventGroupSetBits(s_wifi_event_group, CONNECTED_BIT);
	// 扫描完成事件	
    } else if (event_base == SC_EVENT && event_id == SC_EVENT_SCAN_DONE) {
        ESP_LOGI(TAG, "Scan done");
	// 找到可用的频道事件	
    } else if (event_base == SC_EVENT && event_id == SC_EVENT_FOUND_CHANNEL) {
        ESP_LOGI(TAG, "Found channel");
	// 当SmartConfig获取到Wi-Fi SSID和密码时，更新Wi-Fi配置并重新连接。	
    } else if (event_base == SC_EVENT && event_id == SC_EVENT_GOT_SSID_PSWD) {
        ESP_LOGI(TAG, "Got SSID and password");
		// 当 SC_EVENT_GOT_SSID_PSWD 事件触发时，表示SmartConfig已经成功获取了Wi-Fi的SSID和密码。
		// 这些信息会被保存到 wifi_config_t 结构中。
        smartconfig_event_got_ssid_pswd_t *evt = (smartconfig_event_got_ssid_pswd_t *)event_data;
        wifi_config_t wifi_config;
        uint8_t ssid[33] = { 0 };
        uint8_t password[65] = { 0 };
        uint8_t rvd_data[33] = { 0 };

        bzero(&wifi_config, sizeof(wifi_config_t));
        memcpy(wifi_config.sta.ssid, evt->ssid, sizeof(wifi_config.sta.ssid));
        memcpy(wifi_config.sta.password, evt->password, sizeof(wifi_config.sta.password));

/*
当 CONFIG_SET_MAC_ADDRESS_OF_TARGET_AP 选项启用时，
这段代码会在 SmartConfig 事件 SC_EVENT_GOT_SSID_PSWD 中捕获到目标 AP 的 SSID 和 密码 后
同时设置目标 AP 的 MAC 地址（BSSID）。
这使得 ESP32 只会连接到指定的 AP，而忽略其他具有相同 SSID 的 AP。
*/
#ifdef CONFIG_SET_MAC_ADDRESS_OF_TARGET_AP
        wifi_config.sta.bssid_set = evt->bssid_set;
        if (wifi_config.sta.bssid_set == true) {
            ESP_LOGI(TAG, "Set MAC address of target AP: "MACSTR" ", MAC2STR(evt->bssid));
            memcpy(wifi_config.sta.bssid, evt->bssid, sizeof(wifi_config.sta.bssid));
        }
#endif

        memcpy(ssid, evt->ssid, sizeof(evt->ssid));
        memcpy(password, evt->password, sizeof(evt->password));
        ESP_LOGI(TAG, "SSID:%s", ssid);
        ESP_LOGI(TAG, "PASSWORD:%s", password);
        if (evt->type == SC_TYPE_ESPTOUCH_V2) {
            ESP_ERROR_CHECK( esp_smartconfig_get_rvd_data(rvd_data, sizeof(rvd_data)) );
            ESP_LOGI(TAG, "RVD_DATA:");
            for (int i=0; i<33; i++) {
                printf("%02x ", rvd_data[i]);
            }
            printf("\n");
        }
		// 先关闭连接，随后调用 esp_wifi_set_config() 配置Wi-Fi，并尝试连接。
        ESP_ERROR_CHECK( esp_wifi_disconnect() );
        ESP_ERROR_CHECK( esp_wifi_set_config(WIFI_IF_STA, &wifi_config) );
        esp_wifi_connect();
	// SmartConfig过程完成，发送了确认消息。	
    } else if (event_base == SC_EVENT && event_id == SC_EVENT_SEND_ACK_DONE) {
        xEventGroupSetBits(s_wifi_event_group, ESPTOUCH_DONE_BIT);
    }
}

static void initialise_wifi(void)
{
    ESP_ERROR_CHECK(esp_netif_init());
    s_wifi_event_group = xEventGroupCreate();
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    esp_netif_t *sta_netif = esp_netif_create_default_wifi_sta();
    assert(sta_netif);

    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK( esp_wifi_init(&cfg) );

	/*
		注册三个事件：
		监听所有的 WIFI 事件
		监听 Wi-Fi 连接成功并获得 IP 地址时的事件
		监听所有的 SmartConfig 事件
	*/
    ESP_ERROR_CHECK( esp_event_handler_register(WIFI_EVENT, ESP_EVENT_ANY_ID, &event_handler, NULL) );
    ESP_ERROR_CHECK( esp_event_handler_register(IP_EVENT, IP_EVENT_STA_GOT_IP, &event_handler, NULL) );
    ESP_ERROR_CHECK( esp_event_handler_register(SC_EVENT, ESP_EVENT_ANY_ID, &event_handler, NULL) );

	// 设置 WIFI 模式为 STA，并启动
    ESP_ERROR_CHECK( esp_wifi_set_mode(WIFI_MODE_STA) );
    ESP_ERROR_CHECK( esp_wifi_start() );
}

void app_main(void){
	// 调用 nvs_flash_init() 初始化非易失性存储（NVS）
    ESP_ERROR_CHECK( nvs_flash_init() );
	// 初始化 Wi-Fi
    initialise_wifi();
}
```

### APP

ESP32 的 SmartConfig 的 App 叫 ESP touch

[ESP touch官方下载地址](https://www.espressif.com/en/support/download/apps?keys=&field_technology_tid%5B%5D=20)

下载完成后，我们的手机先要连接到一个 WiFi，然后打开  ESP touch，模式要选择与代码中设置的一致，然后操作即可。
