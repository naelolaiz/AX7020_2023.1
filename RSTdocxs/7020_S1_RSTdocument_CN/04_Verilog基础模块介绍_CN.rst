Introduction to Verilog Basic Modules
=====================================

Introduction
------------

This article mainly introduces Verilog basic modules. Building a solid foundation will be very helpful for in-depth FPGA learning.

Data Types
----------

Constants
~~~~~~~~~

**Integers**\ : Integers can be represented in binary with b or B, octal with o or O, decimal with d or D, and hexadecimal with h or H. For example, 8'b00001111 represents an 8-bit binary integer, and 4'ha represents a 4-bit hexadecimal integer.

**X and Z**\ : X represents an unknown value, and z represents a high-impedance value. For example, 5'b00x11 has an unknown value at the third bit, and 3'b00z indicates the least significant bit is high-impedance.

**Underscore**\ : When the number of bits is too long, underscores can be used to separate the bits for better readability, such as 8'b0000_1111.

**Parameter**:
parameter can be used to define constants with identifiers. When used, only the identifier is needed, improving readability and maintainability. For example, defining parameter
width = 8 ; and then defining register reg [width-1:0] a; defines a register with 8-bit width.

Parameter passing: If a module has defined parameters, other modules can pass and modify parameters when instantiating this module, as shown below, using #() after module.

For example, define the module as follows 

Instantiating module

.. code:: verilog

 module rom 
 #( 
   parameter depth =15, 
   parameter width = 8  
   ) 
   ( 
    input [depth-1:0] addr , 
 input [width-1:0] data , 
 output result 
 ) ; 
 
 endmodule

Top-level module

.. code:: verilog

 module top() ; 
  
 wire [31:0] addr ; 
 wire [15:0] data ; 
 wire result ; 
  
 rom 
 #( 
   .depth(32), 
   .width(16)  
   )
 r1  
 ( 
 .addr(addr) , 
 .data(data) , 
 .result(result) 
 ) ;  
 endmodule 

Parameter can be used for parameter passing between modules, while localparam is only used within the current module and cannot be used for parameter passing. Localparam is often used for defining state machine states.

Variables
---------

Variables refer to quantities whose values can change during program execution. The following mainly introduces several commonly used variable types.

Wire Type
~~~~~~~~~

Wire
type variables, also called net type variables, are used for physical connections between structural entities, such as between gates. They cannot store values and are assigned using the continuous assignment statement assign. Defined as wire
[n-1:0] a ; where n represents the bit width. For example, defining wire a ; assign a = b ;
connects node b to wire a. As shown in the figure below, the connections between two entities are wire type variables.

.. image:: images/04_media/image1.png
      
Reg Type
~~~~~~~~

Reg
type variables, also called register variables, can be used to store values and must be used within always statements. They are defined as

reg [n-1:0] a ; representing an n-bit wide register. For example, reg [7:0] a;
defines an 8-bit wide register a. As shown below, register q is defined, and the generated circuit is sequential logic. The figure below shows its structure, which is a D flip-flop.

.. code:: verilog

 module top(d, clk, q) ; 
 input  d  ; 
 input clk ; 
 output reg q ; 
  
 always @(posedge clk) 
 begin 
   q <= d ; 
 end   
 endmodule 

|image1|

It can also generate combinational logic, such as a data selector (multiplexer). The sensitive signals do not include a clock. Reg
Mux is defined, and the final generated circuit is combinational logic.

.. code:: verilog

 module top(a, b, c, d, sel, Mux) ; 
 input   a ; 
 input   b ; 
 input   c ; 
 input   d ; 
 input [1:0] sel ; 
 output reg Mux ; 
  
 always @(sel or a or b or c or d) 
 begin 
   case(sel) 
     2'b00 : Mux = a ; 
     2'b01 : Mux = b ; 
     2'b10 : Mux = c ; 
     2'b11 : Mux = d ; 
   endcase 
 end 
    
 endmodule

|image2|

Memory Type
~~~~~~~~~~~

The memory type can be used to define storage devices such as RAM and ROM. Its structure is reg [n-1:0]
memory_name[m-1:0], meaning m registers with n-bit width. For example, reg [7:0] ram
[255:0] defines 256 8-bit registers, where 256 is the storage depth and 8 is the data width.

Operators
---------

Operators can be classified into the following categories:

1. Arithmetic operators (+, -, \*, /, %)

2. Assignment operators (=, <=)

3. Relational operators (>, <, >=, <=, ==, !=)

4. Logical operators (&&, ||, !)

5. Conditional operator (?:)

6. Bitwise operators (~, \|, ^, &, ^~)

7. Shift operators (<<, >>)

8. Concatenation operator ({ })

Arithmetic Operators
~~~~~~~~~~~~~~~~~~~~

"+" (addition operator), "-" (subtraction operator), "*" (multiplication operator), "/" (division operator, e.g., 7/3
=2), "%" (modulo operator, i.e., remainder, e.g., 7%3=1, remainder is 1)

Assignment Operators
~~~~~~~~~~~~~~~~~~~~

"=" blocking assignment, "<=" non-blocking assignment. Blocking assignment executes one assignment statement before the next, which can be understood as sequential execution, and the assignment takes effect immediately. Non-blocking assignment can be understood as parallel execution, regardless of order, and the assignment only takes effect after the always block finishes execution. The following is an example of blocking assignment:

The code is as follows: 

