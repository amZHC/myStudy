我这个笔记适合你学习着其他课程的时候顺便看一眼，因为好多课程只讲核心实现思路，不带着手敲一遍且视频教程里的代码和开发板配套的课程示例代码不一样，没错我说的就是小梅哥。我作为FPGA初学者就很难受，所以记录一篇这样的笔记，如果能帮到你，我很开心。

祝您生活愉快，学习顺利。



# FPGA_study

个人学习FPGA笔记。
入门课程由DeepSeek生成。
项目是我自己编写项目，每一个包含必要的模块代码、tb代码和xdc约束文件（我的设备是小梅哥7015带光模块的学习板，软件是Vivado2024.2）。



### 入门

#### 第一阶段：认识Verilog的“单词”——关键字

Verilog的**关键字**就是它的保留词，就像英语里的“if”、“else”一样。初学者先记住这**10个最常用的**就够了：

| 关键字                       | 含义（白话版）                                   | 出现场景                   |
| ---------------------------- | ------------------------------------------------ | -------------------------- |
| `module` / `endmodule`       | **模块**，就是一块电路板的“外壳”                 | 每个文件的开头和结尾       |
| `input` / `output` / `inout` | **输入/输出/双向口**，芯片的引脚方向             | 端口声明                   |
| `wire`                       | **线**，就像电路板上的铜导线                     | 连接各个元件               |
| `reg`                        | **寄存器**，能记住数据的单元（但也可做组合逻辑） | 在`always`块里被赋值的变量 |
| `always`                     | **总是执行**，描述“当某条件发生时，做什么事”     | 组合逻辑或时序逻辑         |
| `assign`                     | **赋值**，用“=”连接两个信号                      | 简单组合逻辑               |
| `if` / `else`                | **如果...否则...**，选择逻辑                     | 条件判断                   |
| `case` / `default`           | **分支选择**，多路开关                           | 多条件判断                 |
| `posedge` / `negedge`        | **上升沿/下降沿**，时钟的跳变                    | 时序逻辑触发               |
| `initial`                    | **初始化**，只执行一次（仅仿真用）               | Testbench里                |

> 💡 **记忆口诀**：
> “**模块**两头要记牢，**输入输出**加**线/寄存器**；
> **always**配**posedge**是时序，**assign**配**wire**是组合；
> **if case**做选择，**initial**只在仿真跑。”

#### 第二阶段：理解“电路图”的三种基本结构

Verilog不是让你写代码，而是让你**画电路图**。有三种最基本的电路：

##### 🔹 结构1：直接连接（assign）

就像把两个电阻直接焊在一起。

```verilog
assign y = a & b;  // y 永远等于 a 与 b，延迟极小
```



##### 🔹 结构2：组合逻辑盒子（always @(*)）

就像一块“组合芯片”，输入一变，输出立刻跟着变（有微小延迟）。

```verilog
always @(*) begin  // @(*) 意思是“只要输入变了就触发”
    y = a & b;     // 这里用 = （阻塞赋值）
end
```



##### 🔹 结构3：时序逻辑盒子（always @(posedge clk)）

就像“照相机”，只在**时钟上升沿**那一瞬间，拍下输入的值，存到输出。

```verilog
always @(posedge clk) begin
    y <= a & b;    // 注意这里用 <= （非阻塞赋值）
end
```



> ⚠️ **初学者最容易犯的错**：
>
> - 在 `always @(*)` 里用了 `<=`（应该用 `=`）
> - 在 `always @(posedge clk)` 里用了 `=`（应该用 `<=`）
> 	**口诀**：组合用等号（=），时序用箭头（<=）！

#### 第三阶段：动手写第一个完整模块（与门）

把你学到的关键字用起来：

```verilog
module and_gate (      // 模块开始
    input a, b,        // 输入端口
    output y           // 输出端口
);
    assign y = a & b;  // 连续赋值
endmodule              // 模块结束
```

**这就是一个完整的芯片！** 你做了个“与门”。

#### 第四阶段：学会“记住”数据（D触发器）

要让电路有“记忆”，就需要**寄存器（reg）**。

verilog

```verilog
module d_flipflop (
    input clk,        // 时钟
    input d,          // 数据输入
    input rst_n,      // 复位（低有效）
    output reg q      // 数据输出
);
    always @(posedge clk or negedge rst_n) begin  // 时钟上升沿或复位下降沿
        if (!rst_n) 
            q <= 1'b0;      // 复位时清零
        else 
            q <= d;         // 否则把d的值存到q
    end
endmodule
```

**关键理解**：

- `posedge clk`：只在时钟上升沿干活
- `or negedge rst_n`：复位信号一掉电，马上清零（**异步复位**）
- `q <= d`：**非阻塞赋值**，意味着“这一刻拍个照，下一时刻才输出”

#### 第五阶段：组合逻辑的“大脑”（多路选择器）

用 `always @(*)` + `case` 实现一个4选1选择器：

```verilog
module mux4_1 (
    input [1:0] sel,    // 2位选择信号
    input [3:0] in,     // 4位输入（in[0]~in[3]）
    output reg y
);
    always @(*) begin
        case (sel)
            2'b00: y = in[0];
            2'b01: y = in[1];
            2'b10: y = in[2];
            2'b11: y = in[3];
            default: y = 1'b0;   // 防锁存！必须写
        endcase
    end
endmodule
```

> ⚠️ **防锁存铁律**：
> 所有 `always @(*)` 里的 `if` 必须配 `else`，`case` 必须配 `default`，否则综合器会给你生成一个“锁存器”（不是你想要的）。

#### 第六阶段：组合起来（计数器 + 数码管）

现在把前面学的串起来：用**D触发器**记住计数值，用**组合逻辑**译码显示。

