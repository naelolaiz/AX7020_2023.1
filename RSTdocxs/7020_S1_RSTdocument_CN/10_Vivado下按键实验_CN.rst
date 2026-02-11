Key Press Experiment in Vivado
================================

**The Vivado project for this experiment is "key_test".**

Keys are the most commonly used and simplest peripherals in FPGA design. In this chapter, through a key detection experiment, we verify whether the key functions on the development board work properly, understand the relationship between hardware description languages and FPGA, and learn how to use Vivado
RTL ANALYSIS.

Key Hardware Circuit
--------------------

.. image:: images/10_media/image1.png
      
AX7020/AX7010 Development Board Key Circuit

As shown in the figure, when the key is released, the signal is high; when the key is pressed, the signal is low.

.. image:: images/10_media/image2.png
      
AX7020/AX7010 Development Board LED Circuit

For the LED part, a low level turns the LED on, and a high level turns it off.

Program Design
--------------

This program is not designed to be very complex. Through simple hardware description language, we can understand the relationship between HDL and FPGA hardware. First, we pass the key input through two stages of D flip-flops. The signal passing through a D flip-flop is latched at the rising edge of the D flip-flop clock input and then sent to the output.

.. image:: images/10_media/image3.png

Before coding in hardware description language, we have already completed the hardware design. This is a normal development process. With a hardware design concept, the design can be completed either through schematic drawing or through Verilog HDL or VHDL. The choice of tool depends on the complexity of the design and familiarity with a particular language.

Creating the Vivado Project
---------------------------

1) First, create a key test project, add the Verilog test code, and complete the process of compilation and pin assignment.

.. image:: images/10_media/image4.png


.. code:: verilog

 `timescale 1ns / 1ps
 module key_test
 (
 	input                 clk,       //system clock 50Mhz on board
 	input [3:0]           key,       //input four key signal,when the keydown,the value is 0
 	output[3:0]           led        //LED display ,when the siganl low,LED lighten
 );
 
 reg[3:0] led_r;           //define the first stage register , generate four D Flip-flop 
 reg[3:0] led_r1;          //define the second stage register ,generate four D Flip-flop
 always@(posedge clk)
 begin
 	led_r <=  key;        //first stage latched data
 end
 
 always@(posedge clk)
 begin
 	led_r1 <= led_r;      //second stage latched data
 end
 
 assign led = led_r1;
 
 endmodule

1) We can use the RTL ANALYSIS tool to view the design.

.. image:: images/10_media/image5.png
      
3) Analyzing the RTL diagram, we can see that the first-stage D flip-flop is connected to the key input, and the second stage is directly connected to the output, which is consistent with the expected design.

.. image:: images/10_media/image6.png
      
On-Board Verification
---------------------

After downloading the Bit file to the development board, the "PL LED1", "PL LED2", "PL LED3", and "PL LED4" on the development board are all in the off state. When key "PL KEY1" is pressed, "PL LED1" lights up; when key "PL KEY2" is pressed, "PL LED2" lights up; when key "PL KEY3" is pressed, "PL LED3" lights up; when key "PL KEY4" is pressed, "PL LED4" lights up.