.. code:: verilog

 module top(din,a,b,c,clk); 
  
 input din; 
 input clk; 
 output reg a,b,c; 
  
 always @(posedge clk)  
 begin 
         a = din; 
         b = a; 
         c = b; 
 end 
  
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg din ; 
 reg clk ; 
 wire a,b,c ; 
  
 initial 
 begin 
   din =  0 ; 
   clk = 0  ; 
   forever 
   begin     
     #({$random}%100) 
     din = ~din ; 
   end 
 end 
  
 always #10 clk = ~clk ; 
  
 top  t0(.din(din),.a(a),.b(b),.c(c),.clk(clk)) ; 
 endmodule 

As seen from the simulation results, at the rising edge of clk, the value of a equals din and is immediately assigned to b, and b's value is assigned to c.

.. image:: images/04_media/image4.png
      
If changed to non-blocking assignment, the simulation results are as follows: at the rising edge of clk, the value of a is not immediately assigned to b. b retains the previous value of a, and similarly, c retains the previous value of b.

.. image:: images/04_media/image5.png
      
The differences between the two can be clearly seen from their RTL diagrams:

|image3| |image4|

Blocking assignment RTL diagram  Non-blocking assignment RTL diagram

**In general, non-blocking assignment should be used in sequential logic circuits to avoid race conditions during simulation; blocking assignment should be used in combinational logic, where the assignment takes effect immediately; blocking assignment must be used in assign statements.**

Relational Operators
~~~~~~~~~~~~~~~~~~~~

Used to express the relationship between two operands, such as a>b, a<b. Mostly used for conditional judgment, for example:

::

 If (a>=b) q <=1'b1 ;
 else q <= 1'b0 ;

This means if the value of a is greater than or equal to b, then q is 1; otherwise q is 0.

Logical Operators
~~~~~~~~~~~~~~~~~

"&&" (logical AND of two operands), "||" (logical OR of two operands), "!" (logical NOT of a single operand). For example:

If (a>b && c <d) means the condition is a>b AND c<d; if
(!a) means the condition is that the value of a is not 1, i.e., it is 0.

Conditional Operator
~~~~~~~~~~~~~~~~~~~~

"?:" is a conditional judgment, similar to if else. For example, assign a = (i>8)?1'b1:1'b0
; checks whether the value of i is greater than 8. If greater than 8, a is 1; otherwise a is 0.

Bitwise Operators
~~~~~~~~~~~~~~~~~

"~" bitwise NOT, "|" bitwise OR, "^" bitwise XOR, "&" bitwise AND, "^~" bitwise XNOR. Except for "~" which requires only one operand, the others require two operands, such as a&b, a|b. Specific applications are explained in the combinational logic section later.

Shift Operators
~~~~~~~~~~~~~~~

"<<" left shift operator, ">>" right shift operator. For example, a<<1 means shift left by 1 bit, a>>2 means shift right by 2 bits.

Concatenation Operator
~~~~~~~~~~~~~~~~~~~~~~

"{ }" concatenation operator, used to concatenate multiple signals by bit. For example, {a[3:0],
b[1:0]} concatenates the lower 4 bits of a and the lower 2 bits of b into 6-bit data. Additionally, {n{a[3:0]}} means concatenating n copies of a[3:0], and {n{1'b0}} means concatenating n zeros. For example, {8{1'b0}} represents 8'b0000_0000.

Priority Levels
~~~~~~~~~~~~~~~

The priority levels of various operators are as follows:

.. image:: images/04_media/image8.png
      
Combinational Logic
-------------------

This section mainly introduces combinational logic. The characteristic of combinational logic circuits is that the output at any moment depends solely on the input signals. When the input signals change, the output changes immediately, independent of the clock.

AND Gate
~~~~~~~~

In Verilog, "&" represents bitwise AND. For example, c=a&b. The truth table is shown below. The result is 1 only when both a and b are 1. The RTL representation is shown on the right.

|image5| |image6|

The code implementation is as follows: 

.. code:: verilog

 module top(a, b, c) ; 
 input  a ; 
 input  b ; 
 output c ; 
  
 assign c = a & b ; 
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg a ; 
 reg b ; 
 wire c ; 
  
 initial 
 begin 
   a = 0 ; 
   b = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     a = ~a ; 
     #({$random}%100)  
     b = ~b ;  
   end 
 end 
  
 top  t0(.a(a), .b(b),.c(c)) ; 
  
 endmodule 

The simulation results are as follows:

.. image:: images/04_media/image11.png
      
If the bit width of a and b is greater than 1, for example, defining input [3:0] a, input
[3:0]b, then a&b means the corresponding bits of a and b are ANDed, such as a[0]&b[0], a[1]&b[1].

OR Gate
~~~~~~~

In Verilog, "|" represents bitwise OR. For example, c = a|b.
The truth table is shown below. The result is 0 only when both a and b are 0.

|image7| |image8|

The code implementation is as follows:

.. code:: verilog

 module top(a, b, c) ; 
 input  a ; 
 input  b ; 
 output c ; 
  
 assign c = a | b ; 
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg a ; 
 reg b ; 
 wire c ; 
  
 initial 
 begin 
   a = 0 ; 
   b = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     a = ~a ; 
     #({$random}%100)  
     b = ~b ;  
   end 
 end 
  
 top  t0(.a(a), .b(b),.c(c)) ; 
  
 endmodule 

The simulation results are as follows:

.. image:: images/04_media/image14.png
      
Similarly, if the bit width is greater than 1, the operation is performed bitwise OR.

NOT Gate
~~~~~~~~

In Verilog, "~" represents bitwise NOT. For example, b=~a. The truth table is shown below. b equals the inverse of a.

|image9| |image10|

The code implementation is as follows: 