```verilog
module counter_top (
    input clk, rst_n,
    output reg [6:0] seg   // 7段数码管
);
    reg [3:0] cnt;         // 计数值
    
    // 时序部分：计数器
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) 
            cnt <= 4'd0;
        else 
            cnt <= cnt + 1'b1;   // 每拍加1
    end
    
    // 组合部分：译码器（用 always @(*)）
    always @(*) begin
        case (cnt)
            4'd0: seg = 7'b1111110;
            4'd1: seg = 7'b0110000;
            4'd2: seg = 7'b1101101;
            4'd3: seg = 7'b1111001;
            4'd4: seg = 7'b0110011;
            4'd5: seg = 7'b1011011;
            4'd6: seg = 7'b1011111;
            4'd7: seg = 7'b1110000;
            4'd8: seg = 7'b1111111;
            4'd9: seg = 7'b1111011;
            default: seg = 7'b0000000;
        endcase
    end
endmodule
```

#### 第七阶段：参数化设计（让模块变成“万能积木”）

用 `parameter` 定义常量，让模块可以**配置**，而不是写死。

```verilog
module param_counter #(
    parameter WIDTH = 8,        // 位宽可调
    parameter MAX = 255         // 计数最大值可调
)(
    input clk, rst_n,
    input en,
    output reg [WIDTH-1:0] cnt,
    output wire rollover        // 溢出标志
);
    assign rollover = (cnt == MAX);
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) 
            cnt <= 0;
        else if (en) begin
            if (cnt == MAX) 
                cnt <= 0;
            else 
                cnt <= cnt + 1'b1;
        end
    end
endmodule

// 例化时定制不同的宽度
module top;
    wire [7:0] cnt8;
    wire [15:0] cnt16;
    
    param_counter #(.WIDTH(8), .MAX(100)) u_cnt8 (...);   // 8位，数到100
    param_counter #(.WIDTH(16), .MAX(65535)) u_cnt16 (...); // 16位
endmodule
```

##### parameter 详解

###### 📌 基础语法结构

```verilog
module module_name #(
    parameter NAME1 = default_value1,
    parameter NAME2 = default_value2,
    ...
)(
    input/output ports
);
    // 模块内部逻辑
endmodule
```



###### 📌 三种声明形式对比

| 写法                     | 示例                             | 适用场景               |
| ------------------------ | -------------------------------- | ---------------------- |
| **单个声明**             | `parameter WIDTH = 8;`           | 简单模块，参数少       |
| **列表声明（推荐）**     | `#(parameter WIDTH=8, DEPTH=16)` | 专业写法，参数多时清晰 |
| **本地参数（不可覆盖）** | `localparam IDLE = 2'b00;`       | 状态机编码，不对外暴露 |

###### 📌 完整示例（带注释）

```verilog
// ============================================
// 方法1：在模块名后声明（最常用）
// ============================================
module fifo #(
    parameter DATA_WIDTH = 32,        // 数据位宽
    parameter ADDR_WIDTH = 8,         // 地址位宽（深度=2^ADDR_WIDTH）
    parameter DEPTH = 1 << ADDR_WIDTH // 支持表达式计算
)(
    input clk,
    input [DATA_WIDTH-1:0] wr_data,
    output [DATA_WIDTH-1:0] rd_data
);
    // 内部可以使用这些参数
    reg [DATA_WIDTH-1:0] mem [0:DEPTH-1];
    // ...
endmodule

// ============================================
// 方法2：在模块内部声明（传统写法，旧标准）
// ============================================
module old_fifo (clk, wr_data, rd_data);
    parameter DATA_WIDTH = 32;   // 注意：这里在端口列表后面
    parameter ADDR_WIDTH = 8;
    input clk;
    input [DATA_WIDTH-1:0] wr_data;
    output [DATA_WIDTH-1:0] rd_data;
    // ...
endmodule
```



> ⚠️ **重要区别**：
>
> - **方法1（#()）**：参数列表在端口列表**前面**，现代Verilog/SystemVerilog标准推荐
> - **方法2（内部声明）**：参数在端口列表**后面**，旧代码常见，但综合工具都支持

##### parameter 的重定义语法（修改默认值）

例化模块时，用 `#()` 传递新值，有两种写法：

###### 📌 语法对比表

| 重定义方式           | 语法结构                             | 优点           | 缺点             |
| -------------------- | ------------------------------------ | -------------- | ---------------- |
| **顺序映射**         | `#(8, 16)`                           | 简洁           | 容易错位，不推荐 |
| **名称映射（推荐）** | `#(.DATA_WIDTH(8), .ADDR_WIDTH(16))` | 清晰，顺序无关 | 稍长但安全       |

###### 📌 完整示例

```verilog
// 1. 先定义带参数的模块
module param_counter #(
    parameter WIDTH = 4,
    parameter MAX = 15
)(
    input clk, rst_n,
    output reg [WIDTH-1:0] cnt
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) cnt <= 0;
        else if (cnt < MAX) cnt <= cnt + 1'b1;
        else cnt <= 0;
    end
endmodule

// 2. 顶层例化时重定义
module top;
    wire [7:0] cnt_out;
    wire [3:0] cnt_small;
    
    // ❌ 不推荐：顺序映射（容易混淆）
    param_counter #(8, 255) u_counter1 (
        .clk(clk), 
        .rst_n(rst_n), 
        .cnt(cnt_out)
    );
    
    // ✅ 推荐：名称映射（清晰明了）
    param_counter #(
        .WIDTH(4),
        .MAX(10)
    ) u_counter2 (
        .clk(clk),
        .rst_n(rst_n),
        .cnt(cnt_small)
    );
endmodule
```

##### 高级用法（表达式和依赖）

参数之间可以相互依赖，甚至调用系统函数：

```verilog
module advanced_param #(
    // 1. 基础参数
    parameter DATA_WIDTH = 16,
    parameter DEPTH = 32,
    
    // 2. 依赖其他参数（必须在后面声明）
    parameter ADDR_WIDTH = $clog2(DEPTH),  // SystemVerilog函数，自动计算位宽
    parameter BYTE_WIDTH = DATA_WIDTH / 8,
    
    // 3. 字符串参数（用于仿真）
    parameter string MEM_INIT_FILE = "init.hex"
)(
    input clk,
    input [DATA_WIDTH-1:0] data_in,
    input [ADDR_WIDTH-1:0] addr
);
    // 使用参数
    reg [DATA_WIDTH-1:0] memory [0:DEPTH-1];
    
    // 仿真时打印参数值
    initial begin
        $display("DATA_WIDTH = %0d", DATA_WIDTH);
        $display("ADDR_WIDTH = %0d", ADDR_WIDTH);
        $display("BYTE_WIDTH = %0d", BYTE_WIDTH);
        $display("Init file: %s", MEM_INIT_FILE);
    end
endmodule
```

