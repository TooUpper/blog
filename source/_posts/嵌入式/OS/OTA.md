---
title: "OTA"
date: 2024-12-18 10:24:00
categories: "OS"
tags: 
- "嵌入式系统"
---

OTA（Over-The-Air）即“空中升级”，指通过无线通信（如蓝牙、Wi-Fi、蜂窝网络等）向设备发送数据包进行远程更新的一种技术。它可以用于更新设备的固件、软件应用程序或配置数据，无需通过物理接口（如 USB、JTAG）连接设备。这种技术广泛用于 IoT 设备、嵌入式系统、智能设备（如手表、家电）、汽车电子等领域。

## OTA 的核心概念

1. **固件更新（Firmware Update）**
   OTA 中最常见的用途是更新 MCU 的固件。固件是嵌入式设备运行的核心程序。通过 OTA 更新固件，可以修复漏洞、添加新功能或优化性能。
2. **增量更新（Delta Update）**
   增量更新指只发送固件中修改的部分数据，而不是整个固件，从而减少传输数据量，提高更新效率。
3. **数据传输协议**
   OTA 通常依赖蓝牙（如 BLE）、Wi-Fi（MQTT/HTTP）、LoRa 或蜂窝网络。蓝牙设备常用 GATT 服务进行数据传输，而 Wi-Fi 设备则常用 HTTP 或自定义协议。
4. **可靠性保证**
   为了确保 OTA 更新的可靠性，通常需要以下机制：
   - **校验机制**：通过校验和（Checksum）、CRC 或数字签名验证数据完整性和来源可信性。
   - **回滚机制**：更新失败后，系统应能够恢复到之前的稳定版本。
   - **双分区机制**：设备固件分为两个分区（Active/Backup），更新时在备用分区写入新固件，测试成功后切换分区。

## OTA 升级的存储需求

为了实现 OTA 升级及回滚，典型的存储需求如下：

1. **当前固件**：即设备当前正在运行的固件。
2. **新固件**：从上位机通过 OTA 下载的新固件。
3. **BootLoader**：负责固件加载、升级验证以及回滚逻辑。
4. **元数据区域**：OTA 升级标志位、固件版本信息、存储升级状态、校验信息（如 CRC32）等。

如果 MCU 内部 Flash 空间不足，可以通过以下两种方式解决：

- **压缩升级固件**（如使用差分升级）。
- **使用外部 Flash**（SPI Flash、NOR Flash 等）。

## OTA 的实现流程

1.**新固件存储到外部 Flash**

- 在 OTA 数据接收阶段，通过蓝牙、Wi-Fi 等无线协议接收新固件，将其逐包写入外部 Flash 的指定存储区。
- 写入完成后，可以通过校验（如 CRC32、MD5）验证固件数据的完整性。
- 如果校验失败，可以删除这部分数据或通知上位机重新上传。
- 当新的固件下载完成后设置 OAT_Flag = 1，表示固件准备好了。

2.**等待触发升级**

- **触发方式**：升级可以通过多种方式触发，例如：
  - 按键触发：当用户按下特定按键（例如 UI 上的“升级”按钮或硬件按键）时，MCU 检查 OAT_Flag。如果标志位为 1，则进入 BootLoader 执行固件升级。
  - 定时器触发：可以设置定时器，在设备空闲时自动检查 OAT_Flag。如果标志位为 1，则自动进入 BootLoader 执行升级流程。
  - 外部指令触发：可以通过外部设备（如手机 App 或云端）发送指令，MCU 在接收到指令后检查 OAT_Flag，如果标志位为 1，则进入 BootLoader 执行固件升级。
- 升级时机：
  - 选择在设备空闲时进行，避免正在运行的任务中断。
  - 如果设备有低功耗模式，确保升级期间设备有足够的电量支持完成升级流程。

3.**固件升级（BootLoader 阶段）**

- MCU 的 BootLoader 程序负责从外部 Flash 中读取新固件，并将其烧录到内部 Flash。
- 升级步骤如下：
  1.  BootLoader 检测到 OAT_Flag 标志位为 1。
  2. 将外部 Flash 中存储的新固件逐页（Page）拷贝到 MCU 的内部 Flash。
  3. 拷贝完成后，通过固件校验机制（如再次校验 CRC32）验证固件是否完整无误。
  4. 如果验证通过，设置启动标志位，指向新固件所在地址，将 OAT_Flag 标志位为 0。
  5. 重启 MCU，运行新固件。

