---
created_at: 2026-09-17
updated_at: 2026-09-18
tags:
    - 数电
    - 电子电路基础
archived: false
---

[数电](./数电.md) / 9. 硬件描述语言 / 9.3 描述电路 / 9.3.2 Verilog时序逻辑与状态机

# Verilog时序逻辑与状态机

## 核心结论

时序逻辑的模板只有一句: **`always @(posedge clk)` + 非阻塞赋值 `<=`**. 所有时序电路(寄存器, 计数器, 移位寄存器, 状态机) 都是这一模板的变体.

## 一、D 触发器与寄存器

### 最基本的 D 触发器

```verilog
module dff(
    input  wire clk,
    input  wire d,
    output reg  q
);

always @(posedge clk) begin
    q <= d;
end

endmodule
```

### 带异步复位的触发器

```verilog
module dff_async_rst(
    input  wire clk,
    input  wire rst_n,     // 低电平有效的异步复位
    input  wire d,
    output reg  q
);

always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        q <= 1'b0;         // 复位优先,且不受时钟控制
    else
        q <= d;
end

endmodule
```

**异步复位的要点**: 敏感表中必须同时列出 `posedge clk` 与 `negedge rst_n`, 且复位分支写在最前面.

### 带同步复位的触发器

```verilog
always @(posedge clk) begin
    if (!rst_n)
        q <= 1'b0;         // 只在时钟沿生效
    else
        q <= d;
end
```

**同步与异步复位的对比**:

| 对比项   | 异步复位               | 同步复位                 |
| -------- | ---------------------- | ------------------------ |
| 敏感表   | 含 `negedge rst_n`     | 只有 `posedge clk`       |
| 生效时刻 | 立即                   | 等下一个时钟沿           |
| 资源消耗 | 使用触发器的异步复位端 | 占用数据输入端的组合逻辑 |
| 时序要求 | 需满足**恢复时间**     | 需满足**建立/保持时间**  |
| 推荐做法 | "异步复位, 同步释放"   | 时序干净的场合           |

## 二、计数器

### 带使能的加法计数器

```verilog
module counter #(
    parameter WIDTH = 8
)(
    input  wire             clk,
    input  wire             rst_n,
    input  wire             en,
    output reg  [WIDTH-1:0] cnt
);

always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        cnt <= {WIDTH{1'b0}};
    else if (en)
        cnt <= cnt + 1'b1;
end

endmodule
```

### 模 M 计数器(计数到设定值归零)

```verilog
module counter_mod10(
    input  wire       clk,
    input  wire       rst_n,
    output reg  [3:0] cnt,
    output wire       cout
);

always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        cnt <= 4'd0;
    else if (cnt == 4'd9)      // 计到 9 归零
        cnt <= 4'd0;
    else
        cnt <= cnt + 1'b1;
end

assign cout = (cnt == 4'd9);   // 计满产生进位

endmodule
```

**这正是一个同步十进制计数器**, 与用 74LS160 实现的功能完全等价.

### 可逆计数器

```verilog
always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        cnt <= 4'd0;
    else if (en) begin
        if (up)
            cnt <= cnt + 1'b1;
        else
            cnt <= cnt - 1'b1;
    end
end
```

## 三、移位寄存器

```verilog
module shift_reg #(
    parameter WIDTH = 8
)(
    input  wire clk,
    input  wire rst_n,
    input  wire din,
    output wire dout
);

reg [WIDTH-1:0] q;

always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        q <= {WIDTH{1'b0}};
    else
        q <= {q[WIDTH-2:0], din};   // 左移,低位补入新数据
end

assign dout = q[WIDTH-1];           // 串行输出

endmodule
```

**要点**: 移位用**拼接运算符**描述, 一行即可; 由于所有位在同一个 `always` 块中用非阻塞赋值, 所有位同时更新, 不会出现逐级穿透.

## 四、分频器

```verilog
module clk_div #(
    parameter DIV = 10      // 分频系数
)(
    input  wire clk,
    input  wire rst_n,
    output reg  clk_out
);

reg [3:0] cnt;

always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        cnt     <= 4'd0;
        clk_out <= 1'b0;
    end
    else if (cnt == DIV / 2 - 1) begin
        cnt     <= 4'd0;
        clk_out <= ~clk_out;      // 半周期翻转
    end
    else
        cnt <= cnt + 1'b1;
end

endmodule
```

**注意**: 这种用计数器直接产生时钟的做法在 FPGA 中会产生**额外的时钟域**, 应优先使用器件自带的 **PLL/MMCM** 或**时钟使能**方式.

## 五、有限状态机(FSM)

