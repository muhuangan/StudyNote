---
created_at: 2026-09-17
updated_at: 2026-09-18
tags:
    - 数电
    - 电子电路基础
archived: false
---

[数电](./数电.md) / 9. 硬件描述语言 / 9.3 描述电路 / 9.3.1 Verilog组合逻辑描述

# Verilog组合逻辑描述

## 核心结论

组合逻辑只有两种写法: **`assign` 连续赋值**与 **`always @(*)` 阻塞赋值**. 两条铁律: 敏感表必须用 `@(*)`, 所有分支必须完整赋值(否则产生锁存器).

## 一、两种写法的选择

| 场景                             | 推荐写法      |
| -------------------------------- | ------------- |
| 简单的表达式(与或非, 拼接, 移位) | `assign`      |
| 复杂的多分支选择(if / case)      | `always @(*)` |
| 需要中间变量                     | `always @(*)` |

## 二、基本门电路

```verilog
module basic_gates(
    input  wire a,
    input  wire b,
    output wire y_and,
    output wire y_or,
    output wire y_not,
    output wire y_nand,
    output wire y_nor,
    output wire y_xor,
    output wire y_xnor
);

assign y_and  = a & b;
assign y_or   = a | b;
assign y_not  = ~a;
assign y_nand = ~(a & b);
assign y_nor  = ~(a | b);
assign y_xor  = a ^ b;
assign y_xnor = ~(a ^ b);

endmodule
```

## 三、多路选择器(MUX)

### 用条件运算符(最简洁)

```verilog
module mux2_1(
    input  wire [7:0] d0,
    input  wire [7:0] d1,
    input  wire       sel,
    output wire [7:0] y
);

assign y = sel ? d1 : d0;

endmodule
```

### 用 case 语句(通道多时清晰)

```verilog
module mux4_1(
    input  wire [7:0] d0, d1, d2, d3,
    input  wire [1:0] sel,
    output reg  [7:0] y
);

always @(*) begin
    case (sel)
        2'b00: y = d0;
        2'b01: y = d1;
        2'b10: y = d2;
        2'b11: y = d3;
        default: y = 8'h00;
    endcase
end

endmodule
```

## 四、译码器

```verilog
module decoder_3_8(
    input  wire [2:0] a,
    input  wire       en,
    output reg  [7:0] y
);

always @(*) begin
    if (en == 1'b0)
        y = 8'b0000_0000;        // 使能无效,全部输出无效
    else
        case (a)
            3'b000: y = 8'b0000_0001;
            3'b001: y = 8'b0000_0010;
            3'b010: y = 8'b0000_0100;
            3'b011: y = 8'b0000_1000;
            3'b100: y = 8'b0001_0000;
            3'b101: y = 8'b0010_0000;
            3'b110: y = 8'b0100_0000;
            3'b111: y = 8'b1000_0000;
            default: y = 8'b0000_0000;
        endcase
end

endmodule
```

**等效写法**(更简洁): `assign y = en ? (8'b1 << a) : 8'b0;`

## 五、优先编码器

```verilog
module priority_encoder_8_3(
    input  wire [7:0] req,
    output reg  [2:0] code,
    output reg        valid
);

always @(*) begin
    if (req[7]) begin
        code = 3'd7; valid = 1'b1;
    end else if (req[6]) begin
        code = 3'd6; valid = 1'b1;
    end else if (req[5]) begin
        code = 3'd5; valid = 1'b1;
    end else if (req[4]) begin
        code = 3'd4; valid = 1'b1;
    end else if (req[3]) begin
        code = 3'd3; valid = 1'b1;
    end else if (req[2]) begin
        code = 3'd2; valid = 1'b1;
    end else if (req[1]) begin
        code = 3'd1; valid = 1'b1;
    end else if (req[0]) begin
        code = 3'd0; valid = 1'b1;
    end else begin
        code = 3'd0; valid = 1'b0;
    end
end

endmodule
```

