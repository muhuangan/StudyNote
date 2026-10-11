---
created_at: "2026-10-11"
updated_at: "2026-10-11"
tags:
    - 嵌入式
    - Robo_Master
archived: false
---

[Robo Master电控](./robomaster.md) / 4. FreeRTOS / 4.4 软件定时器

# 软件定时器

1. 定义: 由内核维护的定时器对象, 到期后执行一次回调, 数量不受硬件定时器通道限制
2. 守护任务: 定时器命令队列与全部回调都由守护任务(软件定时器任务)处理, 回调运行在任务上下文, 精度为一个 tick
3. 创建: xTimerCreate(名称, 周期, 重载模式, ID, 回调函数), 周期以 tick 为单位, 用 pdMS_TO_TICKS 换算毫秒
4. 重载模式: pdTRUE 到期后自动重载周期连续触发, pdFALSE 单次触发
5. 控制命令: xTimerStart、xTimerStop、xTimerReset、xTimerChangePeriod 本质是向守护任务投递命令; 中断中用 xTimerStartFromISR 等对应版本
6. 使用要点
    1. 回调在守护任务中执行, 回调耗时会推迟其他定时器的到期处理, 内容应保持简短
    2. 定时器任务优先级由 configTIMER_TASK_PRIORITY 决定, 一般设为较低值, 避免抢占电机控制等高优先级任务
    3. 命令经 configTIMER_QUEUE_LENGTH 长度的队列传递, 短时间大量投递会溢出, xTimerCreate 返回 NULL 即创建失败
    4. xEventGroupSetBitsFromISR 等延迟型 FromISR API 也经守护任务执行, 使用它们必须启用软件定时器
7. 应用: 周期上报传感器数据、通信超时检测、指示灯闪烁等不宜用 vTaskDelay 硬凑的定时需求