##### localparam 与 parameter 的区别

`localparam` 是**模块内部常量**，例化时**不能修改**，常用于状态机编码：

```verilog
module fsm_example (
    input clk, rst_n,
    output reg done
);
    // localparam：只能在模块内使用，无法从外部修改
    localparam IDLE = 2'b00,
               START = 2'b01,
               BUSY = 2'b10,
               DONE = 2'b11;
    
    reg [1:0] state, next_state;
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) state <= IDLE;
        else state <= next_state;
    end
    
    always @(*) begin
        case (state)
            IDLE:  next_state = START;
            START: next_state = BUSY;
            BUSY:  next_state = DONE;
            DONE:  next_state = IDLE;
            default: next_state = IDLE;
        endcase
    end
endmodule

// 例化时不能修改 IDLE 的值
// ❌ 错误：localparam不能重定义
// fsm_example #(.IDLE(3'b000)) u_fsm(...);  
```

##### 常见错误与最佳实践

###### ❌ 常见错误

```verilog
// 错误1：参数名冲突
module bad (
    input WIDTH,        // 端口名与参数同名
    parameter WIDTH = 8 // 编译报错！
);

// 错误2：忘记用 #() 传递参数
counter #(16) u_cnt(...);  // 如果模块定义了参数，必须用 #()
```



###### ✅ 最佳实践清单

1. **参数名用大写**：`DATA_WIDTH` vs `data_width`（约定俗成）
2. **提供合理的默认值**：让模块可以直接使用
3. **名称映射例化**：永远用 `.PARAM_NAME(value)`，别用顺序映射
4. **组合参数放在前面**：让用户先看到最重要的配置
5. **用 localparam 隐藏内部常量**：不让外部污染状态机编码

------

##### 实战模板（复制即可用）

```verilog
// =============================================
// 模板：带参数的AXI4-Lite从机接口
// =============================================
module axi_lite_slave #(
    // 用户可配置参数（提供默认值）
    parameter ADDR_WIDTH = 32,
    parameter DATA_WIDTH = 32,
    parameter REG_COUNT = 16,
    
    // 自动计算的参数（用户无需关心）
    parameter ADDR_LSB = $clog2(DATA_WIDTH/8),
    parameter DECODE_BITS = $clog2(REG_COUNT)
)(
    // AXI接口
    input clk, rst_n,
    input [ADDR_WIDTH-1:0] s_axi_awaddr,
    input s_axi_awvalid,
    // ... 省略其他信号
    output reg [DATA_WIDTH-1:0] reg_file [0:REG_COUNT-1]
);
    // 内部常量
    localparam IDLE = 2'b00,
               WRITE = 2'b01,
               READ = 2'b10;
    
    // 仿真时检查参数合法性
    initial begin
        if (REG_COUNT < 1) begin
            $error("REG_COUNT must be >= 1");
            $finish;
        end
    end
    
    // 实际逻辑...
endmodule
```

#### 第八阶段：双向总线（inout类型）

FPGA引脚经常要**既当输入又当输出**（如I2C的SDA），用 `inout` + **三态门**实现。

```verilog
module bidir_io (
    inout wire sda,      // 双向数据线
    input sda_out,       // 要发送的数据
    input sda_oe,        // 输出使能（1=输出模式，0=输入模式）
    output sda_in        // 读入的数据
);
    assign sda = sda_oe ? sda_out : 1'bz;   // 高阻态“交出总线”
    assign sda_in = sda;                     // 任何时候都能读
endmodule
```

#### 第九阶段：时钟分频器（从高频生成低频）

FPGA晶振通常50MHz或100MHz，但很多外设需要慢速时钟（如1Hz闪烁LED）。

verilog

```verilog
module clk_divider #(
    parameter DIV = 25_000_000    // 50MHz → 1Hz需要25_000_000次翻转
)(
    input clk, rst_n,
    output reg clk_out
);
    reg [31:0] counter;
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            counter <= 0;
            clk_out <= 0;
        end else begin
            if (counter == DIV/2 - 1) begin   // 翻转点
                clk_out <= ~clk_out;
                counter <= 0;
            end else begin
                counter <= counter + 1'b1;
            end
        end
    end
endmodule
```

#### 第十阶段：时钟使能（专业工程师的写法）

不生成新时钟，而是用**脉冲使能**让逻辑在特定时刻动作。

```verilog
module pulse_gen #(DIV=25_000_000)(
    input clk, rst_n,
    output reg pulse    // 每DIV个周期出一个高电平脉冲
);
    reg [31:0] cnt;
    
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            cnt <= 0;
            pulse <= 0;
        end else begin
            if (cnt == DIV - 1) begin
                cnt <= 0;
                pulse <= 1'b1;   // 只拉高一个周期
            end else begin
                cnt <= cnt + 1'b1;
                pulse <= 1'b0;
            end
        end
    end
endmodule

// 使用方式：所有逻辑仍然用主时钟clk
always @(posedge clk) begin
    if (pulse) begin    // 只有脉冲来时执行
        led_state <= ~led_state;
    end
end
```

#### 第十一阶段：流水线加法器（提升速度的杀手锏）

组合逻辑太深会导致频率跑不高，**流水线**能把大计算拆成小步骤。

```verilog
module pipeline_adder #(WIDTH=16)(
    input clk,
    input [WIDTH-1:0] a, b, c, d,   // 四个数相加
    output reg [WIDTH+1:0] result
);
    reg [WIDTH:0] sum1, sum2;       // 中间寄存器
    
    // 第一级流水：a+b 和 c+d 同时算
    always @(posedge clk) begin
        sum1 <= a + b;
        sum2 <= c + d;
    end
    
    // 第二级流水：两个中间结果相加
    always @(posedge clk) begin
        result <= sum1 + sum2;
    end
endmodule
```

**效果**：原本一次加法延时要10ns，现在每级只要5ns，时钟频率直接翻倍！

#### 第十二阶段：同步FIFO（跨时钟域的预备课）