**要点**: `if...else if` 链天然表达优先级, 第 7 章所学的 74LS148 功能完全一样.

## 六、加法器

```verilog
module adder_4bit(
    input  wire [3:0] a,
    input  wire [3:0] b,
    input  wire       cin,
    output wire [3:0] sum,
    output wire       cout
);

assign {cout, sum} = a + b + cin;   // 拼接:最高位承接进位

endmodule
```

**位宽要点**: `{cout, sum}` 共 5 位, 足以容纳 4 位加 4 位再加进位的结果, 进位不会丢失.

## 七、数值比较器

```verilog
module comparator_4bit(
    input  wire [3:0] a,
    input  wire [3:0] b,
    output wire       gt,   // a > b
    output wire       eq,   // a == b
    output wire       lt    // a < b
);

assign gt = (a > b);
assign eq = (a == b);
assign lt = (a < b);

endmodule
```

## 八、七段显示译码器

```verilog
module seg_decoder(
    input  wire [3:0] bcd,
    output reg  [6:0] seg    // seg[6:0] = {a,b,c,d,e,f,g},共阴极,高电平点亮
);

always @(*) begin
    case (bcd)
        4'h0: seg = 7'b111_1110;
        4'h1: seg = 7'b011_0000;
        4'h2: seg = 7'b110_1101;
        4'h3: seg = 7'b111_1001;
        4'h4: seg = 7'b011_0011;
        4'h5: seg = 7'b101_1011;
        4'h6: seg = 7'b101_1111;
        4'h7: seg = 7'b111_0000;
        4'h8: seg = 7'b111_1111;
        4'h9: seg = 7'b111_1011;
        default: seg = 7'b000_0000;   // 伪码熄灭
    endcase
end

endmodule
```

## 九、避免意外锁存器(最重要)

**错误写法**(综合出锁存器):

```verilog
always @(*) begin
    if (sel == 2'b00)
        y = a;
    else if (sel == 2'b01)
        y = b;
    // 缺少 else:sel 为其他值时 y 需要"保持",于是综合出锁存器
end
```

**正确写法一**(补 else):

```verilog
always @(*) begin
    if (sel == 2'b00)
        y = a;
    else if (sel == 2'b01)
        y = b;
    else
        y = 1'b0;
end
```

**正确写法二**(默认赋值):

```verilog
always @(*) begin
    y = 1'b0;
    if (sel == 2'b00)
        y = a;
    else if (sel == 2'b01)
        y = b;
end
```

**为什么锁存器是问题**:

1. 锁存器是电平敏感的, 会给时序分析带来困难.
2. FPGA 中实现锁存器要消耗额外资源, 且容易产生毛刺.
3. 绝大多数情况下"保持原值"并不是设计者的本意, 而是漏写分支造成的.

**例外**: 若确实需要锁存器(如某些总线保持电路), 应显式说明并确认目标器件支持.

## 十、组合逻辑的描述检查清单

1. 敏感表是否为 `@(*)`?
2. `if` / `case` 是否覆盖全部分支?
3. 是否用了阻塞赋值 `=`?
4. 输出是否声明为 `reg`(`always` 中赋值) 或 `wire`(`assign` 中赋值)?
5. 位宽是否足够(尤其加法与移位)?
6. 是否检查了竞争-冒险(关键输出路径)?

## 易错点

- 在 `always` 块中描述组合逻辑却用了非阻塞赋值 `<=`: 功能上往往也能工作, 但仿真行为与综合结果可能不一致, 且不符合编码规范.
- 敏感表写死成 `always @(a)` 而漏掉其他信号: 这是**仿真与综合不一致**的经典来源, 必须写 `@(*)`.

## 相关笔记

- 前置: [Verilog赋值与过程语句](./Verilog赋值与过程语句.md)
- 电路原理: [组合逻辑电路的设计方法](./组合逻辑电路的设计方法.md)
- 对照: [Verilog时序逻辑与状态机](./Verilog时序逻辑与状态机.md)
