---
created_at: "2026-10-11"
updated_at: "2026-10-11"
tags:
    - 嵌入式
    - Robo_Master
archived: false
---

[Robo Master电控](./robomaster.md) / 4. FreeRTOS / 4.1 内存管理

# 内存管理

1. 管理对象: 任务栈、任务控制块、队列、软件定时器等内核对象的存储, 动态创建时从堆分配, 静态创建时由调用者提供内存
2. 两种创建方式
    1. 动态: xTaskCreate、xQueueCreate 等, 内部经 pvPortMalloc 从堆分配, 删除时经 vPortFree 释放
    2. 静态: xTaskCreateStatic、xQueueCreateStatic 等, 参数传入预分配的结构体或数组, 不占用堆
3. 堆实现方案: 编译期在 heap_1~heap_5 中选择一种
    1. heap_1: 只能分配不能释放
    2. heap_2: 可释放但不合并相邻空闲块, 适合固定大小对象反复创建与删除
    3. heap_3: 在 heap_2 基础上加临界区保护, 底层改用 C 库的 malloc 与 free, 线程安全
    4. heap_4: 分配释放并合并相邻空闲块, 碎片最少, 动态创建场景最常用
    5. heap_5: 与 heap_4 相同, 但允许内存来自不连续的多段区域, 如内部 SRAM 加外部 RAM
4. 碎片问题: 频繁创建删除不同大小的对象会把空闲块切碎, 分配大对象时失败; 长期不变的对象尽量静态创建
5. 任务栈
    1. 每个任务独占一块栈, 深度按局部变量、调用链深度与中断嵌套情况估算并留裕量
    2. uxTaskGetStackHighWaterMark 查询任务历史最小剩余栈量, 据此裁剪深度
    3. 栈溢出会破坏相邻内存且现象随机, 开启 configCHECK_FOR_STACK_OVERFLOW 钩子提前捕获
6. 配置与取舍: configSUPPORT_STATIC_ALLOCATION 与 configSUPPORT_DYNAMIC_ALLOCATION 决定可用的创建方式; RM 电控常用 heap_4 加动态创建, 电机控制等关键对象可静态化, 保证分配必然成功