FIFO是最常用的**数据缓冲**，理解它才能理解后面的异步FIFO。

verilog

```verilog
module sync_fifo #(
    parameter DEPTH = 16,
    parameter WIDTH = 8
)(
    input clk, rst_n,
    input wr_en, rd_en,
    input [WIDTH-1:0] wr_data,
    output [WIDTH-1:0] rd_data,
    output full, empty
);
    reg [WIDTH-1:0] mem [0:DEPTH-1];   // 存储数组
    reg [$clog2(DEPTH)-1:0] wr_ptr, rd_ptr;  // 读写指针
    reg [$clog2(DEPTH):0] count;        // 计数（比指针多1位）
    
    // 写操作
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            wr_ptr <= 0;
            count <= 0;
        end else if (wr_en && !full) begin
            mem[wr_ptr] <= wr_data;
            wr_ptr <= wr_ptr + 1'b1;
            count <= count + 1'b1;
        end else if (rd_en && !empty) begin
            count <= count - 1'b1;
        end
    end
    
    // 读操作（独立always，注意指针更新）
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) 
            rd_ptr <= 0;
        else if (rd_en && !empty) 
            rd_ptr <= rd_ptr + 1'b1;
    end
    
    assign rd_data = mem[rd_ptr];
    assign full = (count == DEPTH);
    assign empty = (count == 0);
endmodule
```

**关键点**：

- 指针位宽 `$clog2(DEPTH)` 自动计算所需位数
- `count` 比指针多1位，用来区分“满”和“空”
- 读写同时发生时，count不变（注意else优先级）



### Vivado使用技巧

我是自学FPGA编程，遇到了一些Vivado使用上的一些小细节坑，在这里做一下总结。

#### 启动脚本修改Vivado仿真时默认调用的线程数

生成bit流或其他操作时可以在开始时修改使用的线程数，仿真时就没有，调用的线程少仿真会很慢。通过Tcl Console修改后，软件重启就失效了。所以可以创建启动脚本修改默认调用的线程数，每次vivado启动时自动加载。

**创建流程**

```
1. 在软件安装目录下（...xlinx\Vivado\2024.2\scripts）创建Vivado_init.tcl文件
2. 在文件中添加
	set_param general.maxThreads 10
	后面这个10根据你自己的电脑配置修改，我的电脑是6核12线程，我让他默认调用10个
3. 保存文件并关闭，验证脚本是否生效
	重启软件，打开Tcl Console，输入 
	get_param general.maxThreads
	如果显示的数字与你设置的一致，表明脚本生效了
```

#### tb文件中增加一个你能看懂的结束标志

vivado仿真到固定时间会自动停一次，不论仿真有没有真正结束，我么就需要经常点击继续仿真按钮，我一开始不知道仿真结束后不会自动停，仿真一直在运行，我以为是电脑性能不行，还没跑完。。。。。。。为此还买了个新电脑

```
方法：
在tb文件最后$finish()前加一个打印输出
$display("仿真结束");
这会显示到控制台，提示你不用再点继续仿真了
```

#### 代码都正确，generate bitStream的时候报错没有引脚信息

检查你想使用的模块和xdc文件是不是都已经被 set as top



### 项目

#### mux2 二路选择器

**功能描述**
模块三个输入，其中一个为数据输出选择，根据sel决定模块输出a或b。

```verilog
// mux2.v
module mux2 (
    a,
    b,
    sel,
    out
);

    input a;
    input b;
    input sel;
    output out;

    assign out = (sel == 0) ? a : b;
endmodule

```



```verilog
// mux2_tb.b

`timescale 1ns / 1ns

module mux2_tb();

reg S0;
reg S1;
reg S2;

wire mux2_out;

mux2 mux2_inst0(
    .a(S0),
    .b(S1),
    .sel(S2),
    .out(mux2_out)
);

initial begin
    S2 = 0; S1 = 0; S0 = 0;
    #20;
    S2 = 0; S1 = 0; S0 = 1;
    #20;
    S2 = 0; S1 = 1; S0 = 0;
    #20;
    S2 = 0; S1 = 1; S0 = 1;
    #20;
    S2 = 1; S1 = 0; S0 = 0;
    #20;
    S2 = 1; S1 = 0; S0 = 1;
    #20;
    S2 = 1; S1 = 1; S0 = 0;
    #20;
    S2 = 1; S1 = 1; S0 = 1;
    #20;
end
endmodule

```



#### decoder_3_8 三八译码器

**功能描述**
输入三个1bit的输入，表示一个三位的二进制数据，对应计数为0-7。从而控制八个输出只有一个为选中状态。

```verilog
// decoder_3_8.v
module decoder_3_8(
    A0,
    A1,
    A2,
    Y0,
    Y1,
    Y2,
    Y3,
    Y4,
    Y5,
    Y6,
    Y7
);

    input A0;
    input A1;
    input A2;
    output reg Y0;   // 默认都是信号，标记为reg后才能赋值
    output reg Y1;
    output reg Y2;
    output reg Y3;
    output reg Y4;
    output reg Y5;
    output reg Y6;
    output reg Y7;


// 过程赋值语句
always @(*) 
    case ({A2,A1,A0})
        3'b000: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b0000_0001;  // _ 是占位符，方便查看
        3'b001: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b0000_0010;
        3'b010: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b0000_0100;
        3'b011: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b0000_1000;

        3'b100: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b0001_0000;
        3'b101: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b0010_0000;
        3'b110: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b0100_0000;
        3'b111: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b1000_0000;

        default: {Y7,Y6,Y5,Y4,Y3,Y2,Y1,Y0} = 8'b0000_0000;   // 没有列举所有可能的值时，写default
    endcase

// endmodule后面必须有空行
endmodule

