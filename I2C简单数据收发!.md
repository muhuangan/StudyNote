---
created_at: 2026-06-22
updated_at: 2026-09-18
tags:
    - 嵌入式
    - STM32
archived: false
---

[STM32](./STM32.md) / 4. I2C

# I2C简单数据收发!

目标: 点亮OLED屏幕, 同时通过读取OLED屏幕返回的数据判断OLED屏幕是否被点亮, 如果被点亮则点亮板载LED

本次实验使用一块以ssd1306为驱动芯片的0.96英寸oled屏幕

1. 创建工程, 将debug模式调整为serial
2. 将pc13引脚的模式调整为gpio output, 以开漏模式进行输出, 初始电平为高电平
3. 打开I2C1, 将i2c模式调整为高速模式,
4. 写入代码

```C
uint8_t commands[] = {0x00, 0x8d, 0x14, 0xaf, 0xa5};

HAL_I2C_Master_Transmit(&hi2c1, 0x78, commands, sizeof(commands) / sizeof(commands[0]), HAL_MAX_DELAY);

uint8_t dataRcvd;
HAL_I2C_Master_Receive(&hi2c1, 0x78, &dataRcvd, 1, HAL_MAX_DELAY);

if ((dataRcvd & (0x01 << 6)) == 0) {
    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
}
else {
    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
}
```

5. 如果板载led点亮, 且oled屏幕显示为白色则实验成功

## 相关笔记

- 上一篇:[基础知识](./I2C基础知识.md)  
- 下一篇:[基础知识](./时钟系统基础知识.md)  
- 返回索引:[STM32](./STM32.md)  
