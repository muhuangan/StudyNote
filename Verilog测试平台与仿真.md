---
created_at: 2026-09-17
updated_at: 2026-09-18
tags:
    - 数电
    - 电子电路基础
archived: false
---

[数电](./数电.md) / 第九章 硬件描述语言 / 9.3 testbench 编写与仿真

# Verilog测试平台与仿真

## 核心结论

测试平台(testbench) 是一个**不被综合, 只用于仿真**的 Verilog 模块: 它实例化被测设计(DUT), 产生激励信号, 把响应打印或存成波形供观察. testbench 可以使用全部 Verilog 语法(包括 `initial`,`#` 延迟,`$display`).

## 一、testbench 的基本结构

```verilog
`timescale 1ns / 1ps      // 时间单位 / 时间精度

module tb_design_name;

// ---------- 1. 信号声明 ----------
reg         clk;
reg         rst_n;
reg  [3:0]  din;
wire [6:0]  dout;

// ---------- 2. 实例化被测设计 ----------
design_name uut (
    .clk  (clk),
    .rst_n(rst_n),
    .din  (din),
    .dout (dout)
);

// ---------- 3. 产生时钟 ----------
always #5 clk = ~clk;     // 周期 10 ns,即 100 MHz

// ---------- 4. 产生激励 ----------
initial begin
    // 初始化
    clk   = 1'b0;
    rst_n = 1'b0;
    din   = 4'd0;

    // 复位
    #100 rst_n = 1'b1;

    // 施加激励
    #20 din = 4'd1;
    #20 din = 4'd2;
    #20 din = 4'd3;

    // 结束仿真
    #100 $finish;
end

// ---------- 5. 记录波形 ----------
initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, tb_design_name);
end

endmodule
```

## 二、四个关键要素

| 要素     | 作用               | 常用语法                          |
| -------- | ------------------ | --------------------------------- |
| 时间尺度 | 定义延迟单位与精度 | `` `timescale 1ns/1ps ``          |
| 激励产生 | 给输入信号赋值     | `initial`,`always`                |
| 时钟产生 | 产生周期性时钟     | `always #半周期 clk = ~clk;`      |
| 结果记录 | 打印或存波形       | `$display`,`$monitor`,`$dumpfile` |
| 结束控制 | 终止仿真           | `$finish`,`$stop`                 |

## 三、常用系统任务

| 任务                      | 作用                           |
| ------------------------- | ------------------------------ |
| `$display`                | 打印一次信息(类似 printf)      |
| `$monitor`                | **每当变量变化时**自动打印     |
| `$write`                  | 打印但不换行                   |
| `$strobe`                 | 在仿真时刻结束时打印(取稳定值) |
| `$time`                   | 返回当前仿真时间               |
| `$finish`                 | 结束仿真并退出                 |
| `$stop`                   | 暂停仿真(进入交互模式)         |
| `$dumpfile`               | 指定波形文件名                 |
| `$dumpvars`               | 指定要记录的信号范围           |
| `$readmemh` / `$readmemb` | 从文件读入存储器内容           |
| `$random`                 | 产生随机数                     |

**格式化输出**:

```verilog
$display("time=%0t  din=%b  dout=%h", $time, din, dout);
```

| 格式符 | 含义                 |
| ------ | -------------------- |
| `%b`   | 二进制               |
| `%d`   | 十进制               |
| `%h`   | 十六进制             |
| `%o`   | 八进制               |
| `%t`   | 时间                 |
| `%0d`  | 十进制, 不补前导空格 |

## 四、时钟的几种产生方式

```verilog
// 方式一:固定周期(最常用)
always #5 clk = ~clk;

// 方式二:带占空比调整
always begin
    #3 clk = 1'b1;
    #7 clk = 1'b0;    // 占空比 30%
end

// 方式三:在 initial 中初始化后自翻转
initial begin
    clk = 1'b0;
    forever #5 clk = ~clk;
end
```

**注意**: 时钟初值必须在 `initial` 中给定(否则为 `x`,`~x` 仍是 `x`, 时钟不会翻转).

## 五、复位信号的产生

```verilog
initial begin
    rst_n = 1'b0;      // 复位有效
    #100 rst_n = 1'b1; // 100 ns 后释放
end
```

**实践建议**: 复位释放后应等待若干时钟周期再施加激励, 以便观察复位后的初始状态.

## 六、激励的三种产生方式

### 1. 直接列举(适合简单时序)