4.**回滚机制**

- 如果新固件启动失败或校验失败：
  1. BootLoader 将回滚到旧固件（通常存储在 Flash 的固定分区或另一个镜像中）。
  2. 设备恢复正常运行，避免完全不可用的情况。

> **断点续传**
>
> - MCU 需要跟踪已下载和写入的分片偏移地址，记录进度到非易失性存储（如内部 Flash 或 EEPROM）。
> - 在断电或升级中断后，MCU 可以从上次中断的位置继续下载，而无需重新开始。
>
> CRC32 校验
>
> - **CRC（Cyclic Redundancy Check，循环冗余校验）** 是一种常用的数据校验算法，用于检测数据在传输或存储过程中是否发生错误。
>
> - CRC32
>
>    是一种常见的 CRC 校验方法，产生一个 32 位的校验值（校验码），用于验证数据的完整性。
>
>   - 输入：一段数据（比如 256 字节、4 KB 等）。
>   - 输出：一个 32 位校验值（通常表示为十六进制数）。
>
> 校验的核心思想是通过算法将一段数据压缩为一个固定长度的校验值。如果两次计算的校验值相同，则认为数据完整无误；否则说明数据有损坏。

## 架构设计 

### Flash 划分

**内部 Flash**

| 区域           | 地址范围                | 描述                                       |
| -------------- | ----------------------- | ------------------------------------------ |
| **BootLoader** | 固定区域，0x0000~0x3FFF | 系统启动区，用于引导新旧固件运行和升级管理 |
| **运行固件区** | 0x4000~0xBFFF           | 当前正在运行的固件                         |

**外部 Flash**

| 区域           | 地址范围        | 描述                                     |
| -------------- | --------------- | ---------------------------------------- |
| **运行固件区** | 0x4000~0xBFFF   | 当前正在运行的固件（旧固件）             |
| **备份固件区** | 0xC000~0x13FFF  | 用于存储新固件，升级后切换到此区域       |
| **配置区**     | 0x14000~0x14FFF | 存储固件状态、版本信息、升级标志等元数据 |

### 存储操作

**擦写外部 Flash**

- 蓝牙模块通过 UART 向 MCU 发送数据。
- MCU 将数据包解析后，写入外部 Flash 的固件存储区。
- 每写完一部分数据（比如一页），立即读取并校验，确保写入正确。
- 在写入完成后，保存校验信息（如 CRC32）到 Flash 的校验区。

**擦写内部 Flash**

- 将 BootLoader 的代码烧写到 BootLoader 区中
- 将运行代码烧录到运行固件区中

- 注意要将 BootLoader 放在前面，保证上电第一时间执行的是 BootLoader 中的代码

### BootLoader 流程

BootLoader 的主要工作是：

- 判断是否需要进行升级：
  - 通过读取外部 Flash 的升级标志位，确认是否存在新的固件需要烧录。
- 如果需要升级：
  - 读取外部 Flash 的固件校验信息，验证其完整性。
  - 将新固件逐页写入 MCU 内部 Flash 的应用区。
  - 写入完成后，修改升级标志位 OTA_Flag。
- 如果不需要升级：
  - 设置栈指针（SP），指向运行区的地址
  - 设置程序计数器 PC（Reset_Handler） 指向运行区中程序的地址
  - 复位 B 区中使用的一些外设
  - 跳转到 A 区开始执行
- 如果升级失败：
  - 清除新固件的升级标志位。
  - 回滚到旧固件，确保设备稳定运行。

> 在 Cortex-M 微控制器中，**程序起始位置存储的内容是栈指针（SP）初始值，而不是程序的入口地址**。程序入口地址存储在 **起始位置 + 4 的地址**中。这是 ARM Cortex-M 系列的芯片设计规定的一部分。

## 示例代码