.. code:: verilog

 module top(a, b) ; 
 input   a ; 
 output  b ; 
  
 assign b = ~a ; 
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg  a ; 
 wire b ; 
  
 initial 
 begin 
   a = 0 ;   
   forever 
   begin     
     #({$random}%100) 
     a = ~a ;     
   end 
 end 
  
 top  t0(.a(a), .b(b)) ; 
  
 endmodule

The simulation results are as follows:

.. image:: images/04_media/image17.png
      
XOR Gate
~~~~~~~~

In Verilog, "^" represents XOR. For example, c= a^b. The truth table is shown below. When a and b are the same, the output is 0.

|image11| |image12|

The code implementation is as follows: 

.. code:: verilog

 module top(a, b, c) ; 
 input  a ; 
 input  b ; 
 output c ; 
  
 assign c = a ^ b ; 
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg a ; 
 reg b ; 
 wire c ; 
  
 initial 
 begin 
   a = 0 ; 
   b = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     a = ~a ; 
     #({$random}%100)  
     b = ~b ;  
   end 
 end 
  
 top  t0(.a(a), .b(b),.c(c)) ; 
  
 endmodule 

The simulation results are as follows:

.. image:: images/04_media/image20.png
      
Comparator
~~~~~~~~~~

In Verilog, greater than ">", equal to "==", less than "<", greater than or equal to ">=", less than or equal to "<=", and not equal to "!=" are used. Taking greater than as an example, c=
a > b ; means if a is greater than b, then c is 1; otherwise c is 0. The truth table is as follows:

|image13|\ |image14|

The code implementation is as follows:

.. code:: verilog
 
 module top(a, b, c) ; 
 input  a ; 
 input  b ; 
 output c ; 
  
 assign c = a > b ; 
 endmodule 

The testbench file is as follows:

.. code:: verilog
 
 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg a ; 
 reg b ; 
 wire c ; 
  
 initial 
 begin 
   a = 0 ; 
   b = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     a = ~a ; 
     #({$random}%100)  
     b = ~b ;  
   end 
 end 
  
 top  t0(.a(a), .b(b),.c(c)) ; 
  
 endmodule

The simulation results are as follows:

.. image:: images/04_media/image23.png
      
Half Adder
~~~~~~~~~~

The half adder and full adder are basic units in arithmetic circuits. Since the half adder does not consider the carry from lower bits, it is called a half adder. sum represents the addition result, and count represents the carry. The truth table is as follows:

\ |image15|\ |image16|

The code can be written based on the truth table as follows: 

.. code:: verilog

 module top(a, b, sum, count) ; 
 input  a ; 
 input  b ; 
 output sum ; 
 output count ; 
  
 assign sum = a ^ b ; 
 assign count = a & b ; 
  
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg a ; 
 reg b ; 
 wire sum ; 
 wire count ; 
  
 initial 
 begin 
   a = 0 ; 
   b = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     a = ~a ; 
     #({$random}%100)  
     b = ~b ;  
   end 
   end 
  
 top  t0(.a(a), .b(b), 
 .sum(sum), .count(count)) ; 
  
 endmodule

The simulation results are as follows:

.. image:: images/04_media/image26.png
      
Full Adder
~~~~~~~~~~

The full adder adds the carry input signal cin from the lower bits. The truth table is as follows:

|image17|\ |image18|

The code is as follows: 

.. code:: verilog

 module top(cin, a, b, sum, count) ; 
 input cin ; 
 input  a ; 
 input  b ; 
 output sum ; 
 output count ; 
  
 assign {count,sum} = a + b + cin ; 
  
 endmodule

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg a ; 
 reg b ; 
 reg cin ; 
 wire sum ; 
 wire count ; 
  
 initial 
 begin 
   a = 0 ; 
   b = 0 ; 
   cin = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     a = ~a ; 
     #({$random}%100)  
 b = ~b ;  
 #({$random}%100)  
     cin = ~cin ;  
  
   end 
 end 
  
 top  t0(.cin(cin),.a(a), .b(b), 
 .sum(sum), .count(count)) ; 
  
 endmodule

The simulation results are as follows:

.. image:: images/04_media/image29.png
      
Multiplier
~~~~~~~~~~

Multiplication representation is also very simple, using "*". For example, a*b. Example code is as follows:

.. code:: verilog

 module top(a, b, c) ; 
 input  [1:0] a ; 
 input  [1:0] b ; 
 output [3:0] c ; 
  
 assign c = a * b ; 
 endmodule

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg [1:0] a ; 
 reg [1:0] b ; 
 wire [3:0] c ; 
  
 initial 
 begin 
   a = 0 ; 
   b = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     a = ~a ; 
     #({$random}%100)  
     b = ~b ;  
   end 
 end 
  
 top  t0(.a(a), .b(b),.c(c)) ; 
  
 endmodule

The simulation results are as follows:

.. image:: images/04_media/image30.png
      
Multiplexer
~~~~~~~~~~~

Multiplexers are frequently used in Verilog. Through selection signals, different input signals are routed to the output. As shown in the truth table below, a 4-to-1 multiplexer has sel[1:0] as the selection signal, a, b, c, d as input signals, and Mux as the output signal.

.. image:: images/04_media/image31.png
      
.. image:: images/04_media/image3.png
      
The code is as follows: 

