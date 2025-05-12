---
title: "Socket"
date: 2025-3-5 14:19:00
categories: "ARM"
tags: 
- "LN"
---

**Socket（套接字）** 是一种编程接口（API），用于在计算机网络中实现不同主机之间通信，允许不同设备或同一设备上的不同程序通过网络发送和接收数据。

**本质**：Socket 是网络通信的端点，类似于电话系统中的“插座”，连接应用程序和底层网络协议。

**用途**：

- 客户端与服务器通信（如你的 MCU 和 Python 服务器）。
- 数据传输（例如 HTTP、FTP、邮件等协议都基于 Socket）。

**类型**：

- **流套接字（SOCK_STREAM）**：基于 TCP，提供可靠、面向连接的通信（你的代码中使用）。
- **数据报套接字（SOCK_DGRAM）**：基于 UDP，无连接、不可靠但快速。
- 其他类型（如原始套接字 SOCK_RAW，用于特殊用途）。

![socketlc](/public/image/嵌入式/MCU/ARM/Cortex-M4/LN882H/socketlc.png)

Socket 实现通信的方式很简单，客户端和服务器只需要知道彼此的 **IP 地址** 和**端口号**，即可建立连接并进行数据传输。

## 通信中 IP 和端口的需求

1. **服务器端**

- 需要知道自己的 IP 和端口：
  - 服务器必须绑定到一个特定的 IP 和端口，以便客户端可以找到它。
  - 示例：你的 Python 代码中 server.bind(("192.168.2.54", 8080)) 指定了服务器的 IP（192.168.2.54）和端口（8080）。
  - **原因**：服务器需要监听这个地址，等待客户端连接。
- 不需要知道客户端的 IP 和端口：
  - 服务器通过 accept() 接受连接后，自动获取客户端的 IP 和端口（例如 conn, addr = server.accept() 返回的 addr）。
  - 示例：你的 Python 输出 Connected by ('192.168.2.52', xxxx)，其中 192.168.2.52 是 MCU 的 IP，xxxx 是动态分配的端口。
  - **原因**：客户端主动发起连接，服务器被动接受，无需预先知道客户端地址。

2. **客户端**

- 需要知道服务器的 IP 和端口：
  - 客户端必须明确指定目标服务器的 IP 和端口，才能发起连接。
  - 示例：你的 MCU 代码中 server_addr.sin_addr.s_addr = inet_addr("192.168.2.54") 和 server_addr.sin_port = lwip_htons(8080) 指定了服务器地址。
  - **原因**：客户端通过 connect() 建立连接，需要知道服务器的“地址簿”才能拨通“电话”。
- 不需要知道自己的 IP 和端口：
  - 客户端的 IP 由网络接口自动分配（例如你的 MCU 获取到 192.168.2.52）。
  - 客户端的端口由操作系统动态分配（临时端口，例如 49152-65535 范围内的某个值），无需手动指定。
  - 示例：你的 MCU 不需要显式设置本地端口，lwip_connect() 自动处理。
  - **原因**：客户端的本地地址在连接时由协议栈管理，服务器通过连接自动获知。

所以一般情况下我们只需要知道服务端的 IP 和端口号即可，服务端会被动接受客户端的地址。

## 阻塞与非阻塞

阻塞与非阻塞是对一个文件描述符指定的文件或设备的两种工作方式。 阻塞的意思是指，**当试图对该文件描述符进行读写时，如果当时没有东西可读或者暂时不可写，程序就进入等待状态，直到有东西可读或者可写为止。** 非阻塞的意思是，**当没有东西可读或者不可写时，读写函数就马上返回，而不会等待。**

现在来理解什么是阻塞socket，什么是非阻塞socket。每个通过socket()函数创建的socket，本质就是一个文件描述符，所以对该文件描述符的 **IO 操作**方式不同，就有了阻塞socket和非阻塞socket。 

## 工作原理

**创建**：客户端和服务器分别创建 Socket（socket()）。

**绑定与监听**：服务器绑定 IP 和端口（bind()），监听连接（listen()）。

**连接**：客户端连接服务器（connect()），通过 TCP 三次握手建立连接。

**数据传输**：双方通过 send() 和 recv() 交换数据，TCP 保证可靠传输。

**关闭**：通信结束时关闭 Socket（close()），通过四次挥手释放连接。

## 实现一

该实现是基于亮牛 LN882H 芯片，示例模板为 wifi_mcu_basic_example 

需求一、连通 WiFi 保证 Socket 通信：
可以随时监听来自指定 IP 和端口的信息，并且每 5s 向对方发送一句 Hello from MCU!

```c
static void socket_task_entry(void *params) {
    // 没什么意义，params不用会提示警告，用于去除警告
    LN_UNUSED(params);
    // Socket ID，初始化为 -1，表示未创建。
    int sockfd = -1;
    // 存储服务端地址（IP 和端口）。
    struct sockaddr_in server_addr;
    // 定义连接和接收的超时时间为 5 秒（5000 毫秒）。
    int timeout_ms = 5000;
    // 控制 MCU 向服务端发送数据的频率。
    int send_interval_ms = 5000;

    while (1) {
		// 检查 WiFi 是否获取到 IP 地址，返回 true 表示已连接并有 IP。 
        if (!netdev_got_ip()) {
            LOG(LOG_LVL_INFO, "Socket task: No IP Address, waiting for WiFi...\n");
            OS_MsDelay(1000);
            continue; // 跳回主循环开头，继续检查 IP 状态。
        }

        // 存储网络信息。
        tcpip_ip_info_t ip_info;
        // 获取 STA（客户端）模式的网络信息。
        netdev_get_ip_info(NETIF_IDX_STA, &ip_info);
        // 打印分配的 IP 地址、子网掩码和网关。说明已经配网成功
        LOG(LOG_LVL_INFO, "Socket task: IP assigned: %s, Mask: %s, Gateway: %s\n",
            ip4addr_ntoa(&ip_info.ip),
            ip4addr_ntoa(&ip_info.netmask),
            ip4addr_ntoa(&ip_info.gw));

        // 获取并打印当前 WiFi 信号强度。
        // dBm数值越接近0，表明信号越强。
        int8_t rssi;
        wifi_sta_get_rssi(&rssi);
        LOG(LOG_LVL_INFO, "WiFi RSSI: %d dBm\n", rssi);

        // 确保 WiFi 连接有效时才执行 Socket 操作。
        while (netdev_got_ip()) {
            // 小于 0 表示需要创建新 Socket。
            if (sockfd < 0) {
                // 创建 TCP Socket。返回值：成功为非负整数，失败为 -1。
                sockfd = lwip_socket(AF_INET, SOCK_STREAM, 0);
                if (sockfd < 0) {
                    LOG(LOG_LVL_ERROR, "Socket Creation Failed: %d\n", errno);
                    OS_MsDelay(2000);
                    continue;
                }
                // 创建成功时打印 Socket 文件描述符。
                LOG(LOG_LVL_INFO, "Socket created ID=%d\n", sockfd);

                // 将 Socket 设置为非阻塞模式。
                int flags = 1; 
                // FIONBIO：非阻塞标志。flags = 1：启用非阻塞。
                if (lwip_ioctl(sockfd, FIONBIO, &flags) < 0) {
                    LOG(LOG_LVL_ERROR, "lwip_ioctl failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                // 配置服务端地址。
                memset(&server_addr, 0, sizeof(server_addr));
                server_addr.sin_family = AF_INET; // 设置 IPv4
                server_addr.sin_port = lwip_htons(8080);
                server_addr.sin_addr.s_addr = inet_addr("192.168.2.51");

                // 尝试连接服务端。 
                LOG(LOG_LVL_INFO, "Attempting to connect to 192.168.2.51:8080...\n");
                int connect_result = lwip_connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
                // 非 EINPROGRESS 的错误表示连接失败（例如服务端不可达）。
                if (connect_result < 0 && errno != EINPROGRESS) {
                    LOG(LOG_LVL_ERROR, "lwip_connect immediate failure: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                // 使用 select 检查连接状态。
                fd_set writefds, exceptfds;
                struct timeval tv;
                tv.tv_sec = 5;
                tv.tv_usec = 0;
                FD_ZERO(&writefds);
                FD_ZERO(&exceptfds);
                FD_SET(sockfd, &writefds);
                FD_SET(sockfd, &exceptfds);
                int select_result = lwip_select(sockfd + 1, NULL, &writefds, &exceptfds, &tv);
                // 检查 select 结果。
                // select_result <= 0：超时或错误。
                // FD_ISSET(sockfd, &exceptfds)：异常。
                if (select_result <= 0 || FD_ISSET(sockfd, &exceptfds)) {
                    LOG(LOG_LVL_ERROR, "Socket Connect timed out or failed after 5s, errno=%d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                // 检查 Socket 连接的最终状态。
                int so_error = 0;
                socklen_t len = sizeof(so_error);
                // SO_ERROR：获取连接错误码。
                if (lwip_getsockopt(sockfd, SOL_SOCKET, SO_ERROR, &so_error, &len) < 0 || so_error != 0) {
                    LOG(LOG_LVL_ERROR, "lwip_connect failed with error: %d\n", so_error);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }
				// 连接成功后设置接收超时。
                LOG(LOG_LVL_INFO, "Connected to 192.168.2.51:8080\n");
                tv.tv_sec = timeout_ms / 1000;
                tv.tv_usec = (timeout_ms % 1000) * 1000;
                // SO_RCVTIMEO：接收超时 5 秒。
                if (lwip_setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, &tv, sizeof(tv)) < 0) {
                    LOG(LOG_LVL_ERROR, "lwip_setsockopt SO_RCVTIMEO failed: %d\n", errno);
                }
            }

            // 定义发送消息和时间戳。
            char *msg = "Hello from MCU!";
            uint32_t last_send_time = 0;
            // 检查 Socket 和 WiFi 状态，确保通信有效。
            while (sockfd >= 0 && netdev_got_ip()) {
                // 获取当前时间戳
                uint32_t current_time = OS_GetTicks();
                // 每隔 5s 发送一次
                if (current_time - last_send_time >= send_interval_ms) {
                    int sent = lwip_send(sockfd, msg, strlen(msg), 0);
                    if (sent < 0) {
                        LOG(LOG_LVL_ERROR, "lwip_send failed: %d\n", errno);
                        lwip_close(sockfd);
                        sockfd = -1;
                        break;
                    }
                    LOG(LOG_LVL_INFO, "Sent: %s (%d bytes)\n", msg, sent);
                    last_send_time = current_time;
                }

                // 接收服务端数据。最多 127 个字节
                char buffer[128];
                int len = lwip_recv(sockfd, buffer, sizeof(buffer) - 1, 0);
                if (len > 0) { // len > 0：接收成功。
                    buffer[len] = '\0';
                    LOG(LOG_LVL_INFO, "Received: %s\n", buffer);
                } else if (len == 0) { // len == 0：服务端关闭。
                    LOG(LOG_LVL_INFO, "Server closed connection\n");
                    lwip_close(sockfd);
                    sockfd = -1;
                    break;
                // EAGAIN/ETIMEDOUT：超时，延时 100ms。
                } else if (errno == EAGAIN || errno == ETIMEDOUT) {
                    OS_MsDelay(100);
                // 其他错误：关闭 Socket。    
                } else {
                    LOG(LOG_LVL_ERROR, "lwip_recv failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    break;
                }

                // 收发循环中监控 RSSI，实时检查 WiFi 状态。
                wifi_sta_get_rssi(&rssi);
                LOG(LOG_LVL_INFO, "WiFi RSSI: %d dBm\n", rssi);
            }

            // Socket 断开时延时重试。
            if (sockfd < 0) {
                LOG(LOG_LVL_ERROR, "Socket disconnected, retrying...\n");
                OS_MsDelay(2000);
            }
        }

        // WiFi 断开时打印延时后等待重试。
        LOG(LOG_LVL_ERROR, "WiFi disconnected, waiting for reconnect...\n");
        OS_MsDelay(2000);
    }
}
```

