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

## 实现

该实现是基于亮牛 LN882H 芯片，示例模板为 wifi_mcu_basic_example 

可以随时监听来自指定 IP 和端口的信息，并且每 5s 向对方发送一句 Hello from MCU!

```c
// usr_app.c
// RTOS 任务的入口函数
static void socket_task_entry(void *params) {
    // 声明 socket 文件描述符 sockfd，初始化为 -1，表示未创建。
    int sockfd = -1;
    // 存储服务器地址（IP 和端口）。
    struct sockaddr_in server_addr;
    // 定义连接和接收的超时时间为 5 秒（5000 毫秒）。
    int timeout_ms = 5000;
    // 定义发送消息的间隔为 5 秒。
    int send_interval_ms = 5000;
	// 循环检查 WiFi 是否获取到 IP 地址。
    while (!netdev_got_ip()) {
        LOG(LOG_LVL_INFO, "Socket task: No IP Address...\n");
        OS_MsDelay(1000);
    }

    tcpip_ip_info_t ip_info;
    // 获取 STA（客户端）模式的 IP。
    netdev_get_ip_info(NETIF_IDX_STA, &ip_info);
    // 将 IP 转换为字符串（如 "192.168.2.52"）并打印。
    LOG(LOG_LVL_INFO, "Socket task: IP assigned: %s\n", ip4addr_ntoa(&ip_info.ip));

    while (1) {
        // 检查 socket 是否未创建或已关闭。
        if (sockfd < 0) {
            // 创建 TCP socket。
            // AF_INET：IPv4 协议。
			// SOCK_STREAM：TCP 类型。
			// 0：默认协议（TCP）。SOCK_STREAM 表示是流，默认就是 TCP
            sockfd = lwip_socket(AF_INET, SOCK_STREAM, 0);
            // 检查 socket 创建是否成功。
            if (sockfd < 0) {
                LOG(LOG_LVL_ERROR, "Socket Creation Failed: %d\n", errno);
                OS_MsDelay(2000);
                continue; // 跳回循环开头。
            }
            // 创建成功时，打印 socket 文件描述符。
            LOG(LOG_LVL_INFO, "Socket created, fd=%d\n", sockfd);

            // 将 socket 设置为非阻塞模式。
            int flags = 1;
            // FIONBIO：设置非阻塞标志。flags = 1：启用非阻塞。
            if (lwip_ioctl(sockfd, FIONBIO, &flags) < 0) {
                // 失败时，关闭 socket，重试。
                LOG(LOG_LVL_ERROR, "lwip_ioctl failed: %d\n", errno);
                lwip_close(sockfd);
                sockfd = -1;
                OS_MsDelay(2000);
                continue; // 跳回循环开头重试
            }

            // 配置服务端地址。
            memset(&server_addr, 0, sizeof(server_addr));// 清空结构体。
            server_addr.sin_family = AF_INET; // IPv4。
            server_addr.sin_port = lwip_htons(8080); // 端口 8080
            server_addr.sin_addr.s_addr = inet_addr("192.168.2.217"); // 服务端 IP

            // 尝试连接服务端。
            LOG(LOG_LVL_INFO, "Attempting to connect to 192.168.2.217:8080...\n");
            // 发起 TCP 连接。
            int connect_result = lwip_connect(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr));
            // 非阻塞模式下，可能返回 -1 且 errno = EINPROGRESS（连接正在进行）。
            // 如果失败且不是 EINPROGRESS，关闭 socket 并重试。
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
            // 将 sockfd 添加到 fd_set 中，用于 select 函数监控。
            FD_SET(sockfd, &writefds);
            FD_SET(sockfd, &exceptfds);
            int select_result = lwip_select(sockfd + 1, NULL, &writefds, &exceptfds, &tv);
            // 检查 select 结果。
            if (select_result <= 0 || FD_ISSET(sockfd, &exceptfds)) {
                LOG(LOG_LVL_ERROR, "Socket Connect timed out or failed after 5s\n");
                lwip_close(sockfd);
                sockfd = -1;
                OS_MsDelay(2000);
                continue;
            }

            // 检查 socket 的错误状态。
            int so_error = 0; // 获取连接的最终状态。
            socklen_t len = sizeof(so_error);
            if (lwip_getsockopt(sockfd, SOL_SOCKET, SO_ERROR, &so_error, &len) < 0 || so_error != 0) {
                LOG(LOG_LVL_ERROR, "lwip_connect failed with error: %d\n", so_error);
                lwip_close(sockfd);
               sockfd = -1;
                OS_MsDelay(2000);
                continue;
            }
			// 连接成功时打印日志。
            LOG(LOG_LVL_INFO, "Connected to 192.168.2.217:8080\n");

            // 设置接收超时为 5 秒。
            tv.tv_sec = timeout_ms / 1000;
            tv.tv_usec = (timeout_ms % 1000) * 1000;
            // SO_RCVTIMEO：避免 lwip_recv 无限阻塞。
            if (lwip_setsockopt(sockfd, SOL_SOCKET, SO_RCVTIMEO, &tv, sizeof(tv)) < 0) {
                LOG(LOG_LVL_ERROR, "lwip_setsockopt SO_RCVTIMEO failed: %d\n", errno);
            }
        }

        // 定义发送消息和上一次发送时间。
        char *msg = "Hello from MCU!";
        uint32_t last_send_time = 0;
		// 进入数据收发循环。
        while (sockfd >= 0) {
            // 获取当前时间戳（毫秒）。
            uint32_t current_time = OS_GetTicks();
            // 检查是否到发送时间（每 5 秒）。
            if (current_time - last_send_time >= send_interval_ms) {
                // 发送消息给服务端。
                int sent = lwip_send(sockfd, msg, strlen(msg), 0);
                // 失败时关闭 socket，退出收发循环。
                if (sent < 0) {
                    LOG(LOG_LVL_ERROR, "lwip_send failed: %d\n", errno);
                    lwip_close(sockfd);
                    sockfd = -1;
                    break;
                }
                // 发送成功时打印日志，更新发送时间。
                LOG(LOG_LVL_INFO, "Sent: %s (%d bytes)\n", msg, sent);
                last_send_time = current_time;
            }

            // 接收服务端数据。
            char buffer[128];
            int len = lwip_recv(sockfd, buffer, sizeof(buffer) - 1, 0);
            // 处理接收到的数据。
            if (len > 0) {
                buffer[len] = '\0';
                LOG(LOG_LVL_INFO, "Received: %s\n", buffer);
            // 服务端关闭连接。    
            } else if (len == 0) {
                LOG(LOG_LVL_INFO, "Server closed connection\n");
                lwip_close(sockfd);
                sockfd = -1;
                break;
            // 接收超时或无数据。    
            } else if (errno == EAGAIN || errno == ETIMEDOUT) {
                // EAGAIN/ETIMEDOUT：非阻塞模式下正常，延时 100ms。
                OS_MsDelay(100);
            } else {
                // 其他接收错误。关闭 socket，退出循环。
                LOG(LOG_LVL_ERROR, "lwip_recv failed: %d\n", errno);
                lwip_close(sockfd);
                sockfd = -1;
                break;
            }
        }

        // 连接断开后延时 2 秒。避免频繁重试占用资源。
        if (sockfd < 0) {
            OS_MsDelay(2000);
        }
    }
}

// 在 usr_app_task_entry 中添加这个任务
if (OS_OK != OS_ThreadCreate(&g_socket_thread, "SocketTask", socket_task_entry, NULL, OS_PRIORITY_BELOW_NORMAL, USR_APP_TASK_STACK_SIZE)) {
        LN_ASSERT(1);
}
```

改进：

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

后面需要改进的地方：

1. 账密需要存储在 flash 中不需要用户二次输入
2. 如果账密不对， 他会不停的触发扫描然后读取密码连接，这是需要从 flash 中读取，并且返回账密错误的信息
3. 需要读取 flash 中传递过来的数据，并将其发给客户端
4. 接收服务端传递过来的数据并存入 flash 中

## 测试工具

一个 python 编写的测试工具，只需要修改文件中的 IP 和端口号即可；