```

```verilog
// decode_3_8_tb.v
`timescale 1ns/1ns

module decoder_3_8_tb();

    reg A0;
    reg A1;
    reg A2;

    wire Y0;
    wire Y1;
    wire Y2;
    wire Y3;
    wire Y4;
    wire Y5;
    wire Y6;
    wire Y7;

    // 模块实例
    decoder_3_8 decoder_3_8_inst0 (
        .A0(A0),
        .A1(A1),
        .A2(A2),
        .Y0(Y0),
        .Y1(Y1),
        .Y2(Y2),
        .Y3(Y3),
        .Y4(Y4),
        .Y5(Y5),
        .Y6(Y6),
        .Y7(Y7)

    );

    initial begin
        A2 = 0; A1 = 0; A0 = 0;
        #20;
        A2 = 0; A1 = 0; A0 = 1;
        #20;
        A2 = 0; A1 = 1; A0 = 0;
        #20;
        A2 = 0; A1 = 1; A0 = 1;
        #20;

        A2 = 1; A1 = 0; A0 = 0;
        #20;
        A2 = 1; A1 = 0; A0 = 1;
        #20;
        A2 = 1; A1 = 1; A0 = 0;
        #20;
        A2 = 1; A1 = 1; A0 = 1;
        #20;

        // 停止，防止仿真波形后面拉长
        $stop(0);
    end


endmodule

```



#### Led_twinkle led闪烁，时序逻辑

**功能描述**
通过时钟信号，控制开发板上的LED灯以固定间隔时间闪烁。

```verilog
// led_twinkle.v
module Led_twinkle(
    Clk,
    Reset_n,
    Led
);

    input Clk;
    input Reset_n;
    output reg Led;

    reg  [24:0] counter;

    // always 表示一直执行

    // 利用 50MHz 的晶振，实现1s为周期的Led闪烁
    always @(posedge Clk or negedge Reset_n)  // 时序逻辑  上升沿  或 下降沿
    if (!Reset_n)                   // 非下降沿执行
        counter <= 0;               // counter 复位为 0
    else if(counter == 25_000_000 - 1)  // 25m次，实际是25m-1次变化
        counter <= 0;
    else
        counter <= counter + 1'd1;  // counter += 1  1'd1(十进制的1)
    
    // <= 表示非阻塞赋值
    // = 表示阻塞赋值

    always @(posedge Clk or negedge Reset_n)
    if(!Reset_n)
        Led <= 1'b0;
    else if(counter == 25_000_000 - 1)
        Led <= !Led;     // 取反
endmodule

```

```verilog
// led_twinkle_t.v
`timescale 1ns/1ns

module Led_twinkle_tb();

    // 定义
    reg Clk;
    reg Reset_n;
    wire Led;

    // 例化
    Led_twinkle Led_twinkle(
        .Clk(Clk),
        .Reset_n(Reset_n),
        .Led(Led)
    );

    // 时钟
    initial Clk = 1;
    always #10 Clk = ~Clk;  // 10ns反转一次

    // 激励
    initial begin
        // 重置
        Reset_n = 0;
        #201;       // 错位1ns，方便查看波形
        Reset_n = 1;
        
        // 运行这么长时间查看LED的波形变化
        #2000_000_000;
        
        $stop(0);
    end

endmodule

```



#### 跑马灯

**功能描述**
通过时钟信号控制开发板上的8个LED灯以1s的间隔流动。

```verilog
// LED_run.v
module Led_run(
    Clk,
    Reset_n,
    Led
    );

    input Clk;
    input Reset_n;
    output reg [7:0]Led;   // 8位的端口

    reg [24:0]counter; // 25位计数

    always @(posedge Clk or negedge Reset_n) 
    if(!Reset_n)
        counter <= 0;
    else if(counter == 25_000_000 - 1)   // 缩小1000倍，减小仿真时间
        counter <= 0;
    else    
        counter <= counter + 1'd1;

    // 考虑D触发器的实际物理结构，不能写成下面的样子
    // if((!Reset_n) | (counter == 25_000_000 -1))
    // 复位逻辑对应D触发器专门的复位端口，需要专门写出来，不得与其他条件合并

    always @(posedge Clk or negedge Reset_n)
    if(!Reset_n)
        Led <= 8'b0000_0001;    // 复位
    else if(counter == 25_000_000 - 1)begin
        if(Led == 8'b0000_0000)     // 判断是否溢出
            Led <= 8'b0000_0001;
        else
            Led <= Led << 1;        // 左移一位
    end
        

endmodule

```

```verilog
// LED_run_tb.v
`timescale 1ns / 1ns

module Led_run_tb();

    reg Clk;
    reg Reset_n;
    wire [7:0]Led;

    Led_run Led_run(
        .Clk(Clk),
        .Reset_n(Reset_n),
        .Led(Led)
    );

    initial Clk = 1;
    always #10 Clk = ~Clk;

    initial begin
        Reset_n = 0;
        #201;
        Reset_n = 1;
        #2000_000_000;  // 延迟 2s
        #2000_000_000;  // 延迟 2s   直接写延迟4s会溢出
        //#40_000_000;  // 缩小100倍，对应10轮Led
        $stop(0);
    end

endmodule

```

```verilog
# LED_run.xdc

#For AC7Z015

# 引脚定义
set_property PACKAGE_PIN L5 [get_ports Clk]
set_property PACKAGE_PIN R4 [get_ports Reset_n]

# 电平标准
set_property IOSTANDARD LVCMOS33 [get_ports Clk]
set_property IOSTANDARD LVCMOS33 [get_ports Reset_n]

# 引脚定义
set_property PACKAGE_PIN AB14 [get_ports Led[0]]
set_property PACKAGE_PIN AA14 [get_ports Led[1]]
set_property PACKAGE_PIN AA15 [get_ports Led[2]]
set_property PACKAGE_PIN AA12 [get_ports Led[3]]
set_property PACKAGE_PIN R17 [get_ports Led[4]]
set_property PACKAGE_PIN T17 [get_ports Led[5]]
set_property PACKAGE_PIN U19 [get_ports Led[6]]
set_property PACKAGE_PIN V19 [get_ports Led[7]]

# 电平标准
set_property IOSTANDARD LVCMOS33 [get_ports Led[0]]
set_property IOSTANDARD LVCMOS33 [get_ports Led[1]]
set_property IOSTANDARD LVCMOS33 [get_ports Led[2]]
set_property IOSTANDARD LVCMOS33 [get_ports Led[3]]
set_property IOSTANDARD LVCMOS33 [get_ports Led[4]]
set_property IOSTANDARD LVCMOS33 [get_ports Led[5]]
set_property IOSTANDARD LVCMOS33 [get_ports Led[6]]
set_property IOSTANDARD LVCMOS33 [get_ports Led[7]]
```

#### LED计数器

**功能描述**
使用开发板上的8个LED灯实现二进制计数，每秒加1。

```verilog
// bin_counter.v

`timescale 1ns / 1ps

// 1s计数器
module bin_counter(
    Clk,
    Reset_n,
    LED_Counter
    );

    input Clk;
    input Reset_n;
    output reg [7:0] LED_Counter;
	
    // 时钟计数器最大值，
    localparam MCNT_SEC = 50_000_000 - 1;

    // 时钟计数器
    reg [29:0] Clk_sec;
    
    // LED计数器，内部使用
    reg [7:0] counter;

    // 时钟计数器时序逻辑
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n)
            Clk_sec <= 0;
        else  if (Clk_sec == MCNT_SEC)
            Clk_sec <= 0;
        else
            Clk_sec <= Clk_sec + 1'd1;
    end

    // LED计数器时序逻辑
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n)
            counter <= 0;
        else if(Clk_sec == MCNT_SEC)
            counter <= counter + 1'd1;
    end

    // 将计数结果赋值到输出引脚
    // 此处@(*) 是组合逻辑，所以使用 =
    always @(*)
        LED_Counter = counter;