.. code:: verilog

 module top(a, b, c, d, sel, Mux) ; 
 input   a ; 
 input   b ; 
 input   c ; 
 input   d ; 
  
 input [1:0] sel ; 
  
 output reg Mux ; 
  
 always @(sel or a or b or c or d) 
 begin 
   case(sel) 
     2'b00 : Mux = a ; 
     2'b01 : Mux = b ; 
     2'b10 : Mux = c ; 
     2'b11 : Mux = d ; 
   endcase 
 end 
    
 endmodule

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg  a ; 
 reg  b ; 
 reg  c ; 
 reg  d ; 
 reg [1:0] sel ; 
 wire  Mux ; 
  
 initial 
 begin 
   a = 0 ; 
   b = 0 ; 
   c = 0 ; 
   d = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     a = {$random}%3 ; 
     #({$random}%100)  
     b = {$random}%3 ; 
     #({$random}%100) 
     c = {$random}%3 ; 
     #({$random}%100)  
     d = {$random}%3 ; 
   end 
   end 
  
 initial 
 begin 
   sel = 2'b00 ; 
   #2000 sel =  2'b01 ; 
   #2000 sel =  2'b10 ; 
   #2000 sel =  2'b11 ; 
 end 
  
 top  
 t0(.a(a), .b(b),.c(c),.d(d), .sel(sel),
 .Mux(Mux)) ; 
  
 endmodule 


The simulation results are as follows

.. image:: images/04_media/image32.png
      
3-8 Decoder
~~~~~~~~~~~

The 3-8 decoder is a very commonly used device. Its truth table is shown below. Based on the values of A2, A1, A0, different results are obtained.

.. image:: images/04_media/image33.png
      
.. image:: images/04_media/image34.png
      
The code is as follows: 

.. code:: verilog

 module top(addr, decoder) ; 
 input  [2:0] addr ; 
 output reg [7:0] decoder ; 
  
 always @(addr) 
 begin 
   case(addr) 
     3'b000 : decoder = 8'b1111_1110 ; 
     3'b001 : decoder = 8'b1111_1101 ; 
     3'b010 : decoder = 8'b1111_1011 ; 
     3'b011 : decoder = 8'b1111_0111 ; 
     3'b100 : decoder = 8'b1110_1111 ; 
     3'b101 : decoder = 8'b1101_1111 ; 
     3'b110 : decoder = 8'b1011_1111 ; 
     3'b111 : decoder = 8'b0111_1111 ;    
   endcase 
 end 
    
 endmodule

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg  [2:0]  addr ; 
 wire  [7:0] decoder ;  
  
 initial 
 begin 
   addr = 3'b000 ; 
   #2000 addr =  3'b001 ; 
   #2000 addr =  3'b010 ; 
   #2000 addr =  3'b011 ; 
   #2000 addr =  3'b100 ; 
   #2000 addr =  3'b101 ; 
   #2000 addr =  3'b110 ; 
   #2000 addr =  3'b111 ; 
 end 
  
 top  
 t0(.addr(addr),.decoder(decoder)) ; 
  
 endmodule

The simulation results are as follows:

.. image:: images/04_media/image35.png
      
Tri-State Gate
~~~~~~~~~~~~~~

In FPGA applications, bidirectional IO is frequently used, which requires tri-state gates. For example, bio = en? din: 1'bz
; where en is the enable signal used to open or close the tri-state gate. The RTL diagram below implements bidirectional IO. Refer to the code. The testbench file implements the connection of two bidirectional IOs.

.. image:: images/04_media/image36.png

The code is as follows:

.. code:: verilog
      
 module top(en, din, dout, bio) ; 
 input  din  ; 
 input  en ; 
 output dout ; 
 inout bio ; 
  
 assign bio = en? din : 1'bz ; 
 assign dout = bio ; 
    
 endmodule

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg en0 ; 
 reg din0 ; 
 wire dout0 ; 
 reg en1 ; 
 reg din1 ; 
 wire dout1 ; 
 wire bio ; 
  
 initial 
 begin 
   din0 = 0 ; 
   din1 = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     din0 = ~din0 ; 
     #({$random}%100)     
 din1 = ~din1 ; 
   end 
 end 
  
 initial 
 begin 
   en0 = 0 ; 
   en1 = 1 ; 
   #100000  
   en0 = 1 ; 
   en1 = 0 ;   
 end 
  
 top  
 t0(.en(en0),.din(din0),.dout(dout0),.bi
 o(bio)) ; 
 top  
 t1(.en(en1),.din(din1),.dout(dout1),.bi
 o(bio)) ; 
  
 endmodule

The testbench file structure is shown in the figure below

.. image:: images/04_media/image37.png
      
In the simulation results below, when en0 is 0 and en1 is 1, channel 1 is open. The bidirectional IO
bio equals din1 of channel 1. Channel 1 sends data outward, and channel 0 receives data, so dout0 equals bio. When en0 is 1 and en1 is 0, channel 0 is open. The bidirectional IO
bio equals din0 of channel 0. Channel 0 sends data outward, and channel 1 receives data, so dout1 equals bio.

.. image:: images/04_media/image38.png
      
Sequential Logic
----------------

The characteristic of combinational logic circuits is that the output at any moment depends solely on the current input, independent of the circuit's previous state. Sequential logic, on the other hand, is characterized by the fact that the output at any moment depends not only on the current input signals but also on the circuit's previous state. The following analyzes typical sequential logic circuits.

D Flip-Flop
~~~~~~~~~~~~

The D flip-flop stores data on the rising or falling edge of the clock. The output is the same as the state of the input signal before the clock transition.

The code is as follows: 

