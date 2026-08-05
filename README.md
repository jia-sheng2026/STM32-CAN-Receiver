# STM32 CAN 接收端程序

> STM32F103C8T6 + SN65HVD230 CAN 收发器 | 500 kbps | IDLIST 过滤器

---

## 📋 项目概述

STM32F103C8T6 作为 CAN 接收节点，通过 SN65HVD230 收发器接入 500kbps CAN 总线。
接收标准帧 ID `0x100`（8字节数据），通过 USART1 串口打印帧 ID、DLC 和数据内容，
并翻转板载 LED 作为接收指示。

---

## 🔌 硬件连接

### CAN 模块 (SN65HVD230)

| CAN 模块引脚 | → | STM32F103C8T6 引脚 |
| :--- | :--- | :--- |
| VCC (3.3V) | → | 3.3V |
| GND | → | GND |
| RXD | → | PA11 (CAN_RX) |
| TXD | → | PA12 (CAN_TX) |

### 调试串口 (CH340)

| CH340 引脚 | → | STM32F103C8T6 引脚 |
| :--- | :--- | :--- |
| TX | → | PA9 (USART1_RX) |
| RX | → | PA10 (USART1_TX) |
| GND | → | GND |

### CAN 总线终端电阻
- CAN_H 和 CAN_L 之间 **两端各并联一个 120Ω 电阻**
- 总线总电阻 ≈ **60Ω**

---

## ⚙️ 关键配置参数

### 时钟系统
HSE = 8MHz → PLL ×9 → SYSCLK = 72MHz
AHB = 72MHz, APB1 = 36MHz (CAN 时钟源), APB2 = 72MHz


### CAN 波特率计算（面试高频题）
波特率 = PCLK1 / (Prescaler × (SyncSeg + BS1 + BS2))
= 36MHz / (9 × (1 + 6 + 1))
= 36MHz / 72
= 500 kbps ✅


**CubeMX 对应配置：**
- Prescaler = 9
- TimeQuanta in BS1 = 6
- TimeQuanta in BS2 = 1
- Resynchronization Jump Width = 1

### 过滤器配置（IDLIST 模式）

只接收标准 ID `0x100`，拒绝其他所有 ID。

```c
CAN_FilterTypeDef sFilterConfig;
sFilterConfig.FilterBank = 0;
sFilterConfig.FilterMode = CAN_FILTERMODE_IDLIST;
sFilterConfig.FilterScale = CAN_FILTERSCALE_16BIT;
sFilterConfig.FilterIdHigh = (0x100 << 5) & 0xFFFF;  // = 0x2000
sFilterConfig.FilterMaskIdHigh = 0xFFFF;             // 全匹配
sFilterConfig.FilterFIFOAssignment = CAN_RX_FIFO0;
sFilterConfig.FilterActivation = ENABLE;
HAL_CAN_ConfigFilter(&hcan, &sFilterConfig);

串口调试
USART1, 115200-8-N-1（可通过宏切换至 9600）

🐛 调试记录 & 问题解决
问题 1：串口输出乱码
项目	内容
现象	SSCOM 收到 CAN Receiver Ready 显示为乱码
原因	外部晶振（HSE）未起振，导致系统时钟偏差
解决	检查晶振焊接，或切换到 HSI 内部时钟源（见 SystemClock_Config）
问题 2：ST-LINK 无法连接（No device found）
项目	内容
现象	烧录程序后，ST-LINK 无法识别芯片
原因	PA13/PA14（SWD）被误配置为普通 GPIO
解决	BOOT0 拉高 + 串口 ISP 擦除 Flash
预防	在 CubeMX 中保持 PA13/PA14 为 Serial Wire 模式

📁 工程结构
CAN_Receiver/
├── Core/
│   ├── Inc/               # 头文件
│   │   ├── main.h
│   │   ├── can.h
│   │   └── usart.h
│   └── Src/               # 源文件
│       ├── main.c
│       ├── can.c
│       └── usart.c
├── Drivers/               # HAL 库驱动
│   ├── CMSIS/
│   └── STM32F1xx_HAL_Driver/
└── Debug/                 # 编译输出（已忽略）

🚀 后续优化方向
迁移至 FreeRTOS：CAN 接收任务 + 串口打印任务，使用队列传递数据

增加 CAN 错误处理：Bus Off、Error Passive 等状态监测

支持 CANopen 协议：PDO/SDO 通信

📄 许可证
MIT License

👤 作者
GitHub: jia-sheng2026