endmodule

```

```verilog
# bin_counter.sdc
# XC7Z015

# 引脚定义
set_property PACKAGE_PIN L5 [get_ports Clk]
set_property PACKAGE_PIN R4 [get_ports Reset_n]

# 电平标准
set_property IOSTANDARD LVCMOS33 [get_ports Clk]
set_property IOSTANDARD LVCMOS33 [get_ports Reset_n]

# 引脚定义
set_property PACKAGE_PIN AB14 [get_ports {LED_Counter[0]}]
set_property PACKAGE_PIN AA14 [get_ports {LED_Counter[1]}]
set_property PACKAGE_PIN AA15 [get_ports {LED_Counter[2]}]
set_property PACKAGE_PIN AA12 [get_ports {LED_Counter[3]}]
set_property PACKAGE_PIN R17 [get_ports {LED_Counter[4]}]
set_property PACKAGE_PIN T17 [get_ports {LED_Counter[5]}]
set_property PACKAGE_PIN U19 [get_ports {LED_Counter[6]}]
set_property PACKAGE_PIN V19 [get_ports {LED_Counter[7]}]

# 电平标准
set_property IOSTANDARD LVCMOS33 [get_ports {LED_Counter[0]}]
set_property IOSTANDARD LVCMOS33 [get_ports {LED_Counter[1]}]
set_property IOSTANDARD LVCMOS33 [get_ports {LED_Counter[2]}]
set_property IOSTANDARD LVCMOS33 [get_ports {LED_Counter[3]}]
set_property IOSTANDARD LVCMOS33 [get_ports {LED_Counter[4]}]
set_property IOSTANDARD LVCMOS33 [get_ports {LED_Counter[5]}]
set_property IOSTANDARD LVCMOS33 [get_ports {LED_Counter[6]}]
set_property IOSTANDARD LVCMOS33 [get_ports {LED_Counter[7]}]
```

#### 串口通信

##### 串口发送

**功能描述**
实现四种波特率可选 数据位8 停止位1 校验位none的串口发送。

```verilog
// uart_tx.v
`timescale 1ns/1ps

// 选择波特率，串口数据发送模块，默认 数据位8 停止位1 校验位none

// 例化示例代码
// reg Clk;
// reg Reset_n;
// reg [2:0] Baud_sel;
// reg Send_en;
// reg [7:0] Data;
// wire uart_tx;
// wire tx_done;
// wire uart_state;

// uart_tx your_inst_name(
//     .Clk(Clk),
//     .Reset_n(Reset_n),
//     .Baud_sel(Baud_sel),
//     .Send_en(Send_en),
//     .Data_byte(Data_byte),
//     .uart_tx(uart_tx),
//     .tx_done(tx_done),
//     .uart_state(uart_state)
// );

module uart_tx (
    Clk,			// 时钟输入
    Reset_n,		// 重置按钮
    Baud_sel,       // 波特率选择
    Send_en,        // 高电平触发一次发送
    Data_byte,		// 要发送的数据

    uart_tx,        // 数据发送，默认为高电平
    tx_done,        // 发送完成时输出一个时钟周期的高脉冲
    uart_state      // 0: 空闲  1: 发送中
);

    input Clk;
    input Reset_n;
    input [2:0]Baud_sel;
    input Send_en;
    input [7:0] Data_byte;

    output reg uart_tx;
    output reg tx_done;
    output reg uart_state;

    // 时钟频率  50MHz晶振
    parameter CLOCK_FREQ = 50_000_000;

    // 波特率计数最大值
    reg [15:0] baud_cnt_max;
    
    // 组合逻辑，使用 =
    // 根据选择波特率，修改对应波特率的时钟计数最大值
    always @(*) begin
        case (Baud_sel)
            3'd0:   baud_cnt_max = CLOCK_FREQ / 9600 - 1;
            3'd1:   baud_cnt_max = CLOCK_FREQ / 19200 - 1;
            3'd2:   baud_cnt_max = CLOCK_FREQ / 38400 - 1;
            3'd3:   baud_cnt_max = CLOCK_FREQ / 57600 - 1;
            3'd4:   baud_cnt_max = CLOCK_FREQ / 115200 - 1;
            default: baud_cnt_max = CLOCK_FREQ / 9600 - 1;
        endcase
    end

    // 状态机标识定义
    localparam IDLE = 2'b00;        // 空闲
    localparam START = 2'b01;       // 起始位
    localparam DATA = 2'b10;        // 数据位
    localparam STOP = 2'b11;        // 停止位

    // 当前状态
    reg [1:0] state; 
    // 波特率分频计数器
    reg [15:0] baud_cnt;
    // 数据位计数器
    reg [3:0] bit_cnt;
    // 数据锁存器，待发送的数据缓存
    reg [7:0] data_reg;

    // 波特率分频完成标志
    wire baud_done = (baud_cnt == baud_cnt_max);

    // 波特率计数器
    // 波特率计数器在非空闲状态时计数，计数到波特率最大值时重置为0
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n)
            baud_cnt <= 16'd0;
        else if(state == IDLE)      
            baud_cnt <= 16'd0;      // 空闲时清0
        else if(baud_cnt == baud_cnt_max)
            baud_cnt <= 16'd0;      // 计数到最大值，清0
        else
            baud_cnt <= baud_cnt + 1'b1;
    end

    // 数据位计数器
    // 在接收数据状态时计数，且波特率计数器达到时 +1，加到最大值时归零
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n)
            bit_cnt <= 4'd0;
        else if(state == IDLE)
            bit_cnt <= 4'd0;
        else if(state == DATA && baud_done) begin
            if(bit_cnt == 4'd7)
                bit_cnt <= 4'd0;
            else
                bit_cnt <= bit_cnt + 1'b1;
        end
    end

    // 主状态机
    // 空闲状态时等待发送使能信号，等到后进入发送起始位状态
    // 发送停止位时等待一个波特率计数周期，完成起始位发送，之后进入数据发送
    // 数据发送状态等到位计数器到达最大值且波特率周期完成，表示数据发送完成，进入停止位状态
    // 停止位等待一个波特率计数周期，停止位发送完成，回到空闲状态
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n)
            state <= IDLE;
        else begin
            case(state)
                IDLE: begin
                    if(Send_en)
                        state <= START;
                end
                START: begin
                    if(baud_done)
                        state <= DATA;
                end
                DATA: begin
                    if(baud_done &&(bit_cnt == 4'd7))
                        state <= STOP;
                end
                STOP: begin
                    if(baud_done)
                        state <= IDLE;
                end
                default: state <= state;
            endcase
        end
    end
    
    // 输出逻辑
    // 重置时发送为高电平，发送完成信号，共状态和数据锁存器都清0
    // 工作时根据当前状态标识修改输出引脚的状态
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n) begin
            uart_tx <= 1'b1;   // 空闲时为高电平
            tx_done <= 1'b0;
            uart_state <= 1'b0; // 空闲
            data_reg <= 8'd0;
        end
        else begin 
            case(state)
                IDLE: begin
                    uart_tx <= 1'b1;
                    tx_done <= 1'b0;
                    uart_state <= 1'b0;
                    if(Send_en) begin
                        data_reg <= Data_byte;      // 锁存待发送的数据
                        uart_state <= 1'b1;         // 进入忙状态
                    end
                end
                
                START: begin
                    uart_tx <= 1'b0;    // 发送起始位
                    tx_done <= 1'b0;
                    uart_state <= 1'b1;
                end
                
                DATA: begin
                    uart_tx <= data_reg[bit_cnt];   // LSB先发
                    tx_done <= 1'b0;
                    uart_state <= 1'b1;
                end
                
                STOP: begin
                    uart_tx <= 1'b1;    // 发送停止位
                    uart_state <= 1'b1;
                    if(baud_done)
                        tx_done <= 1'b1;
                    else
                        tx_done <= 1'b0;
                end
                
                default: begin
                    uart_tx <= 1'b1;
                    tx_done <= 1'b0;
                    uart_state <= 1'b0;
                end
            endcase
        end
    end

endmodule

```

