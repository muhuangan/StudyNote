---
created_at: "2026-10-11"
updated_at: "2026-10-11"
tags:
    - 嵌入式
    - Robo_Master
archived: false
---

[Robo Master电控](./robomaster.md) / 4. FreeRTOS / 4.5 中断管理

# 中断管理

1. 基本分工: 中断响应快于任何任务, 中断服务函数只做最短处理, 耗时工作交给任务; 需要通知任务时经 FromISR API
2. FromISR 规则
    1. 任务级 API 都有带 FromISR 后缀的中断版本, 内部不阻塞, 且在中断退出前触发调度
    2. 中断中不得调用普通版本 API
    3. 带 pxHigherPriorityTaskWoken 参数的版本, 在中断末尾用 portYIELD_FROM_ISR 传入该参数, 决定退出中断后是否立即切换任务
3. 中断优先级配置(Cortex-M)
    1. FreeRTOS 只允许在优先级数值大于等于 configMAX_SYSCALL_INTERRUPT_PRIORITY 的中断中调用 FromISR API; 优先级更高(数值更小)的中断不得触碰内核接口, 只能直接读写外设寄存器
    2. configPRIO_BITS 必须与芯片的 __NVIC_PRIO_BITS 一致, 否则屏蔽阈值算错, 轻则内核异常重则跑飞
4. 临界区: taskENTER_CRITICAL 与 taskEXIT_CRITICAL 进出临界区, 屏蔽阈值以下的全部中断, 只能保护极短的几条语句, 否则中断响应超时
5. 中断嵌套: 阈值以上的中断不被临界区屏蔽, 可嵌套抢占, 电机故障保护等最高优先级中断应配置在该区间
6. 应用: CAN 接收回调中用 xQueueSendToBackFromISR 把帧存入 [队列管理](./FreeRTOS队列管理.md) 中的队列, 解析与控制放在高优先级任务里完成