.. code:: verilog

 module top(d, clk, q) ; 
 input  d  ; 
 input clk ; 
 output reg q ; 
 always @(posedge clk) 
 begin 
   q <= d ; 
 end 
    
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg d ; 
 reg clk ; 
 wire q ; 
  
 initial 
 begin 
   d = 0 ; 
   clk = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     d = ~d ; 
   end 
 end 
  
 always #10 clk = ~clk ; 
 top  t0(.d(d),.clk(clk),.q(q)) ; 
  
 endmodule

The RTL diagram is shown below

.. image:: images/04_media/image2.png
      
In the simulation results below, at time t0, d is 0, so q is also 0. At time t1, d changes to 1, so q also changes to 1. It can be seen that during one clock cycle between t0 and t1, regardless of how the input signal d changes, q remains unchanged. This means it has a storage function, and the stored value is the value of d at the clock transition edge.

.. image:: images/04_media/image39.png
      
Two-Stage D Flip-Flop
~~~~~~~~~~~~~~~~~~~~~

Software performs timing analysis based on the two-stage D flip-flop model. You can analyze the difference in output data between two D flip-flops at the same moment. The RTL diagram is as follows:

.. image:: images/04_media/image40.png
      
 The code is as follows: 

.. code:: verilog

 module top(d, clk, q, q1) ; 
 input  d  ; 
 input clk ; 
 output reg q ; 
 output reg q1 ; 
  
 always @(posedge clk) 
 begin 
   q <= d ; 
 end 
  
 always @(posedge clk) 
 begin 
   q1 <= q ; 
 end 
    
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg d ; 
 reg clk ; 
 wire q ; 
 wire q1 ; 
  
 initial 
 begin 
   d = 0 ; 
   clk = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     d = ~d ; 
   end 
 end 
  
 always #10 clk = ~clk ; 
  
 top  
 t0(.d(d),.clk(clk),.q(q),.q1(q1)) ; 
  
 endmodule


In the simulation results below, at time t0, d is 0 and q outputs 0. At time t1, q changes with d's data change, but since q was still 0 before this clock transition, q1 remains 0. At time t2, q was 1 before the clock transition, so q1 becomes 1. q1 lags behind q by one clock cycle.

.. image:: images/04_media/image41.png
      
D Flip-Flop with Asynchronous Reset
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Asynchronous reset operates independently of the clock. Once the asynchronous reset signal is active, the reset operation is triggered. This function is frequently used when writing code for signal reset and initialization. The RTL diagram is as follows:

.. image:: images/04_media/image42.png
      
The code is as follows. Note that the asynchronous reset signal must be placed in the sensitivity list. If it is active-low reset, use negedge; if it is active-high reset, use posedge.