```verilog
// uart_tx_tb.v
`timescale 1ns / 1ps

module uart_tx_tb();

    // 定义信号
    reg Clk;
    reg Reset_n;
    reg [2:0] Baud_sel;
    reg Send_en;
    reg [7:0] Data_byte;
    wire uart_tx;
    wire tx_done;
    wire uart_state;

    // 例化
    uart_tx uart_tx_inst_0(
        .Clk(Clk),
        .Reset_n(Reset_n),
        .Baud_sel(Baud_sel),
        .Send_en(Send_en),
        .Data_byte(Data_byte),
        .uart_tx(uart_tx),
        .tx_done(tx_done),
        .uart_state(uart_state)
    );
	
    // 初始化时钟
    initial Clk = 0;
    always #10 Clk = ~Clk;

    // 激励
    initial begin
        // 初始化，先进入重置状态，清空要输入的信息
        Reset_n = 0;
        Baud_sel = 3'd4;        // 用115200
        Send_en = 0;
        Data_byte = 8'h00;

        // 等待内部状态稳定，关闭重置
        #200;
        Reset_n = 1;
        #200;

        // 等待时钟上升沿，发送0x55
        @(posedge Clk);
        Data_byte = 8'h55;
        Send_en = 1;
        // 等待时钟上升沿关闭发送使能，发送使能持续了一个时钟周期
        @(posedge Clk);
        Send_en = 0;

        // 等发完
        #(200_000);

        // 发送 0xA5
        @(posedge Clk);
        Data_byte = 8'hA5;
        Send_en = 1;
        @(posedge Clk);
        Send_en = 0;
		
        #(200_000);
        
        // 结束仿真
        $finish();
    end
endmodule