## 实现二

需求二、接收服务端发送过来的打印数据，存入全局缓冲区(共用蓝牙的那一个)，打印业务会读取进行打印

​	Socket 通信：因为 Sokcet 要区分不同的数据类型(打印数据、音频数据、...)，使用 {"type":"text"} 这样的 JSON 进行区分
​	数据存入的逻辑：保持 BLE 原有业务不动，调用 BLE 原来的 API，将 JOSN 解析后对应的数据写入缓冲区

注意：

​	JSON 传输格式必须按要求传输 {"type":"text","data":[打印数据]}、 {"type":"sudio","data":"音频数据"}(暂不实现)
​	JSON 格式的解析，只需要 data 中相关的数据
​	由于 Socket 基于 TCP 流式传输，无法保证数据包的边界。因此，发送的多个独立 JSON 数据（如 319 字节和 398 字节）可能会在网络层被合并成一个数据包，导致我们 JSON 分割的失败，这里要特别注意。
​	由于图片大小不确定，我们无法预留足够大的空间来存放图片。因此，缓冲区需要设置为环形缓冲区。同时，需要注意写入和读取之间的时机和速度关系，以确保数据的正确处理。(可以与 BLE 公用一个)

```c
// 缓存大小定义，如果和 BLE 共用缓冲区可以去掉
#define TEXT_BUFFER_SIZE   20480
#define AUDIO_BUFFER_SIZE  20480
static uint8_t text_buffer[TEXT_BUFFER_SIZE];  // 静态分配环形缓冲区

// 记录写入数据
static uint16_t write_pos = 0;  // 写入位置
static uint16_t read_pos = 0;   // 读取位置（由打印函数更新，初始为 0）
static uint16_t text_used = 0;  // 当前缓冲区中有效数据量

// 用于 Socket 数据接收的临时缓冲区
static char temp_buffer[1024];
static uint16_t temp_pos = 0; 

// 比较 data 中从 offset 开始的子字符串是否与目标字符串 target 匹配。
// 成功匹配：返回匹配后新的偏移量（offset + target_len）。
// 失败：返回 -1。
static int match_string(const uint8_t *data, uint16_t len, uint16_t offset, const char *target) {
    uint16_t target_len = strlen(target);
    if (offset + target_len > len) {
        return -1;
    }

    if (strncmp((const char *)(data + offset), target, target_len) == 0) {
        return offset + target_len;
    }
    return -1;
}

// 跳过 data 中从 offset 开始的空白字符。
static uint16_t skip_whitespace(const uint8_t *data, uint16_t len, uint16_t offset) {
    while (offset < len && (data[offset] == ' ' || data[offset] == '\t' || data[offset] == '\n' || data[offset] == '\r')) {
        offset++;
    }
    return offset;
}

// JSON解析（处理二进制数据流）
// data：JSON 缓冲区指针、len：数据长度
void parse_json(uint8_t *data, uint16_t len) {
    
    LOG(LOG_LVL_INFO, "JSON parsing started, len=%d\n", len);

    // 检查 {} 格式是正确，data[len] = '\0'
    if (len < 2 || data[0] != '{' || data[len - 1] != '}') {
        LOG(LOG_LVL_INFO, " JSON format Error\n");
        return;
    }

    // 跳过开头的 { 和其后的空格
    uint16_t offset = 1;
    offset = skip_whitespace(data, len, offset);

    // 检查是否包含 "type":"，记录其偏移量
    int new_offset;
    if ((new_offset = match_string(data, len, offset, "\"type\":\"")) == -1) {
        LOG(LOG_LVL_INFO, "Missing type field\n");
        return;
    }
    offset = new_offset; // 跳过 "type":" 

    // 是否是 text
    uint8_t is_text = 0;
    // 检查值是否为 text
    if ((new_offset = match_string(data, len, offset, "text")) != -1) {
        is_text = 1;
        offset = new_offset; // 偏移到 text" 的位置
    }
    // 检查值是否为 audio
    else if ((new_offset = match_string(data, len, offset, "audio")) != -1) {
        is_text = 0;
        offset = new_offset; // 位置偏移
    }
    else { // 不识别的类型
        LOG(LOG_LVL_INFO, "type value Error\n");
        return;
    }

	// 当前是否指向 "
    if (offset >= len || data[offset] != '"') {
        LOG(LOG_LVL_INFO, "Missing closing quote after type value\n");
        return;
    }
    offset++; // 跳过 "
    offset = skip_whitespace(data, len, offset);
    if (offset >= len || data[offset] != ',') {
        LOG(LOG_LVL_INFO, "Missing comma after type\n");
        return;
    }

    // 跳过空格和其后的空白
    offset++;
    offset = skip_whitespace(data, len, offset);

    // 检查是否是 "data": 如果正确就偏移跳过，包括其后的空白
    if ((new_offset = match_string(data, len, offset, "\"data\":")) == -1) {
        LOG(LOG_LVL_INFO, "Missing data field\n");
        return;
    }
    offset = new_offset; 
    offset = skip_whitespace(data, len, offset);

    // 可用空间 = 总缓冲区大小 - 已使用空间
    uint16_t available_space = TEXT_BUFFER_SIZE - text_used;


    // 打印处理的逻辑
    if (is_text) {
        LOG(LOG_LVL_INFO, "Type: text detected, parsing array\n");

        // 跳过 [ 和其后的空格
        if (offset >= len || data[offset] != '[') {
            LOG(LOG_LVL_INFO, "Missing opening bracket for data array\n");
            return;
        }
        offset++; 
        offset = skip_whitespace(data, len, offset);

		// 数组元素计数
        uint16_t data_count = 0;
        while (offset < len && data[offset] != ']') {
            uint8_t value;
            int num = 0; // 存储当前解析的数字
			// 检查字符是否为 0~9
            if (offset >= len || (data[offset] < '0' || data[offset] > '9')) {
                LOG(LOG_LVL_INFO, "data[offset] Error offset=%d\n", offset);
                return;
            }

            while (offset < len && (data[offset] >= '0' && data[offset] <= '9')) {
                num = num * 10 + (data[offset] - '0');  // 将连续的数字字符转为整数
                offset++;
            }

            // 检查是否超出范围 unit8_t 0~255
            if (num > 255) {
                LOG(LOG_LVL_INFO, "Number out of range at offset=%d\n", offset);
                return;
            }

            // 检查剩余缓冲区是否足够存储新数据
            if (data_count >= available_space) {
                LOG(LOG_LVL_INFO, "Buffer not enough, count=%d, available=%d\n", data_count, available_space);
                return;
            }

            // 循环存入缓冲区
            text_buffer[(write_pos + data_count) % TEXT_BUFFER_SIZE] = (uint8_t)num;
            data_count++; // 计数++
			
            // 跳过逗号和其后的空白字符
            offset = skip_whitespace(data, len, offset);
            if (offset < len && data[offset] == ',') {
                offset++; // 跳过逗号
                offset = skip_whitespace(data, len, offset);
            }
        }

        // 解析完成后检查 ] 并且跳过其后的空白字符
        if (offset >= len || data[offset] != ']') {
            LOG(LOG_LVL_INFO, "Missing closing bracket for data array\n");
            return;
        }
        offset++; // 跳过 ]
        offset = skip_whitespace(data, len, offset);

        if (offset != len - 1) {
            LOG(LOG_LVL_INFO, "Extra data after array\n");
            return;
        }

        // 更新缓冲区状态
        // write_pos，写入位置
        // text_used 已使用字节数。
        write_pos = (write_pos + data_count) % TEXT_BUFFER_SIZE;
        text_used += data_count;
        LOG(LOG_LVL_INFO, "Text data parsed, count=%d, write_pos=%d, text_used=%d\n", 
            data_count, write_pos, text_used);
		
    }
    // 除打印数据外的信息数据(暂未实现，只是一个模板，但是原理同上面是一样的，只是[]换成了"")
    else {
        LOG(LOG_LVL_INFO, "Type: audio detected, parsing string\n");

        if (offset >= len || data[offset] != '"') {
            LOG(LOG_LVL_INFO, "Missing opening quote for data string\n");
            return;
        }
        offset++;
        offset = skip_whitespace(data, len, offset);

        uint16_t data_len = 0;
        uint16_t start_offset = offset;
        while (offset < len && data[offset] != '"') {
            offset++;
            data_len++;
        }
        if (offset >= len || data[offset] != '"') {
            LOG(LOG_LVL_INFO, "Missing closing quote for data string\n");
            return;
        }
        offset++;
        offset = skip_whitespace(data, len, offset);

        if (offset != len - 1) {
            LOG(LOG_LVL_INFO, "Extra data after string\n");
            return;
        }

        if (data_len > available_space) {
            LOG(LOG_LVL_INFO, "Buffer not enough, data_len=%d, available=%d\n", data_len, available_space);
            return;
        }

        for (uint16_t i = 0; i < data_len; i++) {
            text_buffer[(write_pos + i) % TEXT_BUFFER_SIZE] = data[start_offset + i];
        }

        write_pos = (write_pos + data_len) % TEXT_BUFFER_SIZE;
        text_used += data_len;
        LOG(LOG_LVL_INFO, "Audio data parsed, len=%d, write_pos=%d, text_used=%d\n", 
            data_len, write_pos, text_used);
    }
}

static void socket_task_entry(void *params) {
	// 避免编译器警告
    LN_UNUSED(params);
	// socket 句柄，初始化为 -1（未连接状态）
    int sockfd = -1;
	// 存储 IP 和端口信息
    struct sockaddr_in server_addr;

    int timeout_ms = 5000;

    while (1) {
		// 检查网络设备是否已获取 IP 地址。
        if (!netdev_got_ip()) {
            LOG(LOG_LVL_INFO, "Socket task: No IP Address, waiting for WiFi...\n");
            OS_MsDelay(1000);
            continue;
        }

		// 获取网络接口的 IP 信息
        tcpip_ip_info_t ip_info; 
        netdev_get_ip_info(NETIF_IDX_STA, &ip_info);
		// 打印当前 IP 配置。
        LOG(LOG_LVL_INFO, "Socket task: IP assigned: %s, Mask: %s, Gateway: %s\n",
            ip4addr_ntoa(&ip_info.ip),
            ip4addr_ntoa(&ip_info.netmask),
            ip4addr_ntoa(&ip_info.gw));

//		  信号强度显示	
//        int8_t rssi;
//        wifi_sta_get_rssi(&rssi);
//        LOG(LOG_LVL_INFO, "WiFi RSSI: %d dBm\n", rssi);

		// 在 WiFi 连接状态下，进入 socket 操作循环。
        while (netdev_got_ip()) {
			// 检查 socket 状态
            if (sockfd < 0) {
				// 创建 Socket
                sockfd = lwip_socket(AF_INET, SOCK_STREAM, 0);
                if (sockfd < 0) {
                    LOG(LOG_LVL_ERROR, "Socket Creation Failed: %d\n", errno);
                    OS_MsDelay(2000);
                    continue;
                }
 				// 打印创建成功的 socket ID。
                LOG(LOG_LVL_INFO, "Socket created ID=%d\n", sockfd);

 				// 设置 socket 为非阻塞模式。
                int flags = 1; // 启用非阻塞
                if (lwip_ioctl(sockfd, FIONBIO, &flags) < 0) {
                    LOG(LOG_LVL_ERROR, "lwip_ioctl failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                // 配置服务器地址。memset 用于清空结构体
                memset(&server_addr, 0, sizeof(server_addr));
                server_addr.sin_family = AF_INET; // IPv4
                server_addr.sin_port = lwip_htons(8080);
                server_addr.sin_addr.s_addr = inet_addr("192.168.2.51");

				// 尝试连接服务器
                LOG(LOG_LVL_INFO, "Attempting to connect to 192.168.2.51:8080...\n");
                int connect_result = lwip_connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
				// 检查连接是否立即失败。
                if (connect_result < 0 && errno != EINPROGRESS) {
                    LOG(LOG_LVL_ERROR, "lwip_connect immediate failure: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

				// 使用 select 检查连接状态。
                fd_set writefds, exceptfds;
                struct timeval tv;
                tv.tv_sec = 5;
                tv.tv_usec = 0;
                FD_ZERO(&writefds);
                FD_ZERO(&exceptfds);
                FD_SET(sockfd, &writefds);
                FD_SET(sockfd, &exceptfds);
                int select_result = lwip_select(sockfd + 1, NULL, &writefds, &exceptfds, &tv);
				// 检查 select 结果。
                // <= 0 超时或错误，FD_ISSET(sockfd, &exceptfds)：异常发生
                if (select_result <= 0 || FD_ISSET(sockfd, &exceptfds)) {
                    LOG(LOG_LVL_ERROR, "Socket Connect timed out or failed after 5s, errno=%d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                // 检查 socket 的错误状态
                // SO_ERROR：获取连接的错误码
                // 如果有错误，关闭 socket 并重试。
                int so_error = 0;
                socklen_t len = sizeof(so_error);
                if (lwip_getsockopt(sockfd, SOL_SOCKET, SO_ERROR, &so_error, &len) < 0 || so_error != 0) {
                    LOG(LOG_LVL_ERROR, "lwip_connect failed with error: %d\n", so_error);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                // 连接成功，设置接收超时。超时设置为 5S
                LOG(LOG_LVL_INFO, "Connected to 192.168.2.51:8080\n");//192.168.2.51
                tv.tv_sec = timeout_ms / 1000;
                tv.tv_usec = (timeout_ms % 1000) * 1000;

                if (lwip_setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, &tv, sizeof(tv)) < 0) {
                    LOG(LOG_LVL_ERROR, "lwip_setsockopt SO_RCVTIMEO failed: %d\n", errno);
                }
            }

			// 在 socket 连接有效且 WiFi 正常时，处理数据。
            while (sockfd >= 0 && netdev_got_ip()) {
				// 接收服务器数据，缓冲区大小为 512		
                char buffer[512];
                int len = lwip_recv(sockfd, buffer, sizeof(buffer) - 1, 0);
                
                if (len > 0) { // len > 
					buffer[len] = '\0';
                    // 检查临时缓冲区是否够
                    // temp_pos：当前写入位置。
					if (temp_pos + len < sizeof(temp_buffer)){
						// 将接收到的数据追加到 temp_buffer
						memcpy(temp_buffer + temp_pos, buffer, len);
						temp_pos += len; //写入长度 + len
						
                        // 解析 JSON。
						char *json_start = temp_buffer;
						while (json_start < temp_buffer + temp_pos) {
							char *json_end = memchr(json_start, '}', temp_pos - (json_start - temp_buffer));
							if (!json_end) {
								LOG(LOG_LVL_INFO, "Incomplete JSON, waiting for more data\n");
								break;
							}

							uint16_t json_len = json_end - json_start + 1;
							if (json_len >= 2 && json_start[0] == '{') {
								LOG(LOG_LVL_INFO, "Parsing JSON, len=%d: %.*s\n", json_len, json_len, json_start);
								parse_json((uint8_t*)json_start, json_len);
							} else {
								LOG(LOG_LVL_ERROR, "Invalid JSON start, skipping: %.*s\n", json_len, json_start);
							}

							json_start = json_end + 1;
						}

						uint16_t processed = json_start - temp_buffer;
						if (processed > 0) {
							uint16_t remaining = temp_pos - processed;
							if (remaining > 0) {
								memmove(temp_buffer, json_start, remaining);
							}
							temp_pos = remaining;
						}
					} else {
						LOG(LOG_LVL_ERROR, "Temp buffer overflow, resetting\n");
						temp_pos = 0;
					}
                    // 这段是测试用的，因为传递过来的图片 JOSN 刚好是 24965
                    // 所以此处就写死回传回去
					if(text_used == 24965){
						int total_sent = 0;
						while (total_sent < text_used) {
							int chunk_size = (text_used - total_sent > 521) ? 521 : (text_used - total_sent);
							int sent = lwip_send(sockfd, text_buffer + total_sent, chunk_size, 0);
							
							if (sent < 0) {
								LOG(LOG_LVL_ERROR, "Send error: %d", errno);
								break;
							} else {
								total_sent += sent;
								LOG(LOG_LVL_INFO, "Sent %d bytes, total sent: %d bytes", sent, total_sent);
							}
						}
					}
                } else if (len == 0) { // len == 0:?????
                    LOG(LOG_LVL_INFO, "Server closed connection\n");
                    lwip_close(sockfd);
                    sockfd = -1;
                    break;
                } else if (errno == EAGAIN || errno == ETIMEDOUT) {
                    OS_MsDelay(100);
                // ????:?? Socket?    
                } else {
                    LOG(LOG_LVL_ERROR, "lwip_recv failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    break;
                }

//                wifi_sta_get_rssi(&rssi);
//                LOG(LOG_LVL_INFO, "WiFi RSSI: %d dBm\n", rssi);
            }

            if (sockfd < 0) {
                LOG(LOG_LVL_ERROR, "Socket disconnect, retrying...\n");
                OS_MsDelay(2000);
            }
        }

        LOG(LOG_LVL_ERROR, "WiFi disconnected, waiting for reconnect...\n");
        OS_MsDelay(2000);
    }
}
```

