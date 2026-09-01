# FPGA_study

学习FPGA笔记



### 基础

#### mux2 二路选择器

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

    // 激励信号
    initial Clk = 1;
    always #10 Clk = ~Clk;  // 10ns反转一次

    initial begin
        Reset_n = 0;
        #201;       // 错位1ns，方便查看波形
        Reset_n = 1;
        #2000_000_000;
        $stop(0);
    end

endmodule

```



#### 跑马灯

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