```

##### 串口接收

**功能描述**
实现接收串口数据，波特率四种可选，数据位8 停止位1 校验位none

```verilog
// uart_rx.v
`timescale 1ns/1ps

// 选择波特率，串口数据发送模块，默认 数据位8 停止位1 校验位none

// 例化示例代码
// reg Clk;
// reg Reset_n;
// reg [2:0] Baud_sel;
// reg Send_en;
// reg [7:0] Data;
// wire uart_tx;
// wire tx_done;
// wire uart_state;

// uart_tx your_inst_name(
//     .Clk(Clk),
//     .Reset_n(Reset_n),
//     .Baud_sel(Baud_sel),
//     .Send_en(Send_en),
//     .Data_byte(Data_byte),
//     .uart_tx(uart_tx),
//     .tx_done(tx_done),
//     .uart_state(uart_state)
// );

module uart_tx (
    Clk,			// 时钟输入
    Reset_n,		// 重置
    Baud_sel,       // 波特率选择
    Send_en,        // 高电平触发一次发送
    Data_byte,

    uart_tx,        // 数据发送，默认为高电平
    tx_done,        // 发送完成时输出一个时钟周期的高脉冲
    uart_state      // 0: 空闲  1: 发送中
);

    input Clk;
    input Reset_n;
    input [2:0]Baud_sel;
    input Send_en;
    input [7:0] Data_byte;

    output reg uart_tx;
    output reg tx_done;
    output reg uart_state;

    // 时钟频率
    parameter CLOCK_FREQ = 50_000_000;

    // 波特率选择器
    reg [15:0] baud_cnt_max;
    
    always @(*) begin
        case (Baud_sel)
            3'd0:   baud_cnt_max = CLOCK_FREQ / 9600 - 1;
            3'd1:   baud_cnt_max = CLOCK_FREQ / 19200 - 1;
            3'd2:   baud_cnt_max = CLOCK_FREQ / 38400 - 1;
            3'd3:   baud_cnt_max = CLOCK_FREQ / 57600 - 1;
            3'd4:   baud_cnt_max = CLOCK_FREQ / 115200 - 1;
            default: baud_cnt_max = CLOCK_FREQ / 9600 - 1;
        endcase
    end

    // 状态机
    localparam IDLE = 2'b00;        // 空闲
    localparam START = 2'b01;       // 起始位
    localparam DATA = 2'b10;        // 数据位
    localparam STOP = 2'b11;        // 停止位

    // 状态
    reg [1:0] state; 
    // 波特率分频计数器
    reg [15:0] baud_cnt;
    // 数据位计数器
    reg [3:0] bit_cnt;
    // 待发送的数据缓存
    reg [7:0] data_reg;

    // 波特率分频完成标志
    wire baud_done = (baud_cnt == baud_cnt_max);

    // 波特率计数器
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n)
            baud_cnt <= 16'd0;
        else if(state == IDLE)      
            baud_cnt <= 16'd0;      // 空闲时清0
        else if(baud_cnt == baud_cnt_max)
            baud_cnt <= 16'd0;      // 计数到最大值，清0
        else
            baud_cnt <= baud_cnt + 1'b1;
    end

    // 数据位计数器
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n)
            bit_cnt <= 4'd0;
        else if(state == IDLE)
            bit_cnt <= 4'd0;
        else if(state == DATA && baud_done) begin
            if(bit_cnt == 4'd7)
                bit_cnt <= 4'd0;
            else
                bit_cnt <= bit_cnt + 1'b1;
        end
    end

    // 主状态机
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n)
            state <= IDLE;
        else begin
            case(state)
                IDLE: begin
                    if(Send_en)
                        state <= START;
                end
                START: begin
                    if(baud_done)
                        state <= DATA;
                end
                DATA: begin
                    if(baud_done &&(bit_cnt == 4'd7))
                        state <= STOP;
                end
                STOP: begin
                    if(baud_done)
                        state <= IDLE;
                end
                default: state <= state;
            endcase
        end
    end
    
    // 输出逻辑
    always @(posedge Clk or negedge Reset_n) begin
        if(!Reset_n) begin
            uart_tx <= 1'b1;   // 空闲时为高电平
            tx_done <= 1'b0;
            uart_state <= 1'b0; // 空闲
            data_reg <= 8'd0;
        end
        else begin 
            case(state)
                IDLE: begin
                    uart_tx <= 1'b1;
                    tx_done <= 1'b0;
                    uart_state <= 1'b0;
                    if(Send_en) begin
                        data_reg <= Data_byte;      // 锁存待发送的数据
                        uart_state <= 1'b1;         // 进入忙状态
                    end
                end
                START: begin
                    uart_tx <= 1'b0;    // 发送起始位
                    tx_done <= 1'b0;
                    uart_state <= 1'b1;
                end
                DATA: begin
                    uart_tx <= data_reg[bit_cnt];   // LSB先发
                    tx_done <= 1'b0;
                    uart_state <= 1'b1;
                end
                STOP: begin
                    uart_tx <= 1'b1;    // 发送停止位
                    uart_state <= 1'b1;
                    if(baud_done)
                        tx_done <= 1'b1;
                    else
                        tx_done <= 1'b0;
                end
                default: begin
                    uart_tx <= 1'b1;
                    tx_done <= 1'b0;
                    uart_state <= 1'b0;
                end
            endcase
        end
    end
endmodule

```

```verilog
// uart_rx_tb.v
`timescale 1ns / 1ps


module usrt_rx_tb();

    reg Clk;
    reg Reset_n;
    reg [2:0] Baud_sel;
    reg uart_rx;
    
    wire [7:0] Data_out;
    wire rx_done;
    wire uart_state;

    uart_rx uart_rx_inst_0(
        .Clk(Clk),
        .Reset_n(Reset_n),
        .Baud_sel(Baud_sel),
        .uart_rx(uart_rx),

        .Data_out(Data_out),
        .rx_done(rx_done),
        .uart_state(uart_state)
    );

    // 50MHz 时钟
    initial Clk = 0;
    always #10 Clk = ~Clk;

    // 115200bps 位周期约 8680ns 取整8680
    parameter BIT_TIME = 8680;

    // 模拟发送一个字节
    task send_byte;
        input [7:0] data;
        integer i;
        begin
            // 起始位
            uart_rx = 1'b0;
            #(BIT_TIME);
            // 8位数据， LSB先
            for(i = 0; i < 8; i = i + 1) begin
                uart_rx = data[i];
                #(BIT_TIME);
            end

            // 停止位
            uart_rx = 1'b1;
            #(BIT_TIME);
        end
    endtask

    // 激励
    initial begin
        // 初始化
        Reset_n = 0;
        Baud_sel = 3'd4;    // 波特率选择115200
        uart_rx = 1'b1;     // 空闲时为高电平

        #200;       // 等待状态稳定
        Reset_n = 1;
        #200;

        // 发送第一个字节 0x55
        send_byte(8'h55);
        #(BIT_TIME * 2);

        // 发送第二个字节 0xAA
        send_byte(8'hAA);
        #(BIT_TIME * 2);

        // 发送第三个字节 0xF0a
        send_byte(8'hF0);
        #(BIT_TIME * 2);

        #500;

        $finish();
    end
endmodule

```

##### 串口收发测试

上面的模块只进行了仿真测试，这一步编写一个模块调用串口发送和接收模块，做板级验证。
这里实现一个回环，收到什么就向外转发什么。同时用8个LED灯表示收到的内容。





