## 实现二(改)

基于实现二发现使用 JSON 会有数据量变大、类型相关的问题(接收 JSON 字符串，发送二进制数据)；所以这里将其修改为统一使用二进制进行传输；

自定义数据格式如下：[0xAA] [Type] [2 bytes Length] [Data] [CRC16] [0xFF]

| 字段   | 说明                                                    |
| ------ | ------------------------------------------------------- |
| 0xAA   | **起始字节**，标识数据包开始                            |
| Type   | **指令类型**，决定数据的处理方式                        |
| Length | **2 字节数据长度**，指示 Escaped Data 的大小            |
| Data   | **转义后的数据**，防止特殊字符干扰                      |
| CRC16  | **校验码**，用于完整性校验(只校验 Data 部分)，CRC16_IBM |
| 0xFF   | **结束字节**，标识数据包结束                            |

自定义响应格式：[0xAA] [response_type] [0xFF]

| 0xAA          | **起始字节**，标识数据包开始       |
| ------------- | ---------------------------------- |
| response_type | **响应类型**，指示服务器的响应结果 |
| 0xFF          | **结束字节**，标识数据包结束       |

特殊要求：发送前客户端会发送整个图片的字节数(32 位变量，4 字节)，存入 MCU；通过累加 [2 bytes Length] 判断是否接收一整个图片完成。

**收发流程**

以缓冲区 30k 为例：(打印数据)

以缓冲区 30k 为例：(音频数据)
超过缓冲区一半，就把数据给喇叭
写入缓冲区，超过一半给喇叭

问题：

WiFi 连接正常，但是 socket 一直超时，因为我们使用的非阻塞模式，当 recv 返回 -1 的时候我们不要手动关闭 socket 可减少此类事件的发生；
我们当前使用的 socket 是基于 TCP 协议，所以要注意数据粘包的问题；
基于环形缓冲区，要注意写入和读取的问题；