```c
// BootLoader.h
#ifndef __BOOTLOADER_H_
#define __BOOTLOADER_H_

/* bootloader版本号 */
#define BooTLoader_VERSION  "0.1"

/* 设备序号 */
#define DEVICE_NO        1      //根据设备名进行更改，定死写在bootloader中


/*=====用户配置(根据自己的分区进行配置)=====*/
#define MCU_FLASH 			0x80000U 						   // MCU 的 Flash 大小
#define MCU_FALSH_ADDR		0x08000000U				   		   // MCU 的 Flash 首地址

#define BootLoader_SIZE		0x20000U						   // BootLoader 的大小 128K
#define Application_SIZE	(MCU_FLASH - BootLoader_SIZE) 	   // 应用程序的大小 384K

#define Application_1_Addr	(MCU_FALSH_ADDR + BootLoader_SIZE) // 运行固件区的首地址
#define BACKUP_ADDR =  // 备份固件区的首地址，我们的备份固件存放在外部 Flash 中

/*==========================================*/

/* 启动的步骤 */
#define Startup_Normal 0xFFFFFFFF	// 正常启动
#define Startup_Update 0xAAAAAAAA	// 升级再启动
#define Startup_Reset  0x5555AAAA	// ***恢复出厂 目前没使用***

void Start_BootLoader(void);

#endif // !__BOOTLER_H_
===========================================
// BootLoader.c
#include "bootloader.h"
#include "stdio.h"
#include "main.h"
#include "stdlib.h"
#include "string.h"
#include "usart.h"

/**
 * @brief 串口重定向（需要开启Use MicroLIB）
 * @param ch  		发送的数据	
 * @param f         文件流
 * @return ch
 */
int fputc(int ch, FILE *f) {
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch,1, 0xFFFF);
    return ch;
}

/**
 * @brief 获取地址所在的 Flash 扇区
 * @param address 起始地址
 * @return 返回地址所在的 Flash 扇区编号，若地址无效则返回一个错误标志
 */
static uint32_t Get_Sector(uint32_t address) {
    // 检查地址是否在有效的 Flash 区间内
    if (address < FLASH_BASE || address > FLASH_END) {
        return INVALID_SECTOR;  // 返回无效扇区标志
    }

    // Bank 1 扇区划分
    if (address >= 0x0802C000 && address < 0x08030000) return 11;
    if (address >= 0x08028000 && address < 0x0802C000) return 10;
    if (address >= 0x08024000 && address < 0x08028000) return 9;
    if (address >= 0x08020000 && address < 0x08024000) return 8;
    if (address >= 0x0801C000 && address < 0x08020000) return 7;
    if (address >= 0x08018000 && address < 0x0801C000) return 6;
    if (address >= 0x08014000 && address < 0x08018000) return 5;
    if (address >= 0x08010000 && address < 0x08014000) return 4;
    if (address >= 0x0800C000 && address < 0x08010000) return 3;
    if (address >= 0x08008000 && address < 0x0800C000) return 2;
    if (address >= 0x08004000 && address < 0x08008000) return 1;
    if (address >= 0x08000000 && address < 0x08004000) return 0;

    // Bank 2 扇区划分
    if (address >= 0x08060000 && address < 0x08064000) return 26;
    if (address >= 0x08058000 && address < 0x0805C000) return 25;
    if (address >= 0x08054000 && address < 0x08058000) return 24;
    if (address >= 0x08050000 && address < 0x08054000) return 23;
    if (address >= 0x08048000 && address < 0x0804C000) return 22;
    if (address >= 0x08044000 && address < 0x08048000) return 21;
    if (address >= 0x08040000 && address < 0x08044000) return 20;
    if (address >= 0x0803C000 && address < 0x08040000) return 19;
    if (address >= 0x08038000 && address < 0x0803C000) return 18;
    if (address >= 0x08034000 && address < 0x08038000) return 17;
    if (address >= 0x08030000 && address < 0x08034000) return 16;

    return INVALID_SECTOR;  // 返回无效扇区
}

/**
 * @brief Flash擦除指定范围的扇区
 * @param start_addr 起始地址
 * @param end_addr   结束地址
 * @return 0 成功, 非0 失败，返回错误代码
 */
static int Flash_Erase_Sector(uint32_t start_addr, uint32_t end_addr) {
    uint32_t UserStartSector;
    uint32_t UserEndSector;
    uint32_t SectorError = 0;
    FLASH_EraseInitTypeDef FlashSet;

    // 解锁flash
    HAL_FLASH_Unlock();

    // 获取起始地址和结束地址的扇区号
    UserStartSector = Get_Sector(start_addr);
    UserEndSector = Get_Sector(end_addr);

    // 确保起始扇区不大于结束扇区
    if (UserStartSector > UserEndSector) {
        HAL_FLASH_Lock();
        return -1; // 错误：起始扇区大于结束扇区
    }

    // 配置擦除参数
    FlashSet.TypeErase = TYPEERASE_SECTORS;
    FlashSet.Sector = UserStartSector;
    FlashSet.NbSectors = UserEndSector - UserStartSector + 1;
    FlashSet.VoltageRange = VOLTAGE_RANGE_3;

    // 调用擦除函数
    if (HAL_FLASHEx_Erase(&FlashSet, &SectorError) != HAL_OK) {
        // 错误处理
        HAL_FLASH_Lock();
        return -2; // 错误：擦除操作失败
    }

    // 锁定flash
    HAL_FLASH_Lock();
    return 0; // 成功
}


/**
 * @brief Flash写若干个数据（word）
 * @param addr       写入的地址
 * @param buf        写入数据的起始地址
 * @param word_size  数据的单词数量
 * @return 0 成功，其他值表示失败
 */
static int Flash_Write(uint32_t addr, uint32_t *buf, uint32_t word_size) {
    // 解锁 flash
    if (HAL_FLASH_Unlock() != HAL_OK) {
        return -1; // 解锁失败
    }

    // 写入数据
    for (int i = 0; i < word_size; i++) {
        if (HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD, addr + 4 * i, buf[i]) != HAL_OK) {
            HAL_FLASH_Lock(); // 写入失败后锁定 Flash
            return -2; // 写入失败
        }
    }

    // 锁定 flash
    HAL_FLASH_Lock();

    return 0; // 成功
}

/**
 * @brief flash读若干个数据(word)
 * @param addr       读数据的地址
 * @param buf        读出数据的数组指针
 * @param word_size  长度
 * @return 
 */
static inline void Flash_Read(uint32_t addr, uint32_t *buf,uint32_t word_size) {
	memcpy(buf, (uint32_t*) addr, word_size * sizeof(uint32_t));
}

/**
 * @brief  将固件从源地址拷贝到目标地址
 * @param  src_addr   源地址
 * @param  des_addr   目标地址
 * @param  byte_size  拷贝的字节大小
 */
void Flash_CopyFirmware(uint32_t src_addr, uint32_t des_addr, uint32_t byte_size) {
    printf("> Starting firmware copy...\r\n");

    // 擦除目的地址
    printf("Erasing destination flash (0x%X - 0x%X)...\r\n", des_addr, des_addr + byte_size - 1);
	// FLASH_SUCCESS 表示擦除成功
    if (Flash_Erase_Sector(des_addr, des_addr + byte_size - 1) != FLASH_SUCCESS) {
        printf("Error: Failed to erase destination flash!\r\n");
        return;
    }
    printf("Destination flash erased successfully.\r\n");

    // 分配临时缓冲区
	// Flash 通常不可以同时进行读写操作，所以要使用一个缓冲区进行过度
	// 为了适配同一个 Flash 的操作
    uint8_t *buffer = (uint8_t *)calloc(256, sizeof(uint8_t));
    if (buffer == NULL) {
        printf("Error: Memory allocation failed!\r\n");
        return;
    }

    // 开始拷贝
    printf("Copying firmware from 0x%X to 0x%X, total size: %d bytes.\r\n", src_addr, des_addr, byte_size);
    for (uint32_t offset = 0; offset < byte_size; offset += 1024) {
        if (Flash_Read(src_addr + offset, buffer, 256) != FLASH_SUCCESS) {
            printf("Error: Flash read failed at 0x%X!\r\n", src_addr + offset);
            free(buffer);
            return;
        }

        if (Flash_Write(des_addr + offset, buffer, 256) != FLASH_SUCCESS) {
            printf("Error: Flash write failed at 0x%X!\r\n", des_addr + offset);
            free(buffer);
            return;
        }

        memset(buffer, 0, 256);
        printf("Copied %d KB...\r\n", (offset + 1024) / 1024);
    }
	
    free(buffer);
    printf("Firmware copy completed successfully.\r\n");
}

/**
 * @brief 采用汇编设置栈的值
 * @param  ulAddr	地址
 * @return NULL
 */
__asm void MSR_MSP(uint32_t ulAddr) {
    MSR MSP, r0
    BX r14
}

typedef void (*Jump_Fun)(void);
/**
 * @brief 程序跳转函数
 * @param  App_Addr 应用程序地址
 * @return NULL
 */
void JumpToApplication(uint32_t App_Addr) {
    Jump_Fun JumpToApp;
    uint32_t reset_handler;

    // 检查应用程序的栈顶地址是否合法
    if (((*(__IO uint32_t *) App_Addr) & 0x2FFE0000) == 0x20000000) {
        // 获取复位处理函数地址
        reset_handler = *(__IO uint32_t *)(App_Addr + 4);
        
        // 如果复位地址无效，打印错误信息并退出
        if (reset_handler == 0xFFFFFFFF) {
            printf("Error: Invalid Reset Handler at address: %08X\n", App_Addr + 4);
            return;
        }

        // 设置应用程序的栈顶地址
        MSR_MSP(*(__IO uint32_t *) App_Addr);

        // 禁用所有中断，清除中断标志
        __disable_irq();  // 直接使用 CMSIS 的宏来禁用中断

        // 可选：如果时钟配置需要重置，可以选择调用 HAL_RCC_DeInit()
        // 如果需要保留时钟配置，跳过这一步
        HAL_RCC_DeInit();
        SysTick->CTRL = 0;
        SysTick->LOAD = 0;
        SysTick->VAL = 0;

        // 跳转到应用程序的复位处理函数
        JumpToApp = (Jump_Fun)reset_handler;
        if (JumpToApp != NULL) {
            JumpToApp();  // 执行跳转
        } else {
            printf("Error: Invalid JumpToApp function!\n");
        }
    } else {
        printf("Error: Invalid Stack Pointer address! (Address: %08X)\n", App_Addr);
    }
}

/**
 * @brief 进行BootLoader的启动
 * @param NULL
 * @return NULL
 */
void Start_BootLoader(void) {
	/* Bootloader 信息打印 */
	printf("\r\n");
	printf("Bootloader %s %s\r\n",__DATE__,__TIME__);
	printf("Bootloader Version %s\r\n",BooTLoader_VERSION);
    printf("Device Name: D%d\r\n",DEVICE_NO);
	printf("MCU: STM32F407ZGT6 Running at 168MHz\r\n");
	printf("Bootloader Area from %#x - %#x\r\n",MCU_FALSH_ADDR,APPLICATION_ADDR - 1);
	printf("Application_1 Area from %#x - %#x\r\n",APPLICATION_ADDR,APPLICATION_ADDR + Application_SIZE -1);
    printf("\r\n");

    // 先读取外部 Flash 中的 OTA_Flag 标志位判断是否需要升级
	
	if(OTA.OTA_Flag) {
		// 更新固件
		// 修改 OTA_Flag 标志位
		
	} else {
		// 跳转到 A 区执行代码
	}
	
	/* 跳转到应用程序 */
	__disable_irq(); 	//关闭中断
	printf("=> Start Application_1......\r\n\r\n");	
	JumpToApplication(APPLICATION_ADDR);
}
```

## 注意事项

如果你的 MCU 的内部 Flash 已经可以存放完整的固件，并且足以支持所有应用功能，那么也可以考虑以下优化方案：

- 将运行固件完全存储在 MCU 的内部 Flash 中，避免对外部 Flash 的依赖。
- 外部 Flash 仅用于存储备份固件或临时数据。
- 通过 BootLoader 动态加载外部 Flash 中的新固件到内部 Flash，完成固件更新。

这种方法的优点是：

- **启动速度更快：** 固件直接从内部 Flash 加载，无需访问外存。
- **功耗更低：** 外部 Flash 的读取会增加系统功耗，尤其是在频繁启动的场景下。
- **更高的可靠性：** 外部 Flash 的稳定性可能不如内部 Flash（如受电源波动或环境温度影响）。

二、**什么时候更新外部 Flash 代码**

什么时候将外部 Flash 中**备份固件区**的代码更新到**运行固件区**：

OAT_flag = 0；表示当前不需要进行固件升级，执行的固件是最新的

运行固件区与备份固件区的代码版本不一致；不一致就执行更新，一致就说明不需要更新；