### 三段式写法(推荐)

把状态机分成三个 `always` 块: **状态寄存器 + 次态逻辑 + 输出逻辑**.

```verilog
module fsm_example(
    input  wire clk,
    input  wire rst_n,
    input  wire x,
    output reg  y
);

// ---------- 1. 状态定义 ----------
localparam S0 = 2'b00;
localparam S1 = 2'b01;
localparam S2 = 2'b10;

reg [1:0] state, next_state;

// ---------- 2. 状态寄存器(时序逻辑)----------
always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        state <= S0;
    else
        state <= next_state;
end

// ---------- 3. 次态逻辑(组合逻辑)----------
always @(*) begin
    case (state)
        S0: next_state = x ? S1 : S0;
        S1: next_state = x ? S2 : S0;
        S2: next_state = x ? S2 : S0;
        default: next_state = S0;
    endcase
end

// ---------- 4. 输出逻辑 ----------
// Moore 型:输出只取决于状态
always @(*) begin
    case (state)
        S2: y = 1'b1;
        default: y = 1'b0;
    endcase
end

endmodule
```

上面这个例子正是第 6 章"111 序列检测器"的 Moore 型实现.

**Mealy 型版本**(输出与输入有关):

```verilog
always @(*) begin
    case (state)
        S2: y = x;          // 在 S2 且输入为 1 时输出 1
        default: y = 1'b0;
    endcase
end
```

### 两段式写法

把状态寄存器单独一块, 次态逻辑与输出逻辑合并一块. 代码更短, 但输出可能随输入产生毛刺(Mealy 型), 且综合工具优化空间略小.

### 一段式写法

所有逻辑写在一个 `always` 块中. 代码最紧凑, 但结构混乱, 不易维护,**不推荐**.

### 三种写法的对比

| 写法       | 结构                  | 优点                             | 缺点           |
| ---------- | --------------------- | -------------------------------- | -------------- |
| 一段式     | 全部在一个块          | 代码短                           | 难维护, 易出错 |
| 两段式     | 状态寄存器 + 组合逻辑 | 较常用                           | 输出可能带毛刺 |
| **三段式** | 寄存器 + 次态 + 输出  | **结构清晰, 输出无毛刺, 易维护** | 代码略长       |

## 六、状态编码

| 编码方式          | Verilog 实现                         | 适用                            |
| ----------------- | ------------------------------------ | ------------------------------- |
| 二进制            | `localparam S0 = 2'b00;`             | 通用, 省触发器                  |
| 格雷码            | `S0 = 00, S1 = 01, S2 = 11, S3 = 10` | 状态按序转移时减少竞争          |
| 一位热码(One-Hot) | `S0 = 4'b0001, S1=4'b0010`           | **FPGA 首选**, 速度快, 逻辑简单 |

**FPGA 推荐 One-Hot 的原因**: FPGA 触发器资源丰富, One-Hot 使次态逻辑和输出逻辑都只需检查一位, 组合逻辑级数最少, 速度最快.

## 七、状态机的设计检查清单

1. 是否有复位, 复位后是否进入确定状态?
2. `case` 是否写了 `default`?(防止综合出锁存器, 也防止卡死)
3. 次态逻辑是否覆盖了所有状态与输入组合?
4. 输出逻辑是 Moore 还是 Mealy, 与需求是否一致?
5. 是否考虑了无效状态的恢复(自启动)?

## 八、常见错误

| 错误                              | 后果                               |
| --------------------------------- | ---------------------------------- |
| 状态寄存器块用了阻塞赋值          | 状态更新"短路", 仿真与综合不一致   |
| `case` 缺少 `default`             | 综合出锁存器, 或进入无效状态后卡死 |
| 次态逻辑敏感表不完整              | 仿真与实际不符                     |
| 输出用 `reg` 却在 `assign` 中赋值 | 语法错误                           |
| 计数器位宽不够                    | 计数溢出, 行为异常                 |

## 易错点

- 认为"时序逻辑块里写 `cnt <= cnt + 1` 是组合逻辑": 只要敏感表是 `posedge clk`, 就综合成触发器加加法器.
- 在状态机中把次态逻辑写成时序逻辑(用 `<=` 且在 `posedge` 块内), 会使状态转换延迟一拍, 与预期不符.

## 相关笔记

- 前置: [Verilog赋值与过程语句](./Verilog赋值与过程语句.md)
- 电路原理: [时序逻辑电路的设计方法](./时序逻辑电路的设计方法.md)
- 时序约束: [触发器动态特性](./触发器动态特性.md)
- 验证: [Verilog测试平台与仿真](./Verilog测试平台与仿真.md)