```c
#define SUM_BUFFER_SIZE 30000    // 循环缓冲区
#define TEMP_BUFFER_SIZE 512    // 临时缓冲区
#define MAX_DATA_PER_PACKET 505 // 单个数据包允许的最大数据长度

uint8_t temp_buffer[TEMP_BUFFER_SIZE];   // 临时缓冲区
uint32_t temp_pos = 0;					 // temp_buffer 写入位置
uint32_t data_sum_size = 0;				 // 一段数据大小综合
uint8_t is_get_sum_size = 0;			 // 标记是否获取了 data_sum_size
uint8_t data_type = 0;			         // 数据类型
uint8_t text_buffer[SUM_BUFFER_SIZE];    // 循环缓冲区
uint32_t write_pos = 0;					 // 循环缓冲区的写入位置
uint32_t read_pos = 0;					 // 循环缓冲区的读取位置。
uint32_t total_written = 0;				 // 已经接收并存储的数据总量。

// 计算 CRC16 校验码(CRC16_IBM 初始值0x0000，低位在前，高位在后，结果与0x0000异或)
static uint16_t calculate_crc16(const uint8_t *data, uint16_t length) {
    uint16_t crc = 0x0000;
    for (uint16_t i = 0; i < length; i++) {
        crc ^= (uint16_t)data[i] << 8;
        for (uint8_t j = 0; j < 8; j++) {
            if (crc & 0x8000) crc = (crc << 1) ^ 0x8005; else crc <<= 1;
        }
    }
    return crc;
}

// 验证数据包的格式，
// buffer：指向数据缓冲区的指针（uint8_t *）
// pos: 当前缓冲区中已接收的数据长度
// 1 数据长度不足，-1 数据格式错误，2数据包格式正确
static int validate_format(uint8_t *buffer, uint32_t pos) {
    if (pos < 7) return 1;
    uint16_t data_length = (buffer[2] << 8) | buffer[3]; // 数据长度
    // 数据长度不合法
    if (data_length > TEMP_BUFFER_SIZE - 7) {
        LOG(LOG_LVL_ERROR, "Data length to big langth: %u\n", data_length);
        return -1;
    }
    uint16_t expected_pos = 4 + data_length + 2 + 1; // 完整数据包长度
    if (pos < expected_pos) return 1;
    if (buffer[0] != 0xAA || buffer[expected_pos - 1] != 0xFF || pos != expected_pos) {
        LOG(LOG_LVL_ERROR, "Data format error: head=0x%02X, tail=0x%02X, pos=%u, expected=%u\n",
            buffer[0], buffer[pos - 1], pos, expected_pos);
        return -1;
    }
    return 2; // 完整数据包
}

// 发送 ACK/NACK 0xAA 代表成功，0xEE 代表失败
static void send_ACK(int sockfd, uint8_t response_type) {
    uint8_t response[3] = {0xAA, response_type, 0xFF};
    int attempts = 5;
    int sent;
    // 重试发送循环
    // 循环最多执行 5 次，每次循环减少 attempts，直到成功发送或尝试次数耗尽。
    while (attempts-- > 0) {
        sent = lwip_send(sockfd, response, sizeof(response), 0);
        // 错误处理
        if (sent < 0) {
            LOG(LOG_LVL_ERROR, "send_ACK error, errno=%d, attempts left=%d\n", errno, attempts);
            if (errno == EAGAIN || errno == ETIMEDOUT) {
                OS_MsDelay(50); // 延迟一下再发
                continue;
            }
            break;
        // 如果发送的字节数 sent 不等于预期长度 3，记录错误并退出循环。    
        } else if (sent != sizeof(response)) {
            LOG(LOG_LVL_ERROR, "Partial ACK send: expected 3, sent %d\n", sent);
            break;
        // 如果发送成功（sent == 3），记录日志并返回。    
        } else {
            LOG(LOG_LVL_INFO, "Sent %s\n", response_type == 0xAA ? "ACK" : "NACK");
            return; // ????
        }
    }
    LOG(LOG_LVL_ERROR, "Failed to send ACK after retries\n");
}

// 接收 4 字节的 sum_size（总数据大小）。
static int receive_sum_size(int sockfd, uint32_t *sum_size) {
    uint8_t size_buffer[4];
    int size_pos = 0;
    char buffer[4];
    int len = lwip_recv(sockfd, buffer, sizeof(buffer), 0);
    if (len < 0) {
        //LOG(LOG_LVL_ERROR, " recv_sum_size error: %d\n", errno);
        return -1;
    } else if (len > 0) {
        LOG(LOG_LVL_INFO, "receive_sum_size len: %d\n", len);
        int i = 0;
        while (i < len && size_pos < 4) {
            size_buffer[size_pos] = buffer[i];
            size_pos++;
            i++;
        }
        if (size_pos == 4) {
            *sum_size = ((uint32_t)size_buffer[0] << 24) | ((uint32_t)size_buffer[1] << 16) |
                        ((uint32_t)size_buffer[2] << 8) | (uint32_t)size_buffer[3];
            LOG(LOG_LVL_INFO, "Received total size: %u bytes\n", *sum_size);
            send_ACK(sockfd, 0xAA);
            size_pos = 0;
            return 1;
        }
    }
    return 0;
}

// 获取可用剩余空间
static uint32_t get_free_space(uint32_t write_pos, uint32_t read_pos) {
    if ((write_pos + 1) % SUM_BUFFER_SIZE == read_pos) {
        return 0; // 缓冲区满
    } 
    if (write_pos >= read_pos) {
        return SUM_BUFFER_SIZE - (write_pos - read_pos) - 1; // 预留 1 个字节空间
    } else {
        return read_pos - write_pos - 1; // 预留 1 个字节空间
    }
}

// 入环形缓冲区，如果超出结尾，则分两次写入。
static int write_circular_buffer(uint8_t *buffer, uint32_t *write_pos, uint32_t *read_pos, uint8_t *data, uint16_t data_length) {
    uint32_t free_space = get_free_space(*write_pos, *read_pos);

    // 预留一个字节以确保环形缓冲区不会满
    if (free_space <= data_length) { 
        LOG(LOG_LVL_ERROR, "Not enough space: available=%u, required=%u\n", free_space, data_length);
        return -1;
    }

    uint32_t space_to_end = SUM_BUFFER_SIZE - *write_pos;

    if (space_to_end >= data_length) {
        memcpy(&buffer[*write_pos], data, data_length);
    } else {
        memcpy(&buffer[*write_pos], data, space_to_end);
        memcpy(&buffer[0], data + space_to_end, data_length - space_to_end);
    }

    // 更新写指针
    *write_pos = (*write_pos + data_length) % SUM_BUFFER_SIZE;

    return 0;
}

// Sokcet 数据接收
// 0 超时等待，-1 len <= 0
static int receive_data(int sockfd, uint32_t sum_size, uint8_t *text_buffer, uint32_t *write_pos, uint32_t *read_pos) {
    static uint8_t buffer[512];

    int len = lwip_recv(sockfd, buffer, sizeof(buffer), 0);
    if (len < 0) {
        if (errno == EAGAIN || errno == ETIMEDOUT) {
            OS_MsDelay(100);
            return 0; // 继续等待
        }
        LOG(LOG_LVL_ERROR, "receive_data failed: %d\n", errno);
        return -1;
    } else if (len == 0) {
        LOG(LOG_LVL_INFO, "Server closed connection\n");
        return -1;
    }
	LOG(LOG_LVL_INFO, "Received_data %d bytes...\n", len);
    int i = 0;
    while (i < len) {
        if (temp_pos < TEMP_BUFFER_SIZE) {
            temp_buffer[temp_pos] = buffer[i];
            temp_pos++;
            int validation_result = validate_format(temp_buffer, temp_pos);

            if (validation_result == -1) { // 校验失败，丢弃数据
                send_ACK(sockfd, 0xEE);
                temp_pos = 0;
                return 0;
            }

            if (validation_result == 2) { // 完整数据包
                uint16_t data_length = (temp_buffer[2] << 8) | temp_buffer[3];
                data_type = temp_buffer[1];

                uint8_t *data = &temp_buffer[4];
                uint16_t recv_crc = (temp_buffer[4 + data_length] << 8) | temp_buffer[5 + data_length];
                uint16_t calculated_crc = calculate_crc16(data, data_length);

                if (calculated_crc != recv_crc) {
                    LOG(LOG_LVL_ERROR, "CRC16 mismatch: received=0x%04X, calculated=0x%04X\n", recv_crc, calculated_crc);
                    send_ACK(sockfd, 0xEE);
                } else {
                    if (write_circular_buffer(text_buffer, write_pos, read_pos, data, data_length) != 0) {
                        LOG(LOG_LVL_ERROR, "Write to buffer failed\n");
                        send_ACK(sockfd, 0xEE);
                    } else {
                        total_written += data_length;
                        send_ACK(sockfd, 0xAA);
                        if()
                        LOG(LOG_LVL_INFO, "CRC match, total_written=%u\n", total_written);
                        temp_pos = 0;
                        return 2; // 数据包成功处理
                    }
                }
                temp_pos = 0;
            }
        } else { 
            send_ACK(sockfd, 0xEE);
            temp_pos = 0;
            return 0;
        }
        i++;
    }
    return 0;
}

static void send_data(int sockfd, uint8_t type, uint16_t length, uint8_t *data) {
    uint16_t remaining_length = length;         // 剩余待发送的数据长度
    uint8_t *data_ptr = data;                   // 数据指针，指向当前发送位置
    uint8_t send_buffer[7 + MAX_DATA_PER_PACKET]; // 缓冲区，最大支持 512 字节

    while (remaining_length > 0) {
        // 计算本次发送的数据长度，最大 505 字节（确保总包长 <= 512 字节）
        uint16_t chunk_length = (remaining_length > MAX_DATA_PER_PACKET) ? MAX_DATA_PER_PACKET : remaining_length;
        uint16_t total_packet_length = 7 + chunk_length; // 总包长 = 7（头尾）+ 数据长度

        // 构建数据包
        send_buffer[0] = 0xAA;                    // 帧头
        send_buffer[1] = type;                    // 数据类型
        send_buffer[2] = (chunk_length >> 8) & 0xFF; // 长度高字节
        send_buffer[3] = chunk_length & 0xFF;     // 长度低字节
        memcpy(&send_buffer[4], data_ptr, chunk_length); // 复制数据
        uint16_t crc = calculate_crc16(data_ptr, chunk_length); // 计算 CRC16
        send_buffer[4 + chunk_length] = (crc >> 8) & 0xFF; // CRC 高字节
        send_buffer[5 + chunk_length] = crc & 0xFF; // CRC 低字节
        send_buffer[6 + chunk_length] = 0xFF;     // 帧尾

        // 发送数据包
        int sent = lwip_send(sockfd, send_buffer, total_packet_length, 0);
        if (sent < 0) {
            LOG(LOG_LVL_ERROR, "Send error: %d\n", errno);
            break;
        } else if (sent != total_packet_length) {
            LOG(LOG_LVL_ERROR, "Partial send: expected %d, sent %d\n", total_packet_length, sent);
            break;
        } else {
            LOG(LOG_LVL_INFO, "Sent %d bytes (data length: %u)\n", sent, chunk_length);
        }

        // 更新指针和剩余长度
        remaining_length -= chunk_length;
        data_ptr += chunk_length;

        // 短暂延迟，避免发送过快（可选，根据需求调整）
        OS_MsDelay(10);
    }
}

static void socket_task_entry(void *params) {
    LN_UNUSED(params);
    int sockfd = -1;
    struct sockaddr_in server_addr;
    int timeout_ms = 5000;

    while (1) {
        if (!netdev_got_ip()) {
            LOG(LOG_LVL_INFO, "No IP, waiting for WiFi...\n");
            OS_MsDelay(1000);
            continue;
        }
        tcpip_ip_info_t ip_info;
        netdev_get_ip_info(NETIF_IDX_STA, &ip_info);
        LOG(LOG_LVL_INFO, "IP: %s, Mask: %s, Gateway: %s\n",
            ip4addr_ntoa(&ip_info.ip), ip4addr_ntoa(&ip_info.netmask), ip4addr_ntoa(&ip_info.gw));
        
        int8_t rssi;
        wifi_sta_get_rssi(&rssi);
        LOG(LOG_LVL_INFO, "WiFi RSSI: %d dBm\n", rssi);

        while (netdev_got_ip()) {
            if (sockfd < 0) {
                sockfd = lwip_socket(AF_INET, SOCK_STREAM, 0);
                if (sockfd < 0) {
                    LOG(LOG_LVL_ERROR, "Socket creation failed: %d\n", errno);
                    OS_MsDelay(2000);
                    continue;
                }
                LOG(LOG_LVL_INFO, "Socket create ID=%d\n", sockfd);
                // 设置为非阻塞模式
                int flags = 1;
                if (lwip_ioctl(sockfd, FIONBIO, &flags) < 0) {
                    LOG(LOG_LVL_ERROR, "socket ioctl error: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }
                memset(&server_addr, 0, sizeof(server_addr));
                server_addr.sin_family = AF_INET;
                server_addr.sin_port = lwip_htons(8080);
                server_addr.sin_addr.s_addr = inet_addr("192.168.106.181");
                LOG(LOG_LVL_INFO, "Connecting to 192.168.106.181:8080...\n");
                int connect_result = lwip_connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
                if (connect_result < 0 && errno != EINPROGRESS) {
                    LOG(LOG_LVL_ERROR, "lwip_connect failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }
                fd_set writefds, exceptfds;
                //struct timeval tv = {5, 0};
                struct timeval tv;
                tv.tv_sec = 10;
                tv.tv_usec = 0;
                FD_ZERO(&writefds);
                FD_ZERO(&exceptfds);
                FD_SET(sockfd, &writefds);
                FD_SET(sockfd, &exceptfds);
                int select_result = lwip_select(sockfd + 1, NULL, &writefds, &exceptfds, &tv);
                if (select_result <= 0 || FD_ISSET(sockfd, &exceptfds)) {
                    LOG(LOG_LVL_ERROR, "Connect timed out or failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }
                int so_error = 0;
                socklen_t len = sizeof(so_error);
                if (lwip_getsockopt(sockfd, SOL_SOCKET, SO_ERROR, &so_error, &len) < 0 || so_error != 0) {
                    LOG(LOG_LVL_ERROR, "Connect failed with error: %d\n", so_error);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }
                LOG(LOG_LVL_INFO, "Connected to 192.168.106.181:8080\n");
                tv.tv_sec = timeout_ms / 1000;
                tv.tv_usec = (timeout_ms % 1000) * 1000;
                if (lwip_setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, &tv, sizeof(tv)) < 0) {
                    LOG(LOG_LVL_ERROR, "setsockopt SO_RCVTIMEO failed: %d\n", errno);
                }
            }
            while (sockfd >= 0 && netdev_got_ip()) {
                if (!is_get_sum_size) {
                    if (receive_sum_size(sockfd, &data_sum_size) == 1) {
                        is_get_sum_size = 1;
                    }
                } else {
                    // 接收数据
                    int result = receive_data(sockfd, data_sum_size, text_buffer, &write_pos, &read_pos);
                    if (result == -1) {
                        LOG(LOG_LVL_ERROR, "receive_data result -1...\n");
                        lwip_close(sockfd);
                        sockfd = -1;
                        break;
                    }
                    
                    // 数据包写入成功处理时，检查是否需要发送
                    if (result == 2) {
                        // 计算环形缓冲区中当前可读取的数据量（即已写入但尚未发送的数据）。
                        //write_pos 是环形缓冲区的写入指针，表示最新的写入位置。
						//read_pos 是环形缓冲区的读取指针，表示下一次读取的起始位置。
						//SUM_BUFFER_SIZE 是环形缓冲区的总容量（例如 30000 字节）。
                        uint32_t available_data = (write_pos >= read_pos) 
                            ? (write_pos - read_pos) 
                            : (SUM_BUFFER_SIZE - read_pos + write_pos);
                        LOG(LOG_LVL_INFO, "available_data=%u\n", available_data);

                        // 计算一个动态发送阈值，用于决定何时触发数据发送
                        uint32_t send_threshold;
                        if (data_sum_size >= SUM_BUFFER_SIZE / 2) {
                            send_threshold = SUM_BUFFER_SIZE / 2; // 缓冲区一半满时发送
                        } else {
                            send_threshold = data_sum_size; // 小于一半时，全部数据到达时发送
                        }

                        //判断当前可读取的数据量是否大于等于发送阈值。
                        if (available_data >= send_threshold) {
                            uint16_t length = available_data;
                            if (length > 0) {
                                send_data(sockfd, data_type, length, &text_buffer[read_pos]);
                                read_pos = (read_pos + length) % SUM_BUFFER_SIZE;
                                LOG(LOG_LVL_INFO, "Sent %u bytes, new read_pos=%u\n", length, read_pos);
                            }
                        }
                    }
                    
                    if (total_written >= data_sum_size) {
                        uint32_t available_data = (write_pos >= read_pos) 
                            ? (write_pos - read_pos) 
                            : (SUM_BUFFER_SIZE - read_pos + write_pos);
                        if (available_data > 0) {
                            send_data(sockfd, data_type, available_data, &text_buffer[read_pos]);
                            read_pos = (read_pos + available_data) % SUM_BUFFER_SIZE;
                            LOG(LOG_LVL_INFO, "Final send: Sent %u bytes, new read_pos=%u\n", available_data, read_pos);
                        }
                        is_get_sum_size = 0;
                        total_written = 0;
                        write_pos = 0;
                        read_pos = 0; // 重置
                        LOG(LOG_LVL_INFO, "Reset state for next transmission\n");
                    }
                }                    
            }
            if (sockfd < 0) {
                LOG(LOG_LVL_ERROR, "Socket disconnect, retrying...\n");
                OS_MsDelay(2000);
            }
        }
        LOG(LOG_LVL_ERROR, "WiFi disconnected, waiting...\n");
        OS_MsDelay(2000);
    }
}
```

