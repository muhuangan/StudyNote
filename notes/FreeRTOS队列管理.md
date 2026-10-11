---
created_at: "2026-10-11"
updated_at: "2026-10-11"
tags:
    - 嵌入式
    - Robo_Master
archived: false
---

[Robo Master电控](./robomaster.md) / 4. FreeRTOS / 4.3 队列管理

# 队列管理

1. 定义: 内核管理的先进先出数据缓冲区, 任务与任务、任务与中断之间传递数据的基本手段; 信号量与互斥锁也由队列实现, 见 [资源管理](./FreeRTOS资源管理.md)
2. 创建: xQueueCreate(长度, 单项字节数) 动态创建, 长度与项大小创建后不可变, 静态版为 xQueueCreateStatic
3. 写入
    1. xQueueSend 追加到队尾, xQueueSendToFront 插入队头
    2. xQueueOverwrite 覆盖唯一元素, 只允许长度为 1 的队列, 用于只保留最新值
    3. 中断中用 xQueueSendToBackFromISR 等 FromISR 版本, 配合 pxHigherPriorityTaskWoken 切换任务, 见 [中断管理](./FreeRTOS中断管理.md)
4. 读取
    1. xQueueReceive 取出并移除队首数据
    2. xQueuePeek 只读取不移除, 适合多任务读取同一状态
    3. xQueueMessagesWaiting 查询当前可读的数据条数
5. 阻塞语义
    1. 读方: 队列为空时阻塞等待数据; 写方: 队列为满时阻塞等待空位; 超时可设, portMAX_DELAY 表示一直等
    2. 多个任务在同一方向等待时按优先级解除阻塞, 最高优先级任务先获得数据或空位
6. 数据语义: 队列按值拷贝进出, 读写双方各持一份副本; 大数据结构应传指针, 并保证指针目标在读方使用期间有效
7. 与任务通知的取舍: 队列多占一份内存, 但支持多个任务等待同一个队列; 一对一的简单通知用 [任务管理](./FreeRTOS任务管理.md) 中的任务通知更快
