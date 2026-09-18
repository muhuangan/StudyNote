---
created_at: 2026-06-22
updated_at: 2026-09-18
tags:
    - 嵌入式
    - STM32
archived: false
---

[STM32](./STM32.md) / 3. UART

# 基础知识

1. UART定义与接线  
   UART, 即串口, 是一种通信接口, 由两条线组成, Tx和Rx, 其中Tx(Transmit) 用于发送数据, Rx(Receive) 用于接收数据  
   ![串口接线方法](../resources/串口接线.png)  
   两个设备的Tx和Rx应交错连接
2. 串口的数据帧格式  
   ![串口数据帧格式](../resources/串口数据帧.png)
3. 校验位  
   奇校验: 要求数据中包含奇数个1  
   欧校验: 要求数据中包含偶数个1
   ![奇校验与偶校验](../resources/奇校验与偶校验.png)
4. 波特率  
   每秒钟传输位的数量  
   ![波特率](../resources/波特率.png)  
   收发双方应选择相同的波特率
5. UART vs USART  
   ![UARTvsUSART](../resources/UARTvsUSART.png)

## 相关笔记

- 上一篇:[按钮实验!](./按钮实验!.md)
- 下一篇:[使用串口简单地发送数据!](./使用串口简单地发送数据!.md)
- 返回索引:[STM32](./STM32.md)
