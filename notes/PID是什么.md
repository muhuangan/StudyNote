---
created_at: "2026-10-03"
updated_at: "2026-10-03"
tags:
    - 嵌入式
    - Robo_Master
archived: false
---

[Robo Master电控](./robomaster.md) / 3. PID控制 / 3.1 PID是什么

# PID是什么

1. 定义: PID 是按误差进行比例(P)、积分(I)、微分(D)三种运算并加权求和的反馈控制器, 不依赖被控对象的精确模型, 只凭误差就能工作
2. 闭环过程
    1. 给定值 $r(t)$ 与测量值 $y(t)$ 相减得误差 $e(t) = r(t) - y(t)$
    2. 控制器对 $e(t)$ 做三种运算并求和, 输出控制量 $u(t)$
    3. $u(t)$ 驱动执行器与被控对象, 测量值回到输入端形成闭环, 误差被不断压低
3. 连续算式:

$$u(t) = K_p e(t) + K_i \int_{0}^{t} e(\tau) \, \mathrm{d}\tau + K_d \frac{\mathrm{d}e(t)}{\mathrm{d}t}$$

4. 数字算式(按采样周期 $T$ 离散):

$$u(k) = K_p e(k) + K_i \sum_{j = 0}^{k} e(j) + K_d \left[ e(k) - e(k - 1) \right]$$

5. 位置式与增量式
    1. 位置式: 直接输出 $u(k)$, 计算依赖全部历史误差, 重启、切换或积分饱和时输出易跳变
    2. 增量式: 输出控制量的增量 $\Delta u(k) = u(k) - u(k - 1)$, 只与最近几次误差有关, 无累加项, 限幅与无扰切换更容易
    3. 增量式算式:

$$\Delta u(k) = K_p \left[ e(k) - e(k - 1) \right] + K_i e(k) + K_d \left[ e(k) - 2 e(k - 1) + e(k - 2) \right]$$

6. 工程地位: 结构简单、不需模型、鲁棒性好, 是电机、温度、飞行器等控制中应用最广的算法