```verilog
initial begin
    #100 din = 4'd0;
    #20  din = 4'd1;
    #20  din = 4'd2;
end
```

### 2. 与时钟同步(推荐)

```verilog
initial begin
    @(posedge rst_n);        // 等待复位释放
    repeat (5) @(posedge clk);  // 等 5 个时钟周期
    din <= 4'd1;
    @(posedge clk);
    din <= 4'd2;
end
```

**优势**:` @(posedge clk)` 让激励在时钟沿附近变化, 模拟真实时序, 便于观察建立/保持关系.

### 3. 随机激励(适合大范围验证)

```verilog
always @(posedge clk) begin
    din <= $random % 16;
end
```

## 七、自动检查(自检 testbench)

好的 testbench 不只是打印波形, 还应**自动判断结果是否正确**:

```verilog
integer error_count;

initial error_count = 0;

always @(posedge clk) begin
    if (dout !== expected) begin
        $display("ERROR at time %0t: dout=%h, expected=%h", $time, dout, expected);
        error_count = error_count + 1;
    end
end

initial begin
    #10000;
    if (error_count == 0)
        $display("==== ALL TESTS PASSED ====");
    else
        $display("==== FAILED: %0d errors ====", error_count);
    $finish;
end
```

## 八、文件读写

```verilog
// 从十六进制文件加载存储器
reg [7:0] mem [0:255];
initial $readmemh("data.hex", mem);

// 写文件
integer fd;
initial begin
    fd = $fopen("result.txt", "w");
    $fdisplay(fd, "result = %h", dout);
    $fclose(fd);
end
```

## 九、仿真流程与工具

**基本流程**:

1. 编写 RTL 代码与 testbench.
2. 编译(检查语法).
3. 运行仿真, 生成波形文件.
4. 打开波形查看工具, 检查时序与功能.
5. 若发现问题, 修改代码并重复.

**常见工具**:

| 工具                     | 类型           | 特点                                  |
| ------------------------ | -------------- | ------------------------------------- |
| Icarus Verilog(iverilog) | 开源命令行     | 轻量, 配合 GTKWave 查看波形, 学习首选 |
| Verilator                | 开源           | 速度极快, 把 Verilog 转成 C++ 仿真    |
| GTKWave                  | 开源波形查看器 | 配合 iverilog 使用                    |
| ModelSim / QuestaSim     | 商业           | 功能全面, 支持混合语言仿真            |
| VCS                      | 商业           | 速度最快, 工业界主流                  |
| Vivado XSim              | 厂商自带       | Xilinx FPGA 开发流程集成              |

**快速上手命令(iverilog + GTKWave)**:

```bash
iverilog -o sim.vvp design.v tb_design.v
vvp sim.vvp
gtkwave wave.vcd
```

## 十、仿真中的常见现象

| 现象           | 原因                                         |
| -------------- | -------------------------------------------- |
| 信号全为 `x`   | 没有复位, 或复位未生效                       |
| 时钟不翻转     | 时钟未初始化(`x` 取反仍是 `x`)               |
| 输出延迟一拍   | 正常现象: 寄存器输出本来就比输入晚一个时钟沿 |
| 仿真卡死不结束 | 缺少 `$finish`, 或存在组合逻辑环             |
| 前后仿真不一致 | 敏感表不完整, 或阻塞/非阻塞赋值用错          |

## 十一、编写 testbench 的建议

1. **先写简单激励验证基本功能**, 再逐步增加边界用例.
2. **复位是第一步**, 所有观察都从复位释放之后开始.
3. 用 ` @(posedge clk)` 而不是固定延迟`#`来同步激励.
4. 关键信号加到 `$monitor` 中持续观察.
5. 大设计用**自动检查**代替人工看波形.
6. 仿真时间要足够长, 覆盖状态机的所有状态与边界条件.

## 易错点

- 忘记在 `initial` 中给 `clk` 赋初值, 导致时钟一直是 `x`.
- testbench 中把 `wire` 型的激励信号声明成了 `reg`, 或反之(激励用 `reg`, DUT 输出用 `wire`).
- 认为"仿真通过 = 电路能工作": 功能仿真**不含延迟**, 时序问题(建立/保持违规) 只有在后仿真或静态时序分析中才会暴露.

## 相关笔记

- 前置:[Verilog时序逻辑与状态机](./Verilog时序逻辑与状态机.md)
- 综合与实现:[现场可编程门阵列](./现场可编程门阵列.md)
- 时序约束:[触发器动态特性](./触发器动态特性.md)
