# Verilog_Practice
在[HDLBits](https://hdlbits.01xz.net/wiki/Main_Page)上学习VerilogHDL



暂时没找到好用的HDL编辑器之前先先用MarkDown来写了

[Verilog 2001标准pdf下载地址](https://sutherland-hdl.com/pdfs/verilog_2001_ref_guide.pdf)

# Verilog Language



## Basics



### Simple wire

连续赋值`assign left_side = right_side;`操作本质上是将一根输入线连续到输出上, 这意味着如果输入部分改变输出也会跟着变化。

这也意味着这种赋值是有方向性的。

除了连续赋值还有其他三种赋值方式。



### Four wires

对于连续赋值而言, 赋值的顺序并不影响。因为本质上是电路的一种连接, 这种行为和一般编程语言的赋值是有差异的。因为实际上没有copy的操作发生。

另外, 虽然assign是一种连接的行为, 但是这种行为并不是连线(不是产生wire)。线(wire)本身已经由`input`或者`output`语句声明。`input wire a`和`input a`是同一个意思。



### Inverter

实现一个非门。还是要理解连续赋值, continuously的含义。当输入改变时输出会立即改变。而不像编程语言的NOT那样其实是寄存器进行了有顺序的操作。

Hint: Verilog has separate bitwise-NOT (`~`) and logical-NOT (`!`) operators, like C



### AND gate

Hint: Verilog has separate bitwise-AND (`&`) and logical-AND (`&&`) operators, like C.



### NOR gate

`assign`一次会驱动一个wire或称为net.

Hint: Verilog has separate bitwise-OR (`|`) and logical-OR (`||`) operators, like C.



### XNOR gate

Hint: The bitwise-XOR operator is `^`. There is no logical-XOR operator.



### Declaring wires

```verilog
module top_module (
	input a,
	input b,
	input c,
	input d,
	output out,
	output out_n );
	
	wire w1, w2;		// Declare two wires (named w1 and w2)
	assign w1 = a&b;	// First AND gate
	assign w2 = c&d;	// Second AND gate
	assign out = w1|w2;	// OR gate: Feeds both 'out' and the NOT gate

	assign out_n = ~out;	// NOT gate
	
endmodule
```

实例程序, 注意用了feeds一词



### 7458 chip



## Vectors



### Vectors

在Verilog 2001手册中, 提到了

- range (optional) is a range from [msb :lsb] (most-significant-bit to leastsignificant-bit). 
  - If no range is specified, ports are 1-bit wide. 
  - The msb and lsb must be a literal number, a constant, an expression, or a call to a constant function.
  - Either little-endian convention (the lsb is the smallest bit number) or bigendian convention (the lsb is the largest bit number) may be used.
  - ...

示例代码中提供了一种写法:

```verilog
module top_module(
	input [2:0] vec, 
	output [2:0] outv,
	output o2,
	output o1,
	output o0
);
	
	assign outv = vec;

	// This is ok too: assign {o2, o1, o0} = vec;
	assign o0 = vec[0];
	assign o1 = vec[1];
	assign o2 = vec[2];
	
endmodule
```



### Vectors in more details

- 向量有多种声明方式:

```verilog
wire [7:0] w;         // 8-bit wire
reg  [4:1] x;         // 4-bit reg
output reg [0:0] y;   // 1-bit reg that is also an output port (this is still a vector)
input wire [3:-2] z;  // 6-bit wire input (negative ranges are allowed)
output [3:0] a;       // 4-bit output wire. Type is 'wire' unless specified otherwise.
wire [0:7] b;         // 8-bit wire where b[0] is the most-significant bit.
```

但是, 在语言层面上:

```verilog
type [upper:lower] vector_name;
```



- 注意隐式网络Implicit nets:

```ver
wire [2:0] a, c;   // Two vectors
assign a = 3'b101;  // a = 101
assign b = a;       // b =   1  implicitly-created wire
assign c = b;       // c = 001  <-- bug
my_module i1 (d,e); // d and e are implicitly one-bit wide if not declared.
                    // This could be a bug if the port was intended to be a vector.
```

Adding ``default_nettype none` would make the second line of code an error, which makes the bug more visible.

加上这一句会让这种语句报错。



- 切片

  ```
  assign w = a;
  ```

  takes the entire 4-bit vector *a* and assigns it to the entire 8-bit vector *w* (declarations are taken from above). If the lengths of the right and left sides don't match, it is zero-extended or truncated as appropriate.

下面两句我其实没看懂:

为什么[0:3]是错误的, 但后面赋值却可以那样做?

手册20面:

 The order of the part select must be consistent with the declaration of the vector (e.g. if the lsb is the the lowest bit number in the declaration, then the lsb of the part select must also be the lowest bit number).

```verilog
b[3:0]      // Illegal. Vector part-select must match the direction of the declaration.
b[0:3]      // The *upper* 4 bits of b.
assign w[3:0] = b[0:3];    // Assign upper 4 bits of b to lower 4 bits of w. w[3]=b[0], w[2]=b[1], etc.
```

我试尝试了新的写法:

```verilog
`default_nettype none     // Disable implicit nets. Reduces some types of bugs.
module top_module( 
    input wire [15:0] in,
    output wire [7:0] out_hi,
    output wire [7:0] out_lo );
    
    assign { out_hi, out_lo } = in;

endmodule
```



### vector part select

我写的是:

之前报错是因为我把16写成了26

```verilog
module top_module( 
    input [31:0] in,
    output [31:0] out );//

    // assign out[31:24] = ...;
  assign {out[7:0], out[15:8], out[23:16],out[31:24]} = in;

endmodule
```



### Bitwise operators

我觉得我的代码更好

```verilog
module top_module( 
    input [2:0] a,
    input [2:0] b,
    output [2:0] out_or_bitwise,
    output out_or_logical,
    output [5:0] out_not
);
    
    assign out_or_bitwise = a | b;
    assign out_or_logical = a || b;
    assign out_not = ~ {b, a};

endmodule
```

不过暂时不用担心学不到特性, 看了一下后面练习会讲



### Four-input gates

不能这样写:

```verilog
module top_module( 
    input [3:0] in,
    output out_and,
    output out_or,
    output out_xor
    
);
    
    and(out_and, in);
    or(out_or, in);
    xor(out_xor, in);

endmodule
```

也不能:

```verilog
module top_module( 
    input [3:0] in,
    output out_and,
    output out_or,
    output out_xor
    
);
    
    and(out_and, in[3:0]);
    or(out_or, in[3:0]);
    xor(out_xor, in[3:0]);

endmodule
```



### Vector concatenation operator

不能这样写:

```verilog
module top_module (
    input [4:0] a, b, c, d, e, f,
    output [7:0] w, x, y, z );//

    // assign { ... } = { ... };
    assign { w, x, y, z }[31: 2] = { a, b, c, d, e, f };
    assign { w, x, y, z }[1: 0] = 2'b11;

endmodule
```



### Vector reversal 1

Hint: `assign out[7:0] = in[0:7];` does not work because Verilog does not allow vector bit ordering to be flipped.

有关标答中的注释:

```verilog
	/*
	// I know you're dying to know how to use a loop to do this:

	// Create a combinational always block. This creates combinational logic that computes the same result
	// as sequential code. for-loops describe circuit *behaviour*, not *structure*, so they can only be used 
	// inside procedural blocks (e.g., always block).
	// The circuit created (wires and gates) does NOT do any iteration: It only produces the same result
	// AS IF the iteration occurred. In reality, a logic synthesizer will do the iteration at compile time to
	// figure out what circuit to produce. (In contrast, a Verilog simulator will execute the loop sequentially
	// during simulation.)
	always @(*) begin	
		for (int i=0; i<8; i++)	// int is a SystemVerilog type. Use integer for pure Verilog.
			out[i] = in[8-i-1];
	end


	// It is also possible to do this with a generate-for loop. Generate loops look like procedural for loops,
	// but are quite different in concept, and not easy to understand. Generate loops are used to make instantiations
	// of "things" (Unlike procedural loops, it doesn't describe actions). These "things" are assign statements,
	// module instantiations, net/variable declarations, and procedural blocks (things you can create when NOT inside 
	// a procedure). Generate loops (and genvars) are evaluated entirely at compile time. You can think of generate
	// blocks as a form of preprocessing to generate more code, which is then run though the logic synthesizer.
	// In the example below, the generate-for loop first creates 8 assign statements at compile time, which is then
	// synthesized.
	// Note that because of its intended usage (generating code at compile time), there are some restrictions
	// on how you use them. Examples: 1. Quartus requires a generate-for loop to have a named begin-end block
	// attached (in this example, named "my_block_name"). 2. Inside the loop body, genvars are read only.
	generate
		genvar i;
		for (i=0; i<8; i = i+1) begin: my_block_name
			assign out[i] = in[8-i-1];
		end
	endgenerate
	*/
```



### Replication operator

`{num{vector}}`



### More replication

挺简单的



## Modules: Hierarchy

Hierarchy: 层次结构



### Modules

- The code for one module is not written *inside* another module's body (Code for different modules are not nested).

- 连接模块有两种方式: By position 或者 By name。

  以位置而言和C语言一样, 只要按照C语言调用函数的方式来就好了`mod_a instance1 ( wa, wb, wc );`

  以名字而言则是可以用名字来指定, 例如:`mod_a instance2 ( .out(wc), .in1(wa), .in2(wb) );`

  注意后者要打句点



### Connecting ports by position

练习题而已



### Connecting ports by name

练习题而已



### Three modules

多个实例的练习



### Modules and vectors

Like everywhere else in Verilog, the vector length of the port does not have to match the wire connecting to it, but this will cause zero-padding or trucation of the vector. 

有关always块自学了手册的23页, 其中提到了initial和always, 还有begin-end与fork-join两种

begin-end的时间跟上一条语句的时间有关, 而fork-join则全是以绝对开始的时间为参考

我写的:

```verilog
module top_module ( 
    input clk, 
    input [7:0] d, 
    input [1:0] sel, 
    output [7:0] q 
);
    
    wire [7:0]a,b,c;
    my_dff8 ins1(clk, d, a);
    my_dff8 ins2(clk, a, b);
    my_dff8 ins3(clk, b, c);
    chos ins4(sel, d, a, b, c, q);

endmodule

module chos(
    input [1:0]sel,
    input [7:0]in0,
    input [7:0]in1,
    input [7:0]in2,
    input [7:0]in3,
    output [7:0]out
);
    
    always @(*) begin
        case(sel)
            2'b00: out = in0;
            2'b01: out = in1;
            2'b10: out = in2;
            2'b11: out = in3;
        endcase
    end

endmodule
```

标答:

```verilog
module top_module (
	input clk,
	input [7:0] d,
	input [1:0] sel,
	output reg [7:0] q
);

	wire [7:0] o1, o2, o3;		// output of each my_dff8
	
	// Instantiate three my_dff8s
	my_dff8 d1 ( clk, d, o1 );
	my_dff8 d2 ( clk, o1, o2 );
	my_dff8 d3 ( clk, o2, o3 );

	// This is one way to make a 4-to-1 multiplexer
	always @(*)		// Combinational always block
		case(sel)
			2'h0: q = d;
			2'h1: q = o1;
			2'h2: q = o2;
			2'h3: q = o3;
		endcase

endmodule
```

所以说明这种情况可以省略begin-end之类, 也不用在外部单独声明module.



### Adder 1

简单的连线练习



### Adder 2

简单的连线练习



### Carry-select adder

带选择的练习



### Adder-subtractor

异或b的时候, 可以`assign b1 = b ^ {32{sub}};`,  不能是`assign b1 = b ^ sub;`多余的位数会被0扩展。



## Procedures
Procedures include always, initial, task, and function blocks. Procedures allow sequential statements (which cannot be used outside of a procedure) to be used to describe the behaviour of a circuit.



### Always blocks (combinational)

- combinational与clocked分别代表组合组合逻辑和时序逻辑。

- **The syntax for code inside a procedural block is different from code that is outside.**

- 以下语句具有相同的结果(Combinational always blocks are equivalent to assign statements):

```verilog
assign out1 = a & b | c ^ d;
always @(*) out2 = a & b | c ^ d;
```

assign语句必须是net类型, 例如wire。而always块中则是variable类型, 例如reg。

- 但其实: These types (wire vs. reg) have nothing to do with what hardware is synthesized, and is just syntax left over from Verilog's use as a hardware *simulation* language.



### Always blocks (clocked)

- Clocked always blocks create a blob of combinational logic just like combinational always blocks, but also creates a set of flip-flops (or "registers") at the output of the blob of combinational logic. Instead of the outputs of the blob of logic being visible immediately, the outputs are visible only immediately after the next (posedge clk).

- assign语句不能用在procedure之中, 直接的`=`和`<=`

- In a **combinational** always block, use **blocking** assignments. In a **clocked** always block, use **non-blocking** assignments. 

- 虽然教程中有这样的说明, 但是从绝对verilog语法而言, `=`和`<=`都能用于Procedural Assignment Statements中, 但是手册25页中也明确的说明:

  *MODELING TIP*: To avoid potential simulation race conditions in zero-delay models:

  - Use blocking assignments ( = ) to model combinational logic.

  - Use non-blocking assignments ( <= ) to model sequential logic.

- 此外, 可以指定是posedge上升沿和negedge下降沿。





### If statement

```verilog
always @(*) begin
    if (condition) begin
        out = x;
    end
    else begin
        out = y;
    end
end
```

This is equivalent to using a continuous `assign`ment with a conditional operator:

```verilog
assign out = (condition) ? x : y;
```

注意: However, the procedural `if` statement provides a new way to make mistakes. The circuit is combinational only if `out` is always assigned a value.

我的代码:

```verilog
// synthesis verilog_input_version verilog_2001
module top_module(
    input a,
    input b,
    input sel_b1,
    input sel_b2,
    output wire out_assign,
    output reg out_always   ); 
    
    assign out_assign = (sel_b1 && sel_b2) ? b : a;
    always @(*)
        if (sel_b1 && sel_b2) out_always = b;
    	else out_always = a;

endmodule
```



### If statement latches

一些避免错误的方法:

- When designing circuits, you *must* think first in terms of circuits:

  - I want this logic gate

  - I want a *combinational* blob of logic that has these inputs and produces these outputs

  - I want a combinational blob of logic followed by a set of flip-flops

- What you *must not* do is write the code first, then hope it generates a proper circuit.

  - If (cpu_overheated) then shut_off_computer = 1;

  - If (~arrived) then keep_driving = ~gas_tank_empty;

还是不能像一般的编程语言那样写HDL。

注意:

- Syntactically-correct code does not necessarily result in a reasonable circuit (combinational logic + flip-flops). The usual reason is: "What happens in the cases other than those you specified?". Verilog's answer is: Keep the outputs unchanged.

- This behaviour of "keep outputs unchanged" means the current state needs to be *remembered*, and thus produces a *latch*. Combinational logic (e.g., logic gates) cannot remember any state. Watch out for `Warning (10240): ... inferring latch(es)"` messages. Unless the latch was intentional, it almost always indicates a bug. Combinational circuits must have a value assigned to all outputs under all conditions. This usually means you always need `else` clauses or a default value assigned to the outputs.

其实示例中的问题其实还是围绕着每种情况都要赋值。



### Case statement

- 与C语言switch的区别:
  - The case statement begins with `case` and each "case item" ends with a colon. There is no "switch".
  - Each case item can execute *exactly one* statement. This makes the "break" used in C unnecessary. But this means that if you need more than one statement, you must use `begin ... end`.
  - Duplicate (and partially overlapping) case items are permitted. The first one that matches is used. C does not allow duplicate case items.



### Priority encoder

这题做得挺痛苦的, 之前看到有casez的用法, 但是不知道高阻是什么意思。 

然后又不能像C语言那样用。

最后我用Logisim的自动生成功能生成了一下真值表, 我的代码是:

```verilog
// synthesis verilog_input_version verilog_2001
module top_module (
    input [3:0] in,
    output reg [1:0] pos  );
    
    wire a,b,c,d;
    assign {a,b,c,d} = in;
    
    always @(*)
    case(in)
        default: begin
            pos[1] = b & ~c & ~d | a & ~c & ~d;
            pos[0] = c & ~d | a & ~b & ~d;
        end
    endcase
    
endmodule
```

标答是直接每个数都直接返回一个结果。



### Priority encoder with casez

高阻态百度的结果:

- 高阻态是一个数字电路里常见的术语，指的是电路的一种输出状态，既不是高电平也不是低电平，如果高阻态再输入下一级电路的话，对下级电路无任何影响，和没接一样，如果用万用表测的话有可能是高电平也有可能是低电平，随它后面接的东西定的。
- 在数字电路中，三态逻辑（英语：Three-state logic）允许输出端在0和1两种逻辑电平之外呈现高阻态，等效于将输出的影响从后级电路中移除。这允许多个电路共同使用同一个输出线（例如总线）。

z为高阻态, x为不定值:

- case是一一对应，即0、1、x、z分别对应0、1、x、z；当执行到对应项后，case就会退出
- casex是将高阻值（z）和不定值（x）都视为不关心的状态，即出现x或z会匹配任意0、1、x、z状态；
- casez是将高阻值（z）视为不关心的状态，即出现z会匹配任意0、1、x、z状态；

我其实觉得这道题更应该用不定值的casex:

```verilog
// synthesis verilog_input_version verilog_2001
module top_module (
    input [7:0] in,
    output reg [2:0] pos  );
    
    always @(*)
    casex(in)
        8'b???????1: pos = 0;
        8'b??????10: pos = 1;
        8'b?????100: pos = 2;
        8'b????1000: pos = 3;
        8'b???10000: pos = 4;
        8'b??100000: pos = 5;
        8'b?1000000: pos = 6;
        8'b10000000: pos = 7;
        default: pos = 0;
    endcase

endmodule
```



### Avoiding latches

防止出错的方法之一就是提前赋好"default value":

```verilog
always @(*) begin
    up = 1'b0; down = 1'b0; left = 1'b0; right = 1'b0;
    case (scancode)
        ... // Set to 1 as necessary.
    endcase
end
```

This style of code ensures the outputs are assigned a value (of 0) in all possible cases unless the case statement overrides the assignment. This also means that a `default:` case item becomes unnecessary.

**Reminder:** The logic synthesizer generates a combinational circuit that *behaves* equivalently to what the code describes. Hardware does not "execute" the lines of code in sequence.

并不是像编程语言那样有赋值的先后顺序, 只是在描述硬件的情况。



## More Verilog Features



### Conditional ternary operator

要求四行解决, 一行声明中间要用的wire, 4个数比较二分法三行。

第一次尝试编译出错, 因为wire的位宽声明错了。



### Reduction operators

如果对一个vector的每一位(其实是每一项, 因为vector可以潜逃声明)都做相同的操作, 可以简写:

(& d[7:0]), 还可以对结果再取反做出NAND, NOR, and XNOR gates, e.g., `(~& d[7:0])`.

这样[Four-input gates](### Four-input gates)



### Reduction: Even wider gates

练习而已



### Combinational for-loop: Vector reversal 2

for循环, 注意: The circuit created (wires and gates) does NOT do any iteration: It only produces the same result...



### Combinational for-loop: 255-bit population count

想了半天然后去提前看了答案, 然后发现自己担心多了。

本身for就是一种构造, 而且+操作能直接用。

```verilog
module top_module (
	input [254:0] in,
	output reg [7:0] out
);

	always @(*) begin	// Combinational always block
		out = 0;
		for (int i=0;i<255;i++)
			out = out + in[i];
	end
	
endmodule
```



### Generate for-loop: 100-bit binary adder 2

一般i=0的情况是单独的。

我的代码:

```verilog
module top_module( 
    input [99:0] a, b,
    input cin,
    output [99:0] cout,
    output [99:0] sum );
    
    
    add1 ins0(a[0], b[0], cin, sum[0], cout[0]);
    
    generate
       genvar i;
        for(i = 1; i < 100; i = i + 1) begin:addr_gen
            add1 ins(a[i], b[i], cout[i - 1], sum[i], cout[i]);
        end
    endgenerate

endmodule

module add1 ( input a, input b, input cin,   output sum, output cout );

	assign sum = a ^ b ^ cin;
	assign cout = a&b | a&cin | b&cin;

endmodule
```



### Generate for-loop: 100-digit BCD adder

我的代码:

```verilog
module top_module( 
    input [399:0] a, b,
    input cin,
    output cout,
    output [399:0] sum );
    
    wire [99:0]tmp;
    
    bcd_fadd ins(a[3:0], b[3:0], cin, tmp[0], sum[3: 0]);
    
    generate
        genvar i;
        for(i = 1; i < 100; i = i + 1)begin:bcd_adder
            bcd_fadd ins(a[i * 4 + 3:i * 4], b[i * 4 + 3:i * 4], tmp[i - 1], tmp[i], sum[i * 4 + 3:i * 4]);
        end
    endgenerate
    
    assign cout = tmp[99];
    
endmodule
```



# Circuits



##  Combinational Logic



### Basic Gates

没有笔记的都是太基础的



#### Wire

#### GND

#### NOR

#### Another gate
#### Two gates

#### More logic gates

#### 7420 chip

我的代码:

```verilog
module top_module ( 
    input p1a, p1b, p1c, p1d,
    output p1y,
    input p2a, p2b, p2c, p2d,
    output p2y );
    
    assign p1y = {~& {p1a, p1b, p1c, p1d}};
    assign p2y = {~& {p2a, p2b, p2c, p2d}};


endmodule
```

说明reduce(归约)功能其实可以不指明vector切片的范围。

```verilog
A = 4'b1010 ;
&A ;
```



#### Truth tables

积和范式:

One simple method to create a circuit that implements the truth table's function is to express the function in sum-of-products form. **Sum** (meaning OR) of **products** (meaning AND) means using one *N*-input AND gate per row of the truth table (to detect when the input matches each row), followed by an OR gate that chooses only those rows that result in a '1' output.



#### Two-bit equality

标答很简洁:

```verilog
module top_module(
	input [1:0] A,
	input [1:0] B,
	output z);

	assign z = (A[1:0]==B[1:0]);	// Comparisons produce a 1 or 0 result.
	
	// Another option is to use a 16-entry truth table ( {A,B} is 4 bits, with 16 combinations ).
	// There are 4 rows with a 1 result.  0000, 0101, 1010, and 1111.

endmodule
```



#### Simple circuit A
#### Simple circuit B
#### Combine circuits A and B

#### Ring or vibrate?

**Design hint:** When designing circuits, one often has to think of the problem "backwards", starting from the outputs then working backwards towards the inputs. This is often the opposite of how one would think about a (sequential, imperative) programming problem, where one would look at the inputs first then decide on an action (or output). For sequential programs, one would often think "If (inputs are ___ ) then (output should be ___ )". On the other hand, hardware designers often think "The (output should be ___ ) when (inputs are ___ )".

The above problem description is written in an imperative form suitable for software programming (*if ring then do this*), so you must convert it to a more declarative form suitable for hardware implementation (`*assign ringer = ___*`). Being able to think in, and translate between, both styles is one of the most important skills needed for hardware design.

我的解答和标答一样。



#### Thermostat

练习backwards



#### 3-bit population count

#### Gates and vectors

标答比我的更简答。

我的代码:

```verilog
module top_module( 
    input [3:0] in,
    output [2:0] out_both,
    output [3:1] out_any,
    output [3:0] out_different );
    
    assign out_both = {in[3] & in[2], in[2] & in[1], in[1] & in[0]};
    assign out_any = {in[3] | in[2], in[2] | in[1], in[1] | in[0]};
    assign out_different  = {in[0] ^ in[3], in[3] ^ in[2], in[2] ^ in[1], in[1] ^ in[0]};

endmodule
```

标答:

```verilog
module top_module (
	input [3:0] in,
	output [2:0] out_both,
	output [3:1] out_any,
	output [3:0] out_different
);

	// Use bitwise operators and part-select to do the entire calculation in one line of code
	// in[3:1] is this vector:   					 in[3]  in[2]  in[1]
	// in[2:0] is this vector:   					 in[2]  in[1]  in[0]
	// Bitwise-OR produces a 3 bit vector.			   |      |      |
	// Assign this 3-bit result to out_any[3:1]:	o_a[3] o_a[2] o_a[1]

	// Thus, each output bit is the OR of the input bit and its neighbour to the right:
	// e.g., out_any[1] = in[1] | in[0];	
	// Notice how this works even for long vectors.
	assign out_any = in[3:1] | in[2:0];

	assign out_both = in[2:0] & in[3:1];
	
	// XOR 'in' with a vector that is 'in' rotated to the right by 1 position: {in[0], in[3:1]}
	// The rotation is accomplished by using part selects[] and the concatenation operator{}.
	assign out_different = in ^ {in[0], in[3:1]};
	
endmodule
```



#### Even longer vectors

上一题技巧的运用。



### Multiplexers

#### 2-to-1 multiplexer

#### 2-to-1 bus multiplexer

标答里提了一个问题:

```verilog
module top_module (
	input [99:0] a,
	input [99:0] b,
	input sel,
	output [99:0] out
);

	assign out = sel ? b : a;
	
	// The following doesn't work. Why?
	// assign out = (sel & b) | (~sel & a);
	
endmodule
```

上一题可以用这个写法, 这一题却不行, 原因很简单, 位宽不匹配, 运算时sel的高位自动扩展0。



#### 9-to-1 multiplexer

我的代码:

```verilog
module top_module( 
    input [15:0] a, b, c, d, e, f, g, h, i,
    input [3:0] sel,
    output [15:0] out );
    
    always @(*)
        case(sel)
            0:out = a;
            1:out = b;
            2:out = c;
            3:out = d;
            4:out = e;
                5:out = f;
                6:out = g;
                7:out = h;
                8:out = i;
            default: out = {16{1'b1}};
        endcase

endmodule
```

标答代码更符合我的心意:

```verilog
module top_module (
	input [15:0] a,
	input [15:0] b,
	input [15:0] c,
	input [15:0] d,
	input [15:0] e,
	input [15:0] f,
	input [15:0] g,
	input [15:0] h,
	input [15:0] i,
	input [3:0] sel,
	output logic [15:0] out
);

	// Case statements can only be used inside procedural blocks (always block)
	// This is a combinational circuit, so use a combinational always @(*) block.
	always @(*) begin
		out = '1;		// '1 is a special literal syntax for a number with all bits set to 1.
						// '0, 'x, and 'z are also valid.
						// I prefer to assign a default value to 'out' instead of using a
						// default case.
		case (sel)
			4'h0: out = a;
			4'h1: out = b;
			4'h2: out = c;
			4'h3: out = d;
			4'h4: out = e;
			4'h5: out = f;
			4'h6: out = g;
			4'h7: out = h;
			4'h8: out = i;
		endcase
	end
	
endmodule
```



#### 256-to-1 multiplexer

Hint中的两行:

- With this many options, a case statement isn't so useful.
- Vector indices can be variable, as long as the synthesizer can figure out that the width of the bits being selected is constant. In particular, selecting one bit out of a vector using a variable index will work.

所以我试了试, 我的代码和标答一样(这里放上标答):

```verilog
module top_module (
	input [255:0] in,
	input [7:0] sel,
	output  out
);

	// Select one bit from vector in[]. The bit being selected can be variable.
	assign out = in[sel];
	
endmodule
```



#### 256-to-1 4-bit multiplexer

因为Hint中说:

- Vector indices can be variable, as long as the synthesizer can figure out that the width of the bits being selected is constant. It's not always good at this. An error saying "... is not a constant" means it couldn't prove that the select width is constant. In particular, `in[ sel*4+3 : sel*4 ]` does not work.

所以第一次尝试试着用packed vector:

```verilog
module top_module( 
    input [1023:0] in,
    input [7:0] sel,
    output [3:0] out );
    
    wire [3:0] tmp [255:0];
    assign tmp = in;
    assign out = tmp[sel];

endmodule
```

但是有错误: Error (10928): SystemVerilog error at top_module.v(7): packed array type cannot be assigned to unpacked array type - types do not match File: /home/h/work/hdlbits.5572923/top_module.v

然后Hint中还有一条:

- Bit slicing ("Indexed vector part select", since Verilog-2001) has an even more compact syntax.

然后唯一一个和Hint中不同的方式是16页的**6.4 Vector Bit Selects and Part Selects**, 这种方式会指定起点、方向和长度。从这个角度来考虑可能是即使采用了` sel*4+3 : sel*4 `这样的切片数值, 编译器还是不能确保长度一致。虽然从常理上理解有点不理解, 可能是编译器在设计的时候就有一些额外考量吧。

所以我的代码是:

```verilog
module top_module( 
    input [1023:0] in,
    input [7:0] sel,
    output [3:0] out );
    
    assign out = in[(sel * 4)+:4];

endmodule
```

标答似乎选择了一种比我还直接的方式(当然在额外注释里用了我那种方法):

```verilog
module top_module (
	input [1023:0] in,
	input [7:0] sel,
	output [3:0] out
);

	// We can't part-select multiple bits without an error, but we can select one bit at a time,
	// four times, then concatenate them together.
	assign out = {in[sel*4+3], in[sel*4+2], in[sel*4+1], in[sel*4+0]};

	// Alternatively, "indexed vector part select" works better, but has an unfamiliar syntax:
	// assign out = in[sel*4 +: 4];		// Select starting at index "sel*4", then select a total width of 4 bits with increasing (+:) index number.
	// assign out = in[sel*4+3 -: 4];	// Select starting at index "sel*4+3", then select a total width of 4 bits with decreasing (-:) index number.
	// Note: The width (4 in this case) must be constant.

endmodule
```



### Arithmetic Circuits
#### Half adder
#### Full adder

异或本身就是半加, 半加的结果再加cin, 所以sum = a ^ b ^ cin;



#### 3-bit binary adder
#### Adder

我用了generate, 标答更加简洁:

```verilog
module top_module (
	input [3:0] x,
	input [3:0] y,
	output [4:0] sum
);

	// This circuit is a 4-bit ripple-carry adder with carry-out.
	assign sum = x+y;	// Verilog addition automatically produces the carry-out bit.

	// Verilog quirk: Even though the value of (x+y) includes the carry-out, (x+y) is still considered to be a 4-bit number (The max width of the two operands).
	// This is correct:
	// assign sum = (x+y);
	// But this is incorrect:
	// assign sum = {x+y};	// Concatenation operator: This discards the carry-out
endmodule
```

注意: (x + y)仍然是4位, 但是附带了进位信息; {x+y}则丢弃进位信息。



#### Signed addition overflow

一开始想得是, 直接利用`(x + y)`会附带进位信息来做这道题:

```verilog
module top_module (
    input [7:0] a,
    input [7:0] b,
    output [7:0] s,
    output overflow
); //
 
    // assign s = ...
    // assign overflow = ...
    assign {overflow, s} = a + b;

endmodule
```

但是这样会有错误, 例如, 8bits情况下 0x70 + 0x90 = 0x100, 本质是 0x70 - 0x70 = 0, 就不是溢出了。

正确的方式应该是用提示中检查符号位的方式, 例如两个正数的输出是个负数, 或者两个负数的输出是一个正数就是溢出了。

```verilog
module top_module (
    input [7:0] a,
    input [7:0] b,
    output [7:0] s,
    output overflow
); //
 
    // assign s = ...
    // assign overflow = ...
    assign s = a + b;
    assign overflow = a[7] & b[7] & ~s[7] | ~a[7] & ~b[7] & s[7];

endmodule
```



#### 100-bit binary adder

我的答案跟标答一样:

```verilog
module top_module( 
    input [99:0] a, b,
    input cin,
    output cout,
    output [99:0] sum );
    
    assign {cout, sum} = a + b + cin;

endmodule
```



#### 4-digit BCD adder
### Karnaugh Map to Circuit
#### 3-variable

设原图4行2列, $x \in \{{0,1}\}$, 原图中的0是$\{x, y\} = \{0, 0\}$。

我一开始圈定的范围是x = 1, y = {0, 1, 2, 3}、y = 1, x = {0, 1}、 y = {2, 3}, x = {0, 1}。

这样的范围就是:`assign out = a | b | ~b & c;`。

但是范围其实应该尽量往大的画, 应该画成x = 1, y = {0, 1, 2, 3}、y = {1, 2}, x = {0, 1}、 y = {2, 3}, x = {0, 1}。

这样的范围就是:`assign out = a | b | c;`。



#### 4-variable
#### 4-variable
#### 4-variable
#### Minimum SOP and POS
#### Karnaugh map
#### Karnaugh map
#### K-map implemented with a multiplexer

原题要求只能用2选1选择器来实现。

我先当作普通的verilog练习题来做:

```verilog
module top_module (
    input c,
    input d,
    output [3:0] mux_in
); 
    always @(*) begin
      mux_in = 4'b0;
    	case({c,d})
        2'b00: mux_in = 4'b0100;
            2'b01: mux_in = 4'b0001;
            2'b10: mux_in = 4'b0101;
            2'b11: mux_in = 4'b1001;
            
        endcase
    end

endmodule
```

然后是二选一来表达一些逻辑:

```verilog
module top_module (
	input c,
	input d,
	output [3:0] mux_in
);
	
	// After splitting the truth table into four columns,
	// the rest of this question involves implementing logic functions
	// using only multiplexers (no other gates).
	// I will use the conditional operator for each 2-to-1 mux: (s ? a : b)
	assign mux_in[0] = c ? 1 : d;          // 1 mux:   c|d
	assign mux_in[1] = 0;                  // No muxes:  0
	assign mux_in[2] = d ? 0 : 1;          // 1 mux:    ~d
	assign mux_in[3] = c ? d : 0;          // 1 mux:   c&d
	
endmodule
```

标答里面已经实现了与或非, 包括异或之内的其他逻辑都能用这样的基础来实现。



## Sequential Logic



### Latches and Flip-Flops



#### D flip-flop

我的答案与标答一致, 但标答提出了一个问题我暂时不知道该如何回答。

```verilog
module top_module(
	input clk,
	input d,
	output reg q);
	
	// Use non-blocking assignment for edge-triggered always blocks
	always @(posedge clk)
		q <= d;

	// Undefined simulation behaviour can occur if there is more than one edge-triggered
	// always block and blocking assignment is used. Which always block is simulated first?
	
endmodule
```





#### D flip-flops
#### DFF with reset
#### DFF with reset value
#### DFF with asynchronous reset

异步reset就是reset的行为也在敏感事件中, 然后标答详细解释了为什么reset要设为posedge:

```verilog
module top_module(
	input clk,
	input [7:0] d,
	input areset,
	output reg [7:0] q);
	
	// The only difference in code compared to synchronous reset is in the sensitivity list.
	always @(posedge clk, posedge areset)
		if (areset)
			q <= 0;
		else
			q <= d;


	// In Verilog, the sensitivity list looks strange. The FF's reset is sensitive to the
	// *level* of areset, so why does using "posedge areset" work?
	// To see why it works, consider the truth table for all events that change the input 
	// signals, assuming clk and areset do not switch at precisely the same time:
	
	//  clk		areset		output
	//   x		 0->1		q <= 0; (because areset = 1)
	//   x		 1->0		no change (always block not triggered)
	//  0->1	   0		q <= d; (not resetting)
	//  0->1	   1		q <= 0; (still resetting, q was 0 before too)
	//  1->0	   x		no change (always block not triggered)
	
endmodule
```



#### DFF with byte enable
#### D Latch

在组合逻辑里用<=确实违反常规, 但Hint提示我们这样做:

- Latches are level-sensitive (not edge-sensitive) circuits, so in an always block, they use level-sensitive sensitivity lists.
- However, they are still sequential elements, so should use non-blocking assignments.
- A D-latch acts like a wire (or non-inverting buffer) when enabled, and preserves the current value when disabled.

其实waring信息可以给出一定的解释:

Unless you intentionally wanted to create a latch, this warning usually indicates a bug in a combinational always block. Make sure every variable is assigned a value in all cases so the previous value does not need to be remembered. Possible ways to achieve this include assigning a default value to variables at the top of the always block, using a default case, or having an else clause.

说白了, 就是通过只给一个if不给else的方式,  额外创造一个latch。



#### DFF
#### DFF
#### DFF+gate

第一次尝试失败了, 但是感觉是中间过程的问题, 把d去掉, 直接改用out就好了:

```verilog
module top_module (
    input clk,
    input in, 
    output out);
    
    wire d;

    always @(posedge clk)begin
        d <= in ^ out;
        out <= d;
    end
    	
endmodule
```

第二次的尝试就成功了:

```verilog
module top_module (
    input clk,
    input in, 
    output out);

    always @(posedge clk)begin
        out <= in ^ out;
    end
    	
endmodule
```

我去群里问了一下, 然后有人告诉我, 我的第一种写法相当于构造了两个锁存器。于是我把第一个<=改成了=, 就成功了。

```verilog
module top_module (
    input clk,
    input in, 
    output out);
    
    wire d;

    always @(posedge clk)begin
        d = in ^ out;
        out <= d;
    end
    	
endmodule
```

于是就有了一系列疑问:

1. 在用hdlbits自学之前, 我对`=`与`<=`的看法就是, 前者是顺序执行, 后者是并行。但是当时我就有疑问, 例如在一个`posedge clk`之中, 如果有两个`=`, 为什么在已经是最小单位的一拍里还会有顺序之说?

2. 在学了[Simple wire](### Simple wire)那一节后, 我就意识到, 其实`=`其实就是物理上的连线, 但是包括hdlbits和手册都让我以为, 组合逻辑中的`=`和Procedural`=`有区别。尤其是手册25页:

   variable = expression;
   Blocking procedural assignment. Expression is evaluated and assigned when the statement is encountered. In a begin—end sequential statement group, execution of the next statement is blocked until the assignment is complete. In the sequence begin m=n; n=m; end, the first assignment changes m before the second assignment reads m.

   让我还是认为`=`是有顺序之分的, 但这又和我在[Simple wire](### Simple wire)那一节学到的矛盾。

然后我查了《数字逻辑基础与Verilog设计 原书第三版》相关章节(P154), 我终于理解了`=`。

其中提到:

*"="称为阻塞赋值。Verilog 编译器按照这些语句在always 块中的先后次序顺序地执行。如果一个变量通过阻塞赋值语句赋值，则这个新赋的值会被该块中所有后续语句使用。*

也就是说其实`=`无论在何时只是在连线。

至于m = n, q = m会什么会**表现为**阻塞是因为他们就是同一个节点(姑且可以这样认为, 书中的图片可以更好的表达, 其实是一种替换关系, 有点像宏替换)。而<=也只是**表现的像**计算机中的并行而已。



#### Mux and DFF
#### Mux and DFF
#### DFFs and gates

第一次出错是因为DFF名称与内置的模组名冲突了。



#### Create circuit from truth table
#### Detect an edge

1. 首先尝试直接用vector作为敏感事件, 但是报错表示只有vecotr的LSB会被纳入敏感事件。
2. 接着尝试把vector所有index分开都作为敏感列表, 报警告不要将reg作为敏感事件, 因为这种变化不是显式的。
3. 于是尝试用if来判断tmp和in是否相等, 当发生改变时赋值(就是现在的代码没有else), 然后以为没有else导致输出被维持了。
4. 补上了else发生了在1->0时也会触发这是因为in ^ tmp只是输出了变化的部分,  如果变化的部分是0则不用输出(只输出0->1的变化), 所以还要再&上in一次。

```verilog
module top_module (
    input clk,
    input [7:0] in,
    output [7:0] pedge
);
    
    reg [7:0]tmp;
    always @(posedge clk)begin
        if (in != tmp) begin
        tmp <= in;
        pedge <= in ^ tmp & in;
        end
        else pedge <= '0;
    end
        
endmodule
```

标答更简洁:

```verilog
module top_module(
	input clk,
	input [7:0] in,
	output reg [7:0] pedge);
	
	reg [7:0] d_last;	
			
	always @(posedge clk) begin
		d_last <= in;			// Remember the state of the previous cycle
		pedge <= in & ~d_last;	// A positive edge occurred if input was 0 and is now 1.
	end
	
endmodule
```



#### Detect both edges

就是我上一题第4次尝试写的那个错误答案, 因为这个是同时检测上升沿和下降沿。

然后参考上题标答同时其实可以省去tmp和in的判断, 直接用tmp ^ in来表示anyedge <= '0。

```verilog
module top_module (
    input clk,
    input [7:0] in,
    output [7:0] anyedge
);
    
    reg [7:0]tmp;
    always @(posedge clk)begin
        	tmp <= in;
          anyedge <= tmp ^ in;
    end

endmodule
```



#### Edge capture register

以下代码对了一半:

```verilog
module top_module (
    input clk,
    input reset,
    input [31:0] in,
    output [31:0] out
);
    reg [31:0]old;
    always @(posedge clk) begin
        old <= in;
        if (old & ~in || reset) begin
        	out <= reset? 0: old & ~in;
        end
    end

endmodule
```

其中, 没有else的if用于在没有真正关心的事件时保持。

这个答案可以通过一半的测试, 能通过这些测试的原因是因为在这之前的out为0, 这段代码(不考虑reset部分)的逻辑只会输出发生改变的那几位, 但是题目要求:

*"Capture" means that the output will remain 1 until the register is reset (synchronous reset).*

所以输出的并不是只有当前改变的, 还有之前out也应该输出。所以加上一个或逻辑就好了:

```verilog
module top_module (
    input clk,
    input reset,
    input [31:0] in,
    output [31:0] out
);
    reg [31:0]old;
    always @(posedge clk) begin
        old <= in;
        if (old & ~in || reset) begin
        	out <= reset? 0: old & ~in | out;
        end
    end

endmodule
```



#### Dual-edge triggered flip-flop

第一次的尝试是受到手册中:

*NOTE: The Verilog language does not have a true “sensitivity list”. ...*的启发:

```verilog
module top_module (
    input clk,
    input d,
    output q
);
  
  reg old;
  always @(*) begin
  	old <= clk;
    if (old != clk) q <= d;
  end

endmodule
```

但是果然还是不行, 这样q会一直输出0。

然后还是参考Hint:

- You can't create a dual-edge triggered flip-flop on an FPGA. But you can create both positive-edge triggered *and* negative-edge triggered flip-flops.

```verilog
module top_module (
    input clk,
    input d,
    output q
);
    
	wire q1, q2;
  assign q = q1 | q2;
    
    always @(posedge clk) begin
        q1 <= d;
    end
    
    always @(negedge clk) begin
        q2 <= d;
    end

endmodule
```

但是这样会导致有些时候q的输出会有延时, 导致这样的原因是:

1. 虽然声明为wire, 但其实q1和q2是reg。(对于verilog而言wire和reg其实没有区别。)
2. 对于q1和q2, 其实会保持一个clk周期, 但其实真正需要关注的其实是半个clk周期。

所以只要改进为按照clk分类就好:

```verilog
module top_module (
    input clk,
    input d,
    output q
);
    
	wire q1, q2;
    assign q = clk ? q1 : q2;
    
    always @(posedge clk) begin
        q1 <= d;
    end
    
    always @(negedge clk) begin
        q2 <= d;
    end

endmodule
```

我一开始写的是, `assign q = clk ? q2 : q1;`, 由于我误会了<=会占一拍, 其实这种一拍是相对于那一瞬间的一拍。

标答更简洁, 但更费脑子:

```verilog
module top_module(
	input clk,
	input d,
	output q);
	
	reg p, n;
	
	// A positive-edge triggered flip-flop
    always @(posedge clk)
        p <= d ^ n;
        
    // A negative-edge triggered flip-flop
    always @(negedge clk)
        n <= d ^ p;
    
    // Why does this work? 
    // After posedge clk, p changes to d^n. Thus q = (p^n) = (d^n^n) = d.
    // After negedge clk, n changes to d^p. Thus q = (p^n) = (p^d^p) = d.
    // At each (positive or negative) clock edge, p and n FFs alternately
    // load a value that will cancel out the other and cause the new value of d to remain.
    assign q = p ^ n;
    
    
	// Can't synthesize this.
	/*always @(posedge clk, negedge clk) begin
		q <= d;
	end*/
    
    
endmodule
```



### Counters
#### Four-bit binary counter
#### Decade counter

一开始纠结了一下什么时候去进位归零, 一开始想着用q == 10的时候, 然后感觉不对就直接偷懒了:

```verilog
module top_module (
    input clk,
    input reset,        // Synchronous active-high reset
    output [3:0] q);
    
    always @(posedge clk)
        q <= reset? 4'h0 : (q + 1) % 10;
            
endmodule
```

然后看了一下标答:

```verilog
module top_module(
	input clk,
	input reset,
	output reg [3:0] q);
	
	always @(posedge clk)
		if (reset || q == 9)	// Count to 10 requires rolling over 9->0 instead of the more natural 15->0
			q <= 0;
		else
			q <= q+1;
	
endmodule
```

由于q == 9其实是这一拍开始前瞬间q的状态。感觉自己没想明白是因为我困了(笑哭)。



#### Decade counter again
#### Slow decade counter

这题做了挺久的, 主要是没有省清题目。

题目说:

*slowena input indicates when the counter should increment.*

然后照着之前来写会有一个问题:

```verilog
module top_module (
    input clk,
    input slowena,
    input reset,
    output [3:0] q);
    
    reg [3:0]p;
    
    always @(posedge clk)
      if (reset || q == 4'd9)
            	q <= 4'h0;
    		else begin
                if (slowena)
                    q <= (q + 4'h1) % 4'd10;
            end
          
endmodule
```

所以这也是我觉得之前[Decade counter](#### Decade counter)那题标答有的问题, 因为本质在q == 4'd9的时候将q <= 0也是一种*counter increment*的情况。

所以综合一下我的代码是:

```verilog
module top_module (
    input clk,
    input slowena,
    input reset,
    output [3:0] q);
    
    reg [3:0]p;
    
    always @(posedge clk)
        if (reset)
            q <= 4'h0;
    		else begin
                if (slowena)
                    q <= (q + 4'h1) % 4'd10;
            end
               
endmodule
```



#### Counter 1-12

摸索了一阵:

然后有了这个半对的写法, 还是因为c_load只有if没有else导致其没有归零。

```verilog
module top_module (
    input clk,
    input reset,
    input enable,
    output [3:0] Q,
    output c_enable,
    output c_load,
    output [3:0] c_d
); 

    assign c_enable = enable;
    
    always @(*) begin
        if (reset) begin c_load = '1; c_d = 4'd1; end
        else begin 
            if (Q == 4'd12 && enable) begin c_load = 4'd1; c_d = 4'd1; end
        end 
    end
    
    count4 the_counter (clk, c_enable, c_load, c_d, Q);

endmodule
```

改一下就好了:

```verilog
module top_module (
    input clk,
    input reset,
    input enable,
    output [3:0] Q,
    output c_enable,
    output c_load,
    output [3:0] c_d
); 

    assign c_enable = enable;
    
    always @(*) begin
        if (reset) begin c_load = '1; c_d = 4'd1; end
        else if (Q == 4'd12 && enable) begin c_load = 4'd1; c_d = 4'd1; end
        else begin c_load = '0; end
    end
    
    count4 the_counter (clk, c_enable, c_load, c_d, Q);

endmodule
```



#### Counter 1000

暂不讨论1Hz的输出错误, 这一份代码会导致第三个计数器无法进位, 因为第三个计数器要进位时已经满足了第一个if条件。

```verilog
module top_module (
    input clk,
    input reset,
    output OneHertz,
    output [2:0] c_enable
); //
    
    reg [3:0] q0, q1, q2;
    wire r0, r1, r2;
    reg out;
    
    assign c_enable[0] = 1;
    
    always @(*)begin
        r0 = 0; r1 = 0; r2 = 0; c_enable[2:1] = 2'b0;
        if (q0 == 4'd9)  begin r0 = 1; c_enable[1] = 1; end 
        else if (q1 == 4'd9)  begin r1 = 1; c_enable[2] = 1; end 
        else if (q2 == 4'd9)  begin r2 = 1; out = ~out; end
    end

    bcdcount counter0 (clk, reset | r0, c_enable[0], q0);
    bcdcount counter1 (clk, reset | r1, c_enable[1], q1);
    bcdcount counter2 (clk, reset | r2, c_enable[2], q2);

endmodule
```

所以注意if应该是嵌套的。

然后我一开始以为是实现一个维持高电平0.5s再维持0.5s低电平的时钟, 但其实不是, 只是每1s跳动一下的时钟:

```verilog
module top_module (
    input clk,
    input reset,
    output OneHertz,
    output [2:0] c_enable
); //
    
    reg [3:0] q0, q1, q2;
    wire r0, r1, r2;
    
    assign c_enable[0] = 1;
    
    always @(*)begin
        r0 = 0; r1 = 0; r2 = 0; c_enable[2:1] = 2'b0; OneHertz = '0;
        if (q0 == 4'd9)  begin 
            r0 = 1; c_enable[1] = 1; 
            if(q1 == 4'd9) begin 
                r1 = 1; c_enable[2] = 1'b1; 
                if (q2 == 4'd9)  begin 
                    r2 = 1; OneHertz = 1'b1;
                end
            end
        end 
    end

    bcdcount counter0 (clk, reset | r0, c_enable[0], q0);
    bcdcount counter1 (clk, reset | r1, c_enable[1], q1);
    bcdcount counter2 (clk, reset | r2, c_enable[2], q2);

endmodule
```

**但其实想想对于上升沿而言, 这也是个1Hz的时钟。**(因为每1s出现一次上升沿)

另外尝试了一下其他写法, 跟一遍的编程语言还是有差异的, 以下的写法是有问题的, 达不到相同的效果:

```verilog
module top_module (
    input clk,
    input reset,
    output OneHertz,
    output [2:0] c_enable
); //
    
    reg [3:0] q0, q1, q2;
    wire r0, r1, r2;
    
    assign c_enable[0] = 1;
    
    always @(*)begin
        r0 = 0; r1 = 0; r2 = 0; c_enable[2:1] = 2'b0; OneHertz = '0;
        if (q0 == 4'd9) begin r0 = 1; c_enable[1] = 1; end
        if (q1 == 4'd9) begin r1 = 1; c_enable[2] = 1; end
        if (q2 == 4'd9) begin r2 = 1; OneHertz = 1'b1; end
    end

    bcdcount counter0 (clk, reset | r0, c_enable[0], q0);
    bcdcount counter1 (clk, reset | r1, c_enable[1], q1);
    bcdcount counter2 (clk, reset | r2, c_enable[2], q2);

endmodule
```

然后看了一下网上其他人的代码, 其实是不需要单独去写那几个r的, 因为本身超过那个值就会归0。

另外, 用backwards思想能让代码更简洁:

```verilog
module top_module (
    input clk,
    input reset,
    output OneHertz,
    output [2:0] c_enable
); //
    
  reg [3:0] q0, q1, q2;
  
  bcdcount counter0 (clk, reset, c_enable[0], q0);
  bcdcount counter1 (clk, reset, c_enable[1], q1);
  bcdcount counter2 (clk, reset, c_enable[2], q2);
    
  assign c_enable[0] = 1;
  assign c_enable[1] = (q0 == 4'd9);
  assign c_enable[2] = (q0 == 4'd9) && (q1 == 4'd9);
    
  assign OneHertz = ((q0 == 4'd9) && (q1 == 4'd9) && (q2 == 4'd9))? 1'b1 :1'b0;

endmodule
```



#### 4-digit decimal counter

 改用backwards思想, 代码简洁又快乐:

```verilog
module top_module (
    input clk,
    input reset,   // Synchronous active-high reset
    output [3:1] ena,
    output [15:0] q);
    
    wire [3:0] q0, q1, q2, q3;
    
    assign q = {q3, q2, q1, q0};
    
  	assign ena[1] = (q0 == 4'd9);
  	assign ena[2] = (q0 == 4'd9) && (q1 == 4'd9);
    assign ena[3] = (q0 == 4'd9) && (q1 == 4'd9) && (q2 == 4'd9);
    
    bcdcount counter0 (clk, reset, 1'b1, q0);
    bcdcount counter1 (clk, reset, ena[1], q1);
    bcdcount counter2 (clk, reset, ena[2], q2);
    bcdcount counter3 (clk, reset, ena[3], q3);

endmodule

module bcdcount (
    input clk,
    input reset,
    input enable,
    output [3:0] q);
   
    always @(posedge clk)
        if (reset)
            q <= 4'h0;
    		else begin
          if (enable) q <= (q + 4'h1) % 4'd10;
        end
          
endmodule
```



#### 12-hour clock

挺麻烦的, 单纯记录一下我自己的成功:

```verilog
module top_module(
    input clk,
    input reset,
    input ena,
    output pm,
    output [7:0] hh,
    output [7:0] mm,
    output [7:0] ss); 
    
    wire enable[2:0];
    
    assign enable[0] = ena;
    assign enable[1] = ena && ss == {4'd5, 4'd9};
    assign enable[2] = ena && mm == {4'd5, 4'd9} && ss == {4'd5, 4'd9};
    
    always @(posedge clk)
        pm <= reset? '0: (hh == {4'd1, 4'd1} && mm == {4'd5, 4'd9} && ss == {4'd5, 4'd9})? ~pm: pm;
    
    bcdcount60 S(clk, reset, enable[0], ss);
    bcdcount60 M(clk, reset, enable[1], mm);
    bcdcount12 H(clk, reset, enable[2], hh);
    
endmodule

module bcdcount60 (
    input clk,
    input reset,
    input enable,
    output [7:0] q);
   
    always @(posedge clk) begin
        q[3:0] <= reset? 4'h0: enable? {q[3:0] + 4'h1} % 4'd10 : q[3:0];
    	q[7:4] <= reset? 4'h0: enable? ((q[3:0] == 4'd9)? {q[7:4] + 4'h1} % 4'd6: q[7:4]) :q[7:4];
        /*
        if (reset) begin q <= 8'b0; end
        else begin
            if (enable) begin
                q[3:0] <= {q[3:0] + 4'h1} % 4'd10;
                q[7:4] <= (q[3:0] == 4'd9)? {q[7:4] + 4'h1} % 4'd6: q[7:4];
            end
        end
        */
    end
        
endmodule

module bcdcount12 (
    input clk,
    input reset,
    input enable,
    output [7:0] q);
   
    always @(posedge clk) begin
        if (reset) begin
            q <= {4'd1, 4'd2};
        end 
        else begin
            if (enable) begin
                if (q[7:4] == 4'd1 && q[3:0] == 4'd2) begin q[7:4] <= '0; q[3:0] <= 4'd1; end
                else begin
                    q[3:0] <= {q[3:0] + 4'h1} % 4'd10;
                    q[7:4] <= (q[3:0] == 4'd9)? 4'd1: q[7:4];
                end
            end
        end
    end
        
endmodule
```



### Shift Registers
#### 4-bit shift register

我的代码:

```verilog
module top_module(
    input clk,
    input areset,  // async active-high reset to zero
    input load,
    input ena,
    input [3:0] data,
    output reg [3:0] q); 
    
    always @(posedge clk, posedge areset) begin
        if (areset)begin
        	q <= 4'b0;
        end
        else begin
            if (load) begin
            	q <= data;
            end
            else begin
                if (ena) q <= q >> 1;
            end
        end
    end
        
endmodule
```

我用嵌套的if来解决优先级问题。但似乎标答直接用else if来解决这个问题:

```verilog
module top_module(
	input clk,
	input areset,
	input load,
	input ena,
	input [3:0] data,
	output reg [3:0] q);
	
	// Asynchronous reset: Notice the sensitivity list.
	// The shift register has four modes:
	//   reset
	//   load
	//   enable shift
	//   idle -- preserve q (i.e., DFFs)
	always @(posedge clk, posedge areset) begin
		if (areset)		// reset
			q <= 0;
		else if (load)	// load
			q <= data;
		else if (ena)	// shift is enabled
			q <= q[3:1];	// Use vector part select to express a shift.
	end
	
endmodule
```

我查了一下手册, 发现似乎verilog-2001中只有if和if - else, 没有else if(或者说else if就是另外一种if - else + if的可能性?我不确定)。

但System Verilog明确支持else if。

另外据我查到的说法, if确实有优先级之说, 第一个最高, 最后的else最低。case只用于单变量互斥的情况。



#### Left/right rotator
#### Left/right arithmetic shift by 1 or 8
#### 5-bit LFSR

我的代码:

```verilog
module top_module(
    input clk,
    input reset,    // Active-high synchronous reset to 5'h1
    output [4:0] q
); 
    always @(posedge clk) begin
        if (reset) begin
            q <= 5'h1;
        end
        else begin
            {q[3], q[1:0]} <= {q[4], q[2:1]};
            {q[4], q[2]} <= {1'b0, q[3]} ^ {2{q[0]}};    
        end
    end
        
endmodule
```

表答的思想可以借鉴:

```verilog
module top_module(
	input clk,
	input reset,
	output reg [4:0] q);
	
	reg [4:0] q_next;		// q_next is not a register

	// Convenience: Create a combinational block of logic that computes
	// what the next value should be. For shorter code, I first shift
	// all of the values and then override the two bit positions that have taps.
	// A logic synthesizer creates a circuit that behaves as if the code were
	// executed sequentially, so later assignments override earlier ones.
	// Combinational always block: Use blocking assignments.
	always @(*) begin
		q_next = q[4:1];	// Shift all the bits. This is incorrect for q_next[4] and q_next[2]
		q_next[4] = q[0];	// Give q_next[4] and q_next[2] their correct assignments
		q_next[2] = q[3] ^ q[0];
	end
	
	
	// This is just a set of DFFs. I chose to compute the connections between the
	// DFFs above in its own combinational always block, but you can combine them if you wish.
	// You'll get the same circuit either way.
	// Edge-triggered always block: Use non-blocking assignments.
	always @(posedge clk) begin
		if (reset)
			q <= 5'h1;
		else
			q <= q_next;
	end
	
endmodule
```

注意: **A logic synthesizer creates a circuit that behaves as if the code were executed sequentially, so later assignments override earlier ones.**



#### 3-bit LFSR
#### 32-bit LFSR
#### Shift register
#### Shift register
#### 3-input LUT

### More Circuits
Cellular automata
#### Rule 90
#### Rule 110
#### Conway's Game of Life 16x16

### Finite State Machines
#### Simple FSM 1 (asynchronous reset)

#### Simple FSM 1 (synchronous reset)
Simple FSM 2 (asynchronous reset)
Simple FSM 2 (synchronous reset)
Simple state transitions 3
Simple one-hot state transitions 3
Simple FSM 3 (asynchronous reset)
Simple FSM 3 (synchronous reset)
Design a Moore FSM
Lemmings 1
Lemmings 2
Lemmings 3
Lemmings 4
One-hot FSM
PS/2 packet parser
PS/2 packet parser and datapath
Serial receiver
Serial receiver and datapath
Serial receiver with parity checking
Sequence recognition
Q8: Design a Mealy FSM
Q5a: Serial two's complementer (Moore FSM)
Q5b: Serial two's complementer (Mealy FSM)
Q3a: FSM
Q3b: FSM
Q3c: FSM logic
Q6b: FSM next-state logic
Q6c: FSM one-hot next-state logic
Q6: FSM
Q2a: FSM
Q2b: One-hot FSM equations
Q2a: FSM
Q2b: Another FSM



## Building Larger Circuits

### Counter with period 1000
4-bit shift register and down counter
FSM: Sequence 1101 recognizer
FSM: Enable shift register
FSM: The complete FSM
The complete timer
FSM: One-hot logic equations



# Verification: Reading Simulations

## Finding bugs in code

### Mux
NAND
Mux
Add/sub
Case statement


## Build a circuit from a simulation waveform
### Combinational circuit 1
Combinational circuit 2
Combinational circuit 3
Combinational circuit 4
Combinational circuit 5
Combinational circuit 6
Sequential circuit 7
Sequential circuit 8
Sequential circuit 9
Sequential circuit 10



## Verification: Writing Testbenches
### Clock
Testbench1
AND gate
Testbench2
T flip-flop

