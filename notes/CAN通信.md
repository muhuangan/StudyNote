---
created_at: "2026-10-03"
updated_at: "2026-10-03"
tags:
    - 嵌入式
    - Robo_Master
archived: false
---

[Robo Master电控](./robomaster.md) / 2. CAN通信 / 2.1 CAN通信

# CAN通信

1. 定义: CAN是Controller Area Network的缩写
2. 分层
    1. [物理信号层](./CAN通信的物理信号层.md): 规定硬件设计规则, 用户使用的接线方式, 线路上传输信号与电平格式对应关系, 波特率与比特率, 数据同步异步传输, 收发模式等
    2. [链路传输层](./CAN通信的链路传输层.md): 规定了CAN ID, 冲突检测与避免(仲裁), 误码校验, 滤波器与掩码, 该层的帧格式
    3. 应用数据层: 被传输的数据本体