## 实现三

要区分打印数据和音频数据，利用现用自定义格式重新规划收发流程：

收发流程如下：
**引导阶段**：
MCU 接收一个响应包 [0xAA][0x01][0xFF]，表示数据传输开始。
紧接着接收一个数据包：

- [0xAA][0x11][0x00][0x00][CRC16][0xFF]：打印数据引导。
- [0xAA][0x12][0x00][0x00][CRC16][0xFF]：音频数据引导。

**数据接收阶段**：

- 打印数据（0x11）：
  - 接收 4 字节总长度。
  - 根据总长度接收后续数据包（如 [0xAA][0x11][Length][Data][CRC16][0xFF]）。
  - 每包回写 [0xAA][0xAA][0xFF]（ACK）或 [0xAA][0xEE][0xFF]（NACK）。
- 音频数据（0x12）：
  - 直接接收数据包（如 [0xAA][0x12][Length][Data][CRC16][0xFF]）。
  - 每包回写 ACK/NACK。
  - 无需总长度，动态累积数据。

**结束阶段**：

- 发送端发送 [0xAA][0x02][0xFF] 表示传输结束。
- MCU 检测到此包后停止接收。

**拆包与粘包**
在 TCP 传输中，由于**流式**传输的特性，数据可能会被拆分成多个小包发送（拆包）或者多个小包合并成一个大包接收（粘包）
拆包：如果 send(1024)，recv(512)，TCP 在协议层面会将一个 1024 大小的包拆成两个 512 大小的包
粘包：如果 send(512)，recv(1024)，TCP 在协议层面会将 两个 512 大小的包组合成一个 1024 大小的包

**数据流示例**

```c
// 打印数据
[0xAA][0x01][0xFF]                     // 引导开始
[0xAA][0xAA][0xFF]                     // ACK
[0xAA][0x11][0x00][0x00][CRC16][0xFF]  // 打印引导
[0xAA][0xAA][0xFF]                     // ACK
[0x00][0x00][0x61][0xB5]               // 总长度 24965
[0xAA][0x11][0x01][0xF9][Data 505][CRC16][0xFF]  // 数据包1
[0xAA][0xAA][0xFF]                     // ACK
...
[0xAA][0x02][0xFF]                     // 结束
    
// 音频数据
[0xAA][0x01][0xFF]                     // 引导开始
[0xAA][0xAA][0xFF]                     // ACK
[0xAA][0x12][0x00][0x00][CRC16][0xFF]  // 音频引导
[0xAA][0xAA][0xFF]                     // ACK
[0xAA][0x12][0x01][0xF9][Data 505][CRC16][0xFF]  // 数据包1
[0xAA][0xAA][0xFF]                     // ACK
...
[0xAA][0x02][0xFF]                     // 结束
```