.. code:: verilog

 module top(d, rst, clk, q) ; 
 input  d  ; 
 input rst ; 
 input clk ; 
 output reg q ; 
  
 always @(posedge clk or negedge rst) 
 begin 
   if (rst == 1'b0) 
     q <= 0 ; 
   else 
     q <= d ; 
 end 
  
 endmodule

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg d ; 
  
 reg rst ; 
 reg clk ; 
 wire q ; 
  
 initial 
 begin 
   d = 0 ; 
   clk = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     d = ~d ; 
   end 
 end 
  
 initial 
 begin 
   rst = 0 ; 
   #200 rst = 1 ; 
 end 
  
 always #10 clk = ~clk ; 
  
 top  
 t0(.d(d),.rst(rst),.clk(clk),.q(q)) ; 
  
 endmodule

In the simulation results below, before the reset signal is deasserted, although the input signal d has data changes, since the circuit is in reset state, the output q remains 0. After reset is deasserted, q operates normally.

.. image:: images/04_media/image43.png
      
D Flip-Flop with Asynchronous Reset and Synchronous Clear
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As mentioned earlier, asynchronous reset operates independently of the clock, while synchronous clear operates synchronously with the clock signal. Of course, it is not limited to synchronous clear; it can also be other synchronous operations. The RTL diagram is as follows:

.. image:: images/04_media/image44.png
      
The code is as follows. Unlike asynchronous reset, synchronous operation signals should not be placed in the sensitivity list.

.. code:: verilog

 module top(d, rst, clr, clk, q) ; 
 input  d  ; 
 input rst ; 
 input clr ; 
 input clk ; 
 output reg q ; 
  
 always @(posedge clk or negedge rst) 
 begin 
   if (rst == 1'b0) 
     q <= 0 ; 
   else if (clr == 1'b1) 
     q <= 0 ; 
   else 
     q <= d ; 
 end 
  
 endmodule 

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg d ; 
 reg rst ; 
 reg clr ; 
 reg clk ; 
 wire q ; 
  
 initial 
 begin 
   d = 0 ; 
   clk = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     d = ~d ; 
   end 
 end 
  
 initial 
 begin 
   rst = 0 ; 
   clr = 0 ; 
   #200 rst = 1 ; 
   #200 clr = 1 ; 
   #100 clr = 0 ; 
 end 
  
 always #10 clk = ~clk ; 
  
 top  
 t0(.d(d),.rst(rst),.clr(clr),.clk(clk),
 .q(q)) ; 
  
 endmodule


In the simulation results below, after the clr signal goes high, q does not clear to zero immediately. Instead, the clear operation is performed after the next rising edge of clk, meaning clr is synchronous with clk.

.. image:: images/04_media/image45.png
      
Shift Register
~~~~~~~~~~~~~~

A shift register shifts one bit to the left or right at each clock pulse. Due to the characteristics of D flip-flops, data output is synchronized with the clock edge. Its structure is as follows: at each clock edge, each D flip-flop's output q equals the previous D flip-flop's output value, thus implementing the shift function.

.. image:: images/04_media/image46.png
      
Code implementation:

.. code:: verilog

 module top(d, rst, clk, q) ; 
 input  d  ; 
 input rst ; 
 input clk ; 
 output reg [7:0] q ;

 always @(posedge clk or negedge rst) 
 begin 
   if (rst == 1'b0) 
     q <= 0 ; 
   else 
     q <= {q[6:0], d} ;  //shift left 
   //q <= {d, q[7:1]} ;  //shift right 
 end 
  
 endmodule

Testbench file:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg d ; 
  
 reg rst ; 
 reg clk ; 
 wire [7:0] q ; 
  
 initial 
 begin 
   d = 0 ; 
   clk = 0 ; 
   forever 
   begin     
     #({$random}%100) 
     d = ~d ; 
   end 
 end 
  
 initial 
 begin 
   rst = 0 ; 
   #200 rst = 1 ; 
 end 
  
 always #10 clk = ~clk ; 
  
 top 
 t0(.d(d),.rst(rst),.clk(clk),.q(q)) ; 
  
 endmodule

In the simulation results below, after reset is deasserted, data shifts left by one bit at each rising edge of clk.

.. image:: images/04_media/image47.png
      
Single-Port RAM
~~~~~~~~~~~~~~~

In single-port RAM, the write address and read address share the same address. The code is as follows, where reg [7:0] ram
[63:0] defines 64 registers with 8-bit width. addr_reg is defined to hold the read address, and the data is output after a one-cycle delay.

Code implementation:

.. code:: verilog

 module top  
 ( 
   input [7:0] data, 
   input [5:0] addr, 
   input wr, 
   input clk, 
   output [7:0] q 
 ); 
  
 reg [7:0] ram[63:0];   //declare ram 
 reg [5:0] addr_reg;    //addr register 
  
 always @ (posedge clk) 
 begin 
   if (wr)               //write 
     ram[addr] <= data; 
     
   addr_reg <= addr; 
 end 
  
 assign q = ram[addr_reg];  //read data 
 endmodule 

Testbench file:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg [7:0] data ;  
 reg [5:0] addr ;  
 reg wr ; 
 reg clk ; 
 wire [7:0] q ; 
  
 initial 
 begin 
   data = 0 ; 
   addr = 0 ; 
   wr = 1 ; 
   clk = 0 ; 
   end 
  
 always #10 clk = ~clk ; 
  
 always @(posedge clk) 
 begin 
   data <= data + 1'b1 ; 
   addr <= addr + 1'b1 ; 
 end 
  
 top  t0(.data(data), 
         .addr(addr), 
         .clk(clk), 
         .wr(wr), 
         .q(q)) ; 
 endmodule

In the simulation results below, it can be seen that the output q is consistent with the written data.

.. image:: images/04_media/image48.png
      
Simple Dual-Port RAM
~~~~~~~~~~~~~~~~~~~~

In simple dual-port RAM, the read and write addresses are independent, allowing random selection of write or read addresses and simultaneous read and write operations. The code is as follows. In the testbench file, an en signal is defined, and when it is active, the read address is sent.

Code implementation

.. code:: verilog

 module top  
 ( 
   input [7:0] data, 
   input [5:0] write_addr, 
   input [5:0] read_addr,  
   input wr, 
   input rd, 
   input clk, 
   output reg [7:0] q 
 ); 
  
 reg [7:0] ram[63:0];   //declare ram 
 reg [5:0] addr_reg;    //addr register 
  
 always @ (posedge clk) 
 begin 
   if (wr)               //write 
     ram[write_addr] <= data; 
   if (rd)               //read 
      q <= ram[read_addr]; 
 end 
  
 endmodule 

Testbench file

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg [7:0] data ;  
 reg [5:0] write_addr ; 
 reg [5:0] read_addr ;  
 reg wr ; 
 reg clk ; 
 reg rd ; 
 wire [7:0] q ; 
  
 initial 
 begin 
   data = 0 ; 
   write_addr = 0 ; 
   read_addr = 0 ; 
   wr = 0 ; 
   rd = 0 ; 
   clk = 0 ; 
   #100 wr = 1 ; 
   #20 rd = 1 ; 
 end 
 
 always #10 clk = ~clk ; 
  
 always @(posedge clk) 
 begin 
   if (wr) 
   begin 
      data <= data + 1'b1 ; 
      write_addr <= write_addr + 1'b1 ; 
      if (rd)  
        read_addr <= read_addr + 1'b1 ; 
   end 
 end 
  
 top  t0(.data(data), 
         .write_addr(write_addr), 
         .read_addr(read_addr), 
         .clk(clk), 
         .wr(wr), 
         .rd(rd), 
         .q(q)) ; 
 endmodule 

In the simulation results below, when rd is active, the read address is operated and data is read out.

.. image:: images/04_media/image49.png
      
True Dual-Port RAM
~~~~~~~~~~~~~~~~~~

True dual-port RAM has two sets of control lines and data lines, allowing two systems to perform read and write operations. The code is as follows:

Code implementation

.. code:: verilog

 module top  
 ( 
   input [7:0] data_a, data_b, 
   input [5:0] addr_a, addr_b, 
   input wr_a, wr_b, 
   input rd_a, rd_b, 
   input clk, 
   output reg [7:0] q_a, q_b 
 ); 
  
 reg [7:0] ram[63:0];   //declare ram 
  
 //Port A 
 always @ (posedge clk) 
 begin 
   if (wr_a)               //write 
     begin 
      ram[addr_a] <= data_a; 
      q_a <= data_a ; 
     end 
  if (rd_a)                    
 //read 
      q_a <= ram[addr_a]; 
 end 
  

 //Port B 
 always @ (posedge clk) 
 begin 
   if (wr_b)               //write 
     begin 
      ram[addr_b] <= data_b; 
      q_b <= data_b ; 
     end 
   if (rd_b)                    
 //read 
      q_b <= ram[addr_b]; 
 end 
  
 endmodule 

Testbench file

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg [7:0] data_a, data_b ; 
 reg [5:0] addr_a, addr_b ; 
 reg wr_a, wr_b ; 
 reg rd_a, rd_b ;  
 reg clk ; 
 wire [7:0] q_a, q_b ; 
  
 initial 
 begin 
   data_a = 0 ; 
   data_b = 0 ; 
   addr_a = 0 ; 
   addr_b = 0 ; 
   wr_a = 0 ; 
   wr_b = 0 ; 
   rd_a =  0 ; 
   rd_b = 0 ; 
   clk = 0 ; 
   #100 wr_a = 1 ; 
   #100 rd_b = 1 ; 
 end 
  
 always #10 clk = ~clk ; 
  
 always @(posedge clk) 
 begin 
   if (wr_a) 
   begin 
     data_a <= data_a + 1'b1 ; 
     addr_a <= addr_a + 1'b1 ; 
   end 
   else     
 begin 
      data_a <= 0 ; 
      addr_a <= 0 ; 
   end 
 end 
  
 always @(posedge clk) 
 begin 
   if (rd_b) 
     begin 
      addr_b <= addr_b + 1'b1 ;     
     end 
   else addr_b <= 0 ; 
  
 end 
  
 top  
 t0(.data_a(data_a), .data_b(data_b), 
    .addr_a(addr_a), .addr_b(addr_b), 
    .wr_a(wr_a), .wr_b(wr_b), 
    .rd_a(rd_a), .rd_b(rd_b), 
    .clk(clk),         
    .q_a(q_a), .q_b(q_b)) ; 
 endmodule 

The simulation results are as follows

.. image:: images/04_media/image50.png
      
Single-Port ROM
~~~~~~~~~~~~~~~

ROM is used for data storage. The ROM can be initialized using the code format below, but this method is cumbersome for large-capacity ROMs. It is recommended to use the FPGA's built-in ROM
IP core and add an initialization file.

Code implementation 

.. code:: verilog

 module top
 ( 
   input [3:0] addr, 
   input clk, 
   output reg [7:0] q  
 ); 
  
 always @(posedge clk) 
 begin 
   case(addr) 
    4'd0  : q <= 8'd15  ; 
    4'd1  : q <= 8'd24  ; 
    4'd2  : q <= 8'd100 ; 
    4'd3  : q <= 8'd78  ; 
    4'd4  : q <= 8'd98  ; 
    4'd5  : q <= 8'd105 ; 
    4'd6  : q <= 8'd86  ; 
    4'd7  : q <= 8'd254 ; 
    4'd8  : q <= 8'd76  ; 
    4'd9  : q <= 8'd35  ; 
    4'd10 : q <= 8'd120 ; 
    4'd11 : q <= 8'd85  ; 
    4'd12 : q <= 8'd37  ; 
    4'd13 : q <= 8'd19  ; 
    4'd14 : q <= 8'd22  ; 
    4'd15 : q <= 8'd67  ; 
    default: q <= 8'd0 ;
   endcase 
 end  
 
 endmodule

Testbench file

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg [3:0] addr ; 
 reg clk ; 
 wire [7:0] q ; 
  
 initial 
 begin 
   addr = 0 ; 
   clk = 0 ; 
 end 
  
 always #10 clk = ~clk ; 
  
 always @(posedge clk) 
 begin 
      addr <= addr + 1'b1 ; 
 end 
  
 top  t0(.addr(addr), 
         .clk(clk), 
         .q(q)) ; 
 endmodule 

The simulation results are as follows

.. image:: images/04_media/image51.png
      
Finite State Machine
~~~~~~~~~~~~~~~~~~~~

Finite state machines are frequently used in Verilog for handling relatively complex logic. Different states are defined, and transitions to corresponding states occur based on trigger conditions, with appropriate processing performed in each state. Finite state machines mainly use always and case statements. The following uses a four-state finite state machine as an example.

.. image:: images/04_media/image52.png
      
In the program, an 8-bit shift register is designed. In the Idle state, it checks whether the shift_start signal is high. If high, it enters the Start state. In the Start state, after a delay of 100 cycles, it enters the Run state for shift processing. If the shift_stop signal becomes active, it enters the Stop state. In the Stop state, the value of q is cleared to zero, and then it transitions back to the Idle state.

Mealy finite state machine: the output depends not only on the current state but also on the input signals. In the RTL diagram, it has connections with input signals.

.. code:: verilog

 module top  
 ( 
   input shift_start, 
   input shift_stop, 
   input rst, 
   input clk, 
   input d, 
   output reg [7:0] q  
 ); 
  
 parameter Idle  = 2'd0 ;    //Idle state 
 parameter Start = 2'd1 ;    //Start state 
 parameter Run   = 2'd2 ;    //Run state 
 parameter Stop  = 2'd3 ;    //Stop state 
   
 reg [1:0] state ;           //statement 
 reg [4:0] delay_cnt ;       //delay counter 
  
 always @(posedge clk or negedge rst) 
 begin 
   if (!rst) 
   begin 
    state <= Idle ; 
    delay_cnt <= 0 ; 
    q <= 0 ; 
    end 
   else 
   case(state) 
     Idle  : begin 
              if (shift_start) 
                 state <= Start ; 
     end 
     Start : begin 
               if (delay_cnt == 5'd99) 
               begin 
                 delay_cnt <= 0 ; 
                 state <= Run ; 
               end 
               else 
                 delay_cnt <= delay_cnt + 1'b1 ; 
             end 
     Run   : begin 
               if (shift_stop) 
                  state <= Stop ; 
               else 
                  q <= {q[6:0], d} ; 
             end 
     Stop  : begin 
               q <= 0 ; 
               state <= Idle ; 
            end 
   default: state <= Idle ; 
    endcase 
 end           
 endmodule 

Moore finite state machine: the output depends only on the current state, not on input signals. Input signals only affect state transitions, not outputs. For example, the processing of delay_cnt and q depends only on the state.

.. code:: verilog

 module top  
 ( 
   input shift_start, 
   input shift_stop, 
   input rst, 
   input clk, 
   input d, 
   output reg [7:0] q  
 ); 
  
 parameter Idle  = 2'd0 ;    //Idle state 
 parameter Start = 2'd1 ;    //Start state 
 parameter Run   = 2'd2 ;    //Run state 
 parameter Stop  = 2'd3 ;    //Stop state 
   
 reg [1:0] current_state ;           //statement 
 reg [1:0] next_state ; 
 reg [4:0] delay_cnt ;       //delay counter 
 //First part: statement transition 
 always @(posedge clk or negedge rst) 
 begin 
   if (!rst) 
    current_state <= Idle ; 
   else 
    current_state <= next_state ; 
 end 
 //Second part: combination logic, judge statement transition condition 
 always @(*) 
 begin 
   case(current_state) 
     Idle  : begin 
               if (shift_start) 
                   next_state <= Start ; 
               else 
                   next_state <= Idle ; 
     end 
     Start : begin 
               if (delay_cnt == 5'd99) 
                   next_state <= Run ; 
               else 
                   next_state <= Start ; 
             end 
     Run   : begin 
               if (shift_stop) 
                  next_state <= Stop ; 
               else 
                  next_state <= Run ; 
             end 
     Stop  :      next_state <= Idle ; 
    default:      next_state <= Idle ; 
   endcase 
 end 
 //Last part: output data 
 always @(posedge clk or negedge rst) 
 begin 
   if (!rst) 
     delay_cnt <= 0 ; 
   else if (current_state == Start) 
     delay_cnt <= delay_cnt + 1'b1 ; 
   else 
     delay_cnt <= 0 ; 
 end 
  
 always @(posedge clk or negedge rst) 
 begin 
   if (!rst) 
     q <= 0 ; 
   else if (current_state == Run) 
     q <= {q[6:0], d} ; 
   else 
     q <= 0 ; 
 end   
            
  
 endmodule

In the two programs above, two different coding styles are used. The first Mealy state machine uses a one-segment style with only one always statement, where all state transitions, state transition condition judgments, and data outputs are in a single always statement. The disadvantage is that if there are too many states, the entire program becomes lengthy. The second Moore state machine uses a three-segment style: state transition uses one always statement, state transition condition judgment uses combinational logic in one always statement, and data output uses a separate always statement. This approach is more intuitive and clear, and does not become cumbersome even with many states.

.. image:: images/04_media/image53.png
      
Mealy Finite State Machine RTL Diagram

.. image:: images/04_media/image54.png
      
Moore Finite State Machine RTL Diagram

The testbench file is as follows:

.. code:: verilog

 `timescale 1 ns/1 ns 
 module top_tb() ; 
 reg shift_start ; 
 reg shift_stop ; 
 reg rst ; 
 reg clk ; 
 reg d ; 
 wire [7:0] q ; 
  
 initial 
 begin 
   rst = 0 ;   
   clk = 0 ; 
   d = 0 ; 
   #200 rst = 1 ; 
   forever 
   begin 
     #({$random}%100) 
     d = ~d ; 
   end 
 end 
  
 initial 
 begin 
   shift_start = 0 ; 
   shift_stop = 0 ; 
   #300 shift_start = 1 ; 
   #1000 shift_start = 0 ; 
         shift_stop  = 1 ; 
   #50 shift_stop = 0 ; 
 end 
  
 always #10 clk = ~clk ; 
  
 top  t0 
 ( 
   .shift_start(shift_start), 
   .shift_stop(shift_stop), 
   .rst(rst), 
   .clk(clk), 
   .d(d), 
   .q(q)  
 ); 
 endmodule

The simulation results are as follows:

.. image:: images/04_media/image55.png
      
Summary
-------

This document introduced commonly used modules in combinational logic and sequential logic. The finite state machine is relatively complex but frequently used. We hope that everyone can deeply understand it, apply it in code, and think more about it, which will help rapidly improve your skills.

.. |image1| image:: images/04_media/image2.png
.. |image2| image:: images/04_media/image3.png
.. |image3| image:: images/04_media/image6.png
.. |image4| image:: images/04_media/image7.png
.. |image5| image:: images/04_media/image9.png
.. |image6| image:: images/04_media/image10.png
.. |image7| image:: images/04_media/image12.png
.. |image8| image:: images/04_media/image13.png
.. |image9| image:: images/04_media/image15.png
.. |image10| image:: images/04_media/image16.png
.. |image11| image:: images/04_media/image18.png
.. |image12| image:: images/04_media/image19.png
.. |image13| image:: images/04_media/image21.png
.. |image14| image:: images/04_media/image22.png
.. |image15| image:: images/04_media/image24.png
.. |image16| image:: images/04_media/image25.png
.. |image17| image:: images/04_media/image27.png
.. |image18| image:: images/04_media/image28.png