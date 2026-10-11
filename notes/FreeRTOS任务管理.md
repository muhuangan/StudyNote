---
created_at: "2026-10-11"
updated_at: "2026-10-11"
tags:
    - 嵌入式
    - Robo_Master
archived: false
---

[Robo Master电控](./robomaster.md) / 4. FreeRTOS / 4.2 任务管理

# 任务管理

1. 任务: 一个无限循环函数加一块独立栈, 经 xTaskCreate(函数指针, 名称, 栈深度, 参数, 优先级, 句柄) 创建, 静态版为 xTaskCreateStatic
2. 任务状态
    1. 就绪: 具备运行条件, 等待被调度
    2. 运行: 占用 CPU, 单核下同一时刻只有一个任务处于该状态
    3. 阻塞: 等待的事件未到或延时尚未结束, 不占 CPU
    4. 挂起: 由 vTaskSuspend 挂起, 只能由 vTaskResume 恢复, 没有事件能唤醒它
3. 调度规则
    1. 抢占式: 永远运行最高优先级的就绪任务, 更高优先级任务就绪时立即抢占当前任务
    2. 同优先级: 按时间片轮转, 由 configUSE_TIME_SLICING 控制开关
    3. 优先级: 数值越大越高, 0 为最低, 上限由 configMAX_PRIORITIES 决定
4. 周期控制
    1. vTaskDelay: 相对延时, 挂起指定时间后回到就绪
    2. vTaskDelayUntil: 唤醒时刻按绝对时间推算, 自动扣除任务自身的执行耗时, 用于固定周期任务
    3. 高优先级任务必须适时阻塞或延时, 否则低优先级任务连同回收已删任务内存的空闲任务一起饿死
5. 删除: vTaskDelete 删除指定任务或任务自身, 被删任务的栈与控制块由空闲任务回收, 不能长期使空闲任务得不到运行
6. 任务通知: 每个任务自带一个通知值, xTaskNotify 发送, ulTaskNotifyTake 接收, 相比队列与二值信号量更快且不占额外内存, 适合一对一通知; 每个任务只有一个通知槽, 多个来源同时使用会互相干扰
7. 句柄: 创建时末参数回传任务句柄, vTaskSuspend、vTaskDelete、vTaskPrioritySet 等按句柄操作任务