```c
#define SUM_BUFFER_SIZE 30000    // 循环缓冲区
#define TEMP_BUFFER_SIZE 512     // 临时缓冲区
#define MAX_DATA_PER_PACKET 505  // 单个数据包允许的最大数据长度

// 状态定义
typedef enum {
    STATE_IDLE,      // 等待开始信号
    STATE_TYPE,      // 等待类型包
    STATE_PRINT,     // 打印数据模式
    STATE_AUDIO      // 音频数据模式
} State_t;

uint8_t temp_buffer[TEMP_BUFFER_SIZE];   // 临时缓冲区
uint32_t temp_pos = 0;                   // temp_buffer 写入位置
uint32_t data_sum_size = 0;              // 总数据大小（打印模式）
uint8_t is_get_sum_size = 0;             // 是否获取总长度
uint8_t data_type = 0;                   // 数据类型
uint8_t text_buffer[SUM_BUFFER_SIZE];    // 循环缓冲区
uint32_t write_pos = 0;                  // 写入位置
uint32_t read_pos = 0;                   // 读取位置
uint32_t total_written = 0;              // 已接收数据总量
State_t state = STATE_IDLE;              // 当前状态

// 计算 CRC16
static int calculate_crc16(const uint8_t *data, uint16_t length) {
    uint16_t crc = 0x0000;
    for (uint16_t i = 0; i < length; i++) {
        crc ^= (uint16_t)data[i] << 8;
        for (uint8_t j = 0; j < 8; j++) {
            if (crc & 0x8000) crc = (crc << 1) ^ 0x8005; else crc <<= 1;
        }
    }
    return crc;
}

// 验证数据包格式
static int validate_format(uint8_t *buffer, uint32_t pos) {
    if (pos < 7) return 1;
    uint16_t data_length = (buffer[2] << 8) | buffer[3];
    if (data_length > TEMP_BUFFER_SIZE - 7) {
        LOG(LOG_LVL_ERROR, "Data length too big: %u\n", data_length);
        return -1;
    }
    uint16_t expected_pos = 4 + data_length + 2 + 1;
    if (pos < expected_pos) return 1;
    if (buffer[0] != 0xAA || buffer[expected_pos - 1] != 0xFF || pos != expected_pos) {
        LOG(LOG_LVL_ERROR, "Data format error: head=0x%02X, tail=0x%02X, pos=%u, expected=%u\n",
            buffer[0], buffer[pos - 1], pos, expected_pos);
        return -1;
    }
    return 2;
}

// 发送 ACK/NACK
static void send_ACK(int sockfd, uint8_t response_type) {
    uint8_t response[3] = {0xAA, response_type, 0xFF};
    int attempts = 5;
    int sent;
    while (attempts-- > 0) {
        sent = lwip_send(sockfd, response, sizeof(response), 0);
        if (sent < 0) {
            LOG(LOG_LVL_ERROR, "send_ACK error, errno=%d, attempts left=%d\n", errno, attempts);
            if (errno == EAGAIN || errno == ETIMEDOUT) {
                OS_MsDelay(50);
                continue;
            }
            break;
        } else if (sent != sizeof(response)) {
            LOG(LOG_LVL_ERROR, "Partial ACK send: expected 3, sent %d\n", sent);
            break;
        } else {
            switch(response_type){
				case 0xAA: LOG(LOG_LVL_INFO, "Sent %s\n", "ACK_0xAA");break;
				case 0xEE: LOG(LOG_LVL_INFO, "Sent %s\n", "NACK_0xEE");break;
				case 0x02: LOG(LOG_LVL_INFO, "Sent %s\n", "ACK_0x02");break;
                case 0x01: LOG(LOG_LVL_INFO, "Sent %s\n", "ACK_0x01");break;
			}
            return;
        }
    }
    LOG(LOG_LVL_ERROR, "Failed to send ACK after retries\n");
}

// 接收 4 字节总长度
static int receive_sum_size(int sockfd, uint32_t *sum_size) {
    uint8_t size_buffer[4];
    int size_pos = 0;
    char buffer[4];
    int len = lwip_recv(sockfd, buffer, sizeof(buffer), 0);
    if (len < 0) {
        //LOG(LOG_LVL_ERROR, " failed: %d\n", errno);
        return -1;
    } else if (len > 0) {
        LOG(LOG_LVL_INFO, "receive_sum_size len: %d\n", len);
        int i = 0;
        while (i < len && size_pos < 4) {
            size_buffer[size_pos] = buffer[i];
            size_pos++;
            i++;
        }
        if (size_pos == 4) {
            *sum_size = ((uint32_t)size_buffer[0] << 24) | ((uint32_t)size_buffer[1] << 16) |
                        ((uint32_t)size_buffer[2] << 8) | (uint32_t)size_buffer[3];
            LOG(LOG_LVL_INFO, "Received total size: %u bytes\n", *sum_size);
            send_ACK(sockfd, 0xAA);
            return 1;
        }
    }
    return 0;
}

// 获取环形缓冲区剩余空间
static int get_free_space(uint32_t write_pos, uint32_t read_pos) {
    if ((write_pos + 1) % SUM_BUFFER_SIZE == read_pos) {
        return 0;
    }
    if (write_pos >= read_pos) {
        return SUM_BUFFER_SIZE - (write_pos - read_pos) - 1;
    } else {
        return read_pos - write_pos - 1;
    }
}

// 写入环形缓冲区
static uint8_t write_circular_buffer(uint8_t *buffer, uint32_t *write_pos, uint32_t *read_pos, uint8_t *data, uint16_t data_length) {
    uint32_t free_space = get_free_space(*write_pos, *read_pos);
    if (free_space <= data_length) {
        LOG(LOG_LVL_ERROR, "Not enough space: available=%u, required=%u\n", free_space, data_length);
        return -1;
    }
    uint32_t space_to_end = SUM_BUFFER_SIZE - *write_pos;
    if (space_to_end >= data_length) {
        memcpy(&buffer[*write_pos], data, data_length);
    } else {
        memcpy(&buffer[*write_pos], data, space_to_end);
        memcpy(&buffer[0], data + space_to_end, data_length - space_to_end);
    }
    *write_pos = (*write_pos + data_length) % SUM_BUFFER_SIZE;
    return 0;
}

// 接收数据(包含粘包处理逻辑)
// 0: ，-1:表示服务端关闭连接
static int receive_data(int sockfd, uint8_t *text_buffer, uint32_t *write_pos, uint32_t *read_pos) {
    static uint8_t buffer[1024];           // 接收缓冲区
    static uint8_t temp_buffer[1024];      // 临时缓冲区，用于累积和拼接数据
    static uint32_t temp_pos = 0;         // 临时缓冲区的写位置
    static int has_partial = 0;           // 标志：是否有未完成的半包数据

    // 如果有未处理的包，先尝试处理
    // has_partial = 1说明有未处理的包，
    // temp_pos 说明缓冲区中还有没用写入全局缓存区的数据
    if (has_partial && temp_pos > 0) {
        // 验证未写入的数据是否是整包
        int validation_result = validate_format(temp_buffer, temp_pos);
        if (validation_result == 2) { // 整包的处理逻辑
            // 处理完整的半包数据
            uint16_t data_length = (temp_buffer[2] << 8) | temp_buffer[3];
            uint8_t data_type = temp_buffer[1];
            uint8_t *data = &temp_buffer[4];
            uint16_t recv_crc = (temp_buffer[4 + data_length] << 8) | temp_buffer[5 + data_length];
            uint16_t calculated_crc = calculate_crc16(data, data_length);

            if (calculated_crc != recv_crc) {
                LOG(LOG_LVL_ERROR, "CRC16 mismatch: received=0x%04X, calculated=0x%04X\n", recv_crc, calculated_crc);
                send_ACK(sockfd, 0xEE);
            } else {
                if (data_length == 0 && (data_type == 0x11 || data_type == 0x12)) {
                    LOG(LOG_LVL_INFO, "recv type : 0x%02X\n", data_type);
                    send_ACK(sockfd, 0xAA);
                    state = (data_type == 0x11) ? STATE_PRINT : STATE_AUDIO;
                } else if (write_circular_buffer(text_buffer, write_pos, read_pos, data, data_length) != 0) {
                    LOG(LOG_LVL_ERROR, "Write to buffer failed\n");
                    send_ACK(sockfd, 0xEE);
                } else {
                    total_written += data_length;
                    send_ACK(sockfd, 0xAA);
                    LOG(LOG_LVL_INFO, "CRC match, total_written=%u\n", total_written);
                }
            }
            temp_pos = 0;
            has_partial = 0; // 重置半包标志
        } else if (validation_result == -1) { // 数据包格式错误
            send_ACK(sockfd, 0xEE);
            temp_pos = 0;
            has_partial = 0;
            return 0;
        }
        // 如果仍不完整，继续等待新数据
    }

    // 接收新数据
    int len = lwip_recv(sockfd, buffer, sizeof(buffer), 0);
    if (len < 0) {
        if (errno == EAGAIN || errno == ETIMEDOUT) {
            OS_MsDelay(100);
            return 0;
        }
        LOG(LOG_LVL_ERROR, "receive_data len < 0: %d\n", errno);
        return -1;
    } else if (len == 0) {
        LOG(LOG_LVL_INFO, "Server closed connection\n");
        return -1;
    }

    int i = 0;
    while (i < len) {
        // 如果 temp_buffer 已满，丢弃并重置
        if (temp_pos >= sizeof(temp_buffer)) {
            send_ACK(sockfd, 0xEE);
            temp_pos = 0;
            has_partial = 0;
            LOG(LOG_LVL_INFO, "temp_pos >= sizeof(temp_buffer)...\n");
            return 0;
        }

        // 将新字节追加到 temp_buffer
        temp_buffer[temp_pos] = buffer[i];
        temp_pos++;

        int validation_result = validate_format(temp_buffer, temp_pos);
        if (validation_result == -1) {
            send_ACK(sockfd, 0xEE);
            temp_pos = 0;
            has_partial = 0;
            return 0;
        } else if (validation_result == 2) {
            // 处理完整包
            uint16_t data_length = (temp_buffer[2] << 8) | temp_buffer[3];
            uint8_t data_type = temp_buffer[1];
            uint8_t *data = &temp_buffer[4];
            uint16_t recv_crc = (temp_buffer[4 + data_length] << 8) | temp_buffer[5 + data_length];
            uint16_t calculated_crc = calculate_crc16(data, data_length);

            if (calculated_crc != recv_crc) {
                LOG(LOG_LVL_ERROR, "CRC16 mismatch: received=0x%04X, calculated=0x%04X\n", recv_crc, calculated_crc);
                send_ACK(sockfd, 0xEE);
            } else {
                if (data_length == 0 && (data_type == 0x11 || data_type == 0x12)) {
                    LOG(LOG_LVL_INFO, "recv type : 0x%02X\n", data_type);
                    send_ACK(sockfd, 0xAA);
                    state = (data_type == 0x11) ? STATE_PRINT : STATE_AUDIO;
                } else if (write_circular_buffer(text_buffer, write_pos, read_pos, data, data_length) != 0) {
                    LOG(LOG_LVL_ERROR, "Write to buffer failed\n");
                    send_ACK(sockfd, 0xEE);
                } else {
                    total_written += data_length;
                    send_ACK(sockfd, 0xAA);
                    LOG(LOG_LVL_INFO, "CRC match, total_written=%u\n", total_written);
                }
            }
            temp_pos = 0;
            has_partial = 0;

            // 检查剩余字节是否为新包的开始
            if (i + 1 < len) {
                if (buffer[i + 1] != 0xAA) {
                    LOG(LOG_LVL_INFO, "Discarding invalid data after packet: %02X\n", buffer[i + 1]);
                    return 0; // 丢弃后续脏数据
                }
                // 如果是 0xAA，继续循环处理下一包
            }
        }
        i++;
    }

    // 检查是否以 0xAA 结尾且不完整（半包）
    if (temp_pos > 0 && temp_buffer[temp_pos - 1] == 0xAA && validate_format(temp_buffer, temp_pos) != 2) {
        has_partial = 1; // 标记为半包，等待下次拼接
        LOG(LOG_LVL_INFO, "Partial packet detected, waiting for more data\n");
    }

    return 0;
}

// 发送数据
static void send_data(int sockfd, uint8_t type, uint16_t length, uint8_t *data) {
    uint8_t send_buffer[7 + MAX_DATA_PER_PACKET];

    // 发送开始响应包
    send_ACK(sockfd, 0x01);

    // 发送类型引导包
    send_buffer[0] = 0xAA;
    send_buffer[1] = type;
    send_buffer[2] = 0x00;
    send_buffer[3] = 0x00;
    uint16_t crc = calculate_crc16(&send_buffer[4], 0); // 长度为 0，数据为空
    send_buffer[4] = (crc >> 8) & 0xFF;
    send_buffer[5] = crc & 0xFF;
    send_buffer[6] = 0xFF;
    int sent = lwip_send(sockfd, send_buffer, 7, 0);
    if (sent != 7) {
        LOG(LOG_LVL_ERROR, "Send type guide failed: expected 7, sent %d\n", sent);
        return;
    }
    LOG(LOG_LVL_INFO, "Sent type guide: 0x%02X\n", type);

    // 发送数据包
    uint16_t remaining_length = length;
    uint8_t *data_ptr = data;

    while (remaining_length > 0) {
        uint16_t chunk_length = (remaining_length > MAX_DATA_PER_PACKET) ? MAX_DATA_PER_PACKET : remaining_length;
        uint16_t total_packet_length = 7 + chunk_length;

        send_buffer[0] = 0xAA;
        send_buffer[1] = type;
        send_buffer[2] = (chunk_length >> 8) & 0xFF;
        send_buffer[3] = chunk_length & 0xFF;
        memcpy(&send_buffer[4], data_ptr, chunk_length);
        crc = calculate_crc16(data_ptr, chunk_length);
        send_buffer[4 + chunk_length] = (crc >> 8) & 0xFF;
        send_buffer[5 + chunk_length] = crc & 0xFF;
        send_buffer[6 + chunk_length] = 0xFF;

        sent = lwip_send(sockfd, send_buffer, total_packet_length, 0);
        if (sent != total_packet_length) {
            LOG(LOG_LVL_ERROR, "Send data failed: expected %d, sent %d\n", total_packet_length, sent);
            break;
        }
        LOG(LOG_LVL_INFO, "Sent %d bytes (data length: %u)\n", sent, chunk_length);

        remaining_length -= chunk_length;
        data_ptr += chunk_length;
    }
}

// 主任务
static void socket_task_entry(void *params) {
    LN_UNUSED(params);
    int sockfd = -1;
    struct sockaddr_in server_addr;
    int timeout_ms = 5000;

    while (1) {
        if (!netdev_got_ip()) {
            LOG(LOG_LVL_INFO, "No IP, waiting for WiFi...\n");
            OS_MsDelay(1000);
            continue;
        }

        tcpip_ip_info_t ip_info;
        netdev_get_ip_info(NETIF_IDX_STA, &ip_info);
        LOG(LOG_LVL_INFO, "IP: %s, Mask: %s, Gateway: %s\n",
            ip4addr_ntoa(&ip_info.ip), ip4addr_ntoa(&ip_info.netmask), ip4addr_ntoa(&ip_info.gw));

        int8_t rssi;
        wifi_sta_get_rssi(&rssi);
        LOG(LOG_LVL_INFO, "WiFi RSSI: %d dBm\n", rssi);

        while (netdev_got_ip()) {
            if (sockfd < 0) {
                sockfd = lwip_socket(AF_INET, SOCK_STREAM, 0);
                if (sockfd < 0) {
                    LOG(LOG_LVL_ERROR, "Socket creation failed: %d\n", errno);
                    OS_MsDelay(2000);
                    continue;
                }
                LOG(LOG_LVL_INFO, "Socket create ID=%d\n", sockfd);

                int flags = 1;
                if (lwip_ioctl(sockfd, FIONBIO, &flags) < 0) {
                    LOG(LOG_LVL_ERROR, "socket ioctl error: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                memset(&server_addr, 0, sizeof(server_addr));
                server_addr.sin_family = AF_INET;
                server_addr.sin_port = lwip_htons(8080);
                server_addr.sin_addr.s_addr = inet_addr("192.168.106.181");
                LOG(LOG_LVL_INFO, "Connecting to 192.168.106.181:8080...\n");

                int connect_result = lwip_connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
                if (connect_result < 0 && errno != EINPROGRESS) {
                    LOG(LOG_LVL_ERROR, "lwip_connect failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                fd_set writefds, exceptfds;
                struct timeval tv;
                tv.tv_sec = 10;
                tv.tv_usec = 0;
                FD_ZERO(&writefds);
                FD_ZERO(&exceptfds);
                FD_SET(sockfd, &writefds);
                FD_SET(sockfd, &exceptfds);
                int select_result = lwip_select(sockfd + 1, NULL, &writefds, &exceptfds, &tv);
                if (select_result <= 0 || FD_ISSET(sockfd, &exceptfds)) {
                    LOG(LOG_LVL_ERROR, "Connect timed out or failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                int so_error = 0;
                socklen_t len = sizeof(so_error);
                if (lwip_getsockopt(sockfd, SOL_SOCKET, SO_ERROR, &so_error, &len) < 0 || so_error != 0) {
                    LOG(LOG_LVL_ERROR, "Connect failed with error: %d\n", so_error);
                    lwip_close(sockfd);
                    sockfd = -1;
                    OS_MsDelay(2000);
                    continue;
                }

                LOG(LOG_LVL_INFO, "Connected to 192.168.106.181:8080\n");
                tv.tv_sec = timeout_ms / 1000;
                tv.tv_usec = (timeout_ms % 1000) * 1000;
                if (lwip_setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, &tv, sizeof(tv)) < 0) {
                    LOG(LOG_LVL_ERROR, "setsockopt SO_RCVTIMEO failed: %d\n", errno);
                }
            }

            while (sockfd >= 0 && netdev_got_ip()) {
                uint8_t response[3];
                int len;

                switch (state) {
                    case STATE_IDLE: // 等待开始信号
                        len = lwip_recv(sockfd, response, 3, 0);
                        if (len == 3 && response[0] == 0xAA && response[1] == 0x01 && response[2] == 0xFF) {
                            LOG(LOG_LVL_INFO, "Received start signal\n");
                            send_ACK(sockfd, 0xAA);
                            state = STATE_TYPE;
                        }
                        break;

                    case STATE_TYPE: // 等待类型包
                        receive_data(sockfd, text_buffer, &write_pos, &read_pos);
                        break;

                    case STATE_PRINT: // 打印业务
                        if (!is_get_sum_size) {
                            if (receive_sum_size(sockfd, &data_sum_size) == 1) {
                                is_get_sum_size = 1;
                            }
                        } else {
                            int result = receive_data(sockfd, text_buffer, &write_pos, &read_pos);
                            if (result == -1) { // -1 recv 返回 0
                                lwip_close(sockfd);
                                sockfd = -1;
                                break;
                            }

                            if (result == 0) { // 
                                len = lwip_recv(sockfd, response, 3, 0);
                                if (len == 3 && response[0] == 0xAA && response[1] == 0x02 && response[2] == 0xFF) {
                                    LOG(LOG_LVL_INFO, "Received end signal\n");
                                    send_ACK(sockfd, 0xAA);
                                    state = STATE_IDLE;
                                    is_get_sum_size = 0;
                                    total_written = 0;
                                    write_pos = 0;
                                    read_pos = 0;
                                    break;
                                }
                            }

                            if (total_written >= data_sum_size) {
                                uint32_t available_data = (write_pos >= read_pos) 
                                    ? (write_pos - read_pos) 
                                    : (SUM_BUFFER_SIZE - read_pos + write_pos);
                                if (available_data > 0) {
                                    send_data(sockfd, data_type, available_data, &text_buffer[read_pos]);
                                    read_pos = (read_pos + available_data) % SUM_BUFFER_SIZE;
                                    LOG(LOG_LVL_INFO, "Final send: Sent %u bytes, new read_pos=%u\n", available_data, read_pos);
                                }
                                // 等待结束信号
                            }
                        }
                        break;

                    case STATE_AUDIO:  // 音频业务
                        int result = receive_data(sockfd, text_buffer, &write_pos, &read_pos);
                        if (result == -1) {
                            lwip_close(sockfd);
                            sockfd = -1;
                            break;
                        }

                        if (result == 0) {
                            len = lwip_recv(sockfd, response, 3, 0);
                            if (len == 3 && response[0] == 0xAA && response[1] == 0x02 && response[2] == 0xFF) {
                                LOG(LOG_LVL_INFO, "Received end signal\n");
                                send_ACK(sockfd, 0xAA);
                                uint32_t available_data = (write_pos >= read_pos) 
                                    ? (write_pos - read_pos) 
                                    : (SUM_BUFFER_SIZE - read_pos + write_pos);
                                if (available_data > 0) {
                                    send_data(sockfd, data_type, available_data, &text_buffer[read_pos]);
                                    read_pos = (read_pos + available_data) % SUM_BUFFER_SIZE;
                                    LOG(LOG_LVL_INFO, "Final send: Sent %u bytes, new read_pos=%u\n", available_data, read_pos);
                                }
                                state = STATE_IDLE;
                                total_written = 0;
                                write_pos = 0;
                                read_pos = 0;
                                break;
                            }
                        }

                        uint32_t available_data = (write_pos >= read_pos) 
                            ? (write_pos - read_pos) 
                            : (SUM_BUFFER_SIZE - read_pos + write_pos);
                        if (available_data >= SUM_BUFFER_SIZE / 2) {
                            send_data(sockfd, data_type, available_data, &text_buffer[read_pos]);
                            read_pos = (read_pos + available_data) % SUM_BUFFER_SIZE;
                            LOG(LOG_LVL_INFO, "Sent %u bytes, new read_pos=%u\n", available_data, read_pos);
                        }
                        break;
                }
            }

            if (sockfd < 0) {
                LOG(LOG_LVL_ERROR, "Socket disconnect, retrying...\n");
                OS_MsDelay(2000);
            }
        }
        LOG(LOG_LVL_ERROR, "WiFi disconnected, waiting...\n");
        OS_MsDelay(2000);
    }
}
```

**流程简单说明**

**状态机引入**

- **STATE_IDLE**：等待 [0xAA][0x01][0xFF]，收到后切换到 STATE_TYPE。
- **STATE_TYPE**：接收类型引导包，设置 data_type 并切换到 STATE_PRINT 或 STATE_AUDIO。
- **STATE_PRINT**：接收总长度和打印数据。
- **STATE_AUDIO**：直接接收音频数据。

**引导包处理**

- 在 receive_data 中，检测长度为 0 的包：

```c
if (data_length == 0 && (data_type == 0x11 || data_type == 0x12)) {
    state = (data_type == 0x11) ? STATE_PRINT : STATE_AUDIO;
}
```

**打印模式**

- 保留 receive_sum_size 获取总长度。
- 接收数据直到 total_written >= data_sum_size，然后发送剩余数据。
- 检测 [0xAA][0x02][0xFF] 结束。

**音频模式**

- 跳过总长度，直接接收数据。
- 当缓冲区超过一半（15000 字节）时发送数据。
- 检测 [0xAA][0x02][0xFF] 结束并发送剩余数据。

**结束处理**

- 两种模式下，收到 [0xAA][0x02][0xFF] 后重置状态和缓冲区。

## 实现四

**打印逻辑与音频逻辑**

音频业务
空放与欠载的问题(数据流控)
空放：音频缓冲区中的数据被播放设备消费得过快，而新的音频数据未能及时填充，导致播放设备没有足够的数据进行播放，进而可能会出现**卡顿、静音或丢帧**的现象。
欠载：音频缓冲区被填充的数据过多，超出了其存储容量，导致新写入的数据无法存储，可能会导致数据丢失或者覆盖已有的数据，影响音频质量。

任务间关系分析
Socket 任务：接收网络数据，写入 text_buffer，需要更新 write_pos 更改缓冲区内容。
audio 任务：读取 text_buffer，播放到喇叭，需要更新 read_pos 读取缓冲区内容。
共享资源：text_buffer(暂定 30K)

```c
void audio_task(void *param) {
    pwm_init(40000, 0, PWM_CHA_0, GPIO_B, GPIO_PIN_5);
    pwm_init(40000, 0, PWM_CHA_1, GPIO_B, GPIO_PIN_6);
    pwm_start(PWM_CHA_0);
    pwm_start(PWM_CHA_1);

    bool speaker_on = false;
    int16_t samples[192]; // ???? 192 ???,? receive_data ??
    const uint32_t samples_per_batch = 192;

    while (1) {
        LOG(LOG_LVL_INFO, "Audio task sem, count=%lu\n", uxSemaphoreGetCount(rx_data_ready_sem));
        if (xSemaphoreTake(rx_data_ready_sem, pdMS_TO_TICKS(10)) == pdTRUE) {
            taskENTER_CRITICAL();
            uint32_t available = (rx_write_pos >= rx_read_pos) ? 
                                 (rx_write_pos - rx_read_pos) : 
                                 (RING_BUFFER_SIZE - rx_read_pos + rx_write_pos);
            LOG(LOG_LVL_INFO, "Audio task: available=%u, write=%u, read=%u\n", 
                available, rx_write_pos, rx_read_pos);
            if (available >= samples_per_batch) {
                if (!speaker_on) {
                    pwm_start(PWM_CHA_0);
                    pwm_start(PWM_CHA_1);
                    speaker_on = true;
                }
                if (read_circular_buffer(PCMBuffer_rx, (uint32_t *)&rx_read_pos, 
                                        (uint32_t *)&rx_write_pos, RING_BUFFER_SIZE, 
                                        samples, samples_per_batch, sizeof(int16_t)) == 0) {
                    LOG(LOG_LVL_INFO, "Audio task read %u samples, new read=%u\n", 
                        samples_per_batch, rx_read_pos);
                    taskEXIT_CRITICAL();
                    // // ??????
                    for (uint32_t i = 0; i < samples_per_batch; i++) {
                        uint16_t pwm_value = (samples[i] + 32768) >> 4;
                        float duty = (pwm_value / 4095.0f) * 100.0f;
                        pwm_set_duty(PWM_CHA_0, duty);
                        vTaskDelay(pdMS_TO_TICKS(1000 / 16000));
                    }
                } else {
                    LOG(LOG_LVL_ERROR, "Read from PCMBuffer_rx failed\n");
                    taskEXIT_CRITICAL();
                }
            } else {
                taskEXIT_CRITICAL();
            }
        } else {
            LOG(LOG_LVL_INFO, "Audio task timeout, no data ready\n");
            if (speaker_on) {
                pwm_stop(PWM_CHA_0);
                pwm_stop(PWM_CHA_1);
                speaker_on = false;
                LOG(LOG_LVL_INFO, "Speaker turned off due to no data\n");
            }
        }
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}
```



## 测试工具

一个 python 编写的测试工具，只需要修改文件中的 IP 和端口号即可；

## 目标

鉴权：安全，mac地址，SN,app写入，flash

mac->sn，sn->mac，一个设备一个SN，socketid给我，登陆状态

UUIDD

响应0.2s

TTS LLM 语音合成，一边合成一边推送

打断功能；

同时，多个socket列表

播放数据的问题，怎么解决？

```c
// 接收数据
static int receive_data(int sockfd, uint8_t *text_buffer, uint32_t *write_pos, uint32_t *read_pos) {
    // 接收缓冲区
    static uint8_t buffer[512];
    // 接收的数据长度
    // 接收 sizeof(buffer) 长度的内容到 buffer 中
    int len = lwip_recv(sockfd, buffer, sizeof(buffer), 0);
    if (len < 0) { // 非阻塞模式下无数据会返回小于0的长度
        if (errno == EAGAIN || errno == ETIMEDOUT) {
            OS_MsDelay(100);
            return 0;
        }
        LOG(LOG_LVL_ERROR, "receive_data len < 0: %d\n", errno);
        return -1;
    } else if (len == 0) { // socket 关闭
        LOG(LOG_LVL_INFO, "Server closed connection\n");
        return -1;
    }

    int i = 0;
    while (i < len) {
        if (temp_pos < TEMP_BUFFER_SIZE) {
            temp_buffer[temp_pos] = buffer[i];
            temp_pos++;
            LOG(LOG_LVL_INFO, "uint8_t = %d\n",buffer[i]);
            int validation_result = validate_format(temp_buffer, temp_pos);

            if (validation_result == -1) {
                send_ACK(sockfd, 0xEE);
                temp_pos = 0;
                return 0;
            }

            if (validation_result == 2) {
                uint16_t data_length = (temp_buffer[2] << 8) | temp_buffer[3];
                data_type = temp_buffer[1];
                uint8_t *data = &temp_buffer[4];
                uint16_t recv_crc = (temp_buffer[4 + data_length] << 8) | temp_buffer[5 + data_length];
                uint16_t calculated_crc = calculate_crc16(data, data_length);

                if (calculated_crc != recv_crc) {
                    LOG(LOG_LVL_ERROR, "CRC16 mismatch: received=0x%04X, calculated=0x%04X\n", recv_crc, calculated_crc);
                    send_ACK(sockfd, 0xEE);
                } else {
                    if (data_length == 0 && (data_type == 0x11 || data_type == 0x12)) {
                        // 引导包
                        LOG(LOG_LVL_INFO, "recv type : 0x%02X\n", data_type);
                        send_ACK(sockfd, 0xAA);
                        // 状态修改，打印业务还是音频业务
                        state = (data_type == 0x11) ? STATE_PRINT : STATE_AUDIO;
                    } else if (write_circular_buffer(text_buffer, write_pos, read_pos, data, data_length) != 0) {
                        LOG(LOG_LVL_ERROR, "Write to buffer failed\n");
                        send_ACK(sockfd, 0xEE);
                    } else {
                        total_written += data_length;
                        send_ACK(sockfd, 0xAA);
                        LOG(LOG_LVL_INFO, "CRC match, total_written=%u\n", total_written);
                        // return 2;
                    }
                }
                temp_pos = 0;
            }
        } else {
            send_ACK(sockfd, 0xEE);
            temp_pos = 0;
            return 0;
        }
        i++;
    }
    return 0;
}
```
