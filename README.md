# STM32 CAN Receiver

基于 STM32F103C8T6 的 CAN 总线接收端程序，接收 500kbps 波特率的 CAN 数据帧并通过串口打印解析结果。

## 功能特性

- CAN 总线 500kbps 通信
- 中断方式接收 CAN 数据帧
- 串口（USART1）打印帧 ID、DLC 及 8 字节数据
- 板载 LED 翻转指示（收到数据时触发）
- 支持标准 ID 过滤器（IDLIST 模式）

## 硬件平台

| 组件 | 型号 |
| :--- | :--- |
| MCU | STM32F103C8T6 |
| CAN 收发器 | SN65HVD230 |
| 调试串口 | CH340 (USB 转 TTL) |
| CAN 分析仪 | CANable |

## 引脚连接

| 功能 | STM32 引脚 | 外设引脚 |
| :--- | :--- | :--- |
| CAN_RX | PA11 | SN65HVD230 RXD |
| CAN_TX | PA12 | SN65HVD230 TXD |
| USART1_TX | PA9 | CH340 RX |
| USART1_RX | PA10 | CH340 TX |
| LED | PC13 | 板载 LED |

## CAN 配置参数

| 参数 | 值 |
| :--- | :--- |
| 波特率 | 500 kbps |
| 工作模式 | Normal 模式 |
| 过滤器模式 | IDLIST（仅接收 ID 0x100） |
| 时钟源 | HSE 外部晶振 (8MHz → 72MHz PLL) |

## 使用说明

1. 给 STM32 板供电（USB 或 3.3V）
2. 连接 CAN 总线（CAN_H ↔ CAN_H，CAN_L ↔ CAN_L）
3. 两端各接一个 120Ω 终端电阻
4. 串口连接：PA9 → CH340 RX，PA10 → CH340 TX
5. 打开串口助手，波特率 9600
6. 发送端发送 ID 为 0x100 的 CAN 帧，接收端打印：
  ID:0x100 Data:11 22 33 44 55 66 77 88

## 测试验证

使用 CANable + SavvyCAN 模拟发送节点，发送 ID 0x100 的数据帧，接收端串口打印结果与发送数据一致。

## 工程结构
CAN_Receiver/
  ├── Core/
  │ ├── Inc/ # 头文件
  │ └── Src/ # 源文件（main.c, can.c, usart.c 等）
  ├── Drivers/ # HAL 库驱动
  └── Debug/ # 编译输出
  
## 作者

- **GitHub**: [jia-sheng2026](https://github.com/jia-sheng2026)

## 许可

MIT License
