Custom IP Experiment
====================

**The Vivado project for this experiment is "custom_pwm_ip".**

Xilinx officially provides many IP cores, which can be viewed in the IP Catalog of Vivado.
When building their own systems, users cannot rely solely on the free IP cores provided by Xilinx. In many cases, it is necessary to create custom user IP cores. Creating your own IP cores has many benefits, such as customized system design; design reuse, with the ability to add licenses to IP cores and
provide them to others for a fee; and simplifying system design while reducing design time. When designing IP cores with the ZYNQ system, the most common approach is to use the AXI bus to connect the PS to the IP cores in the PL section. This experiment will introduce how to build an AXI bus type IP core in Vivado. This IP core is used to generate a PWM signal to control the LED on the development board, creating a breathing light effect.

PWM Introduction
----------------

We often use PWM to control LEDs, buzzers, etc., by adjusting the duty cycle of the pulse to adjust the brightness of the LED.

A PWM module we have used in other development boards is as follows:

.. code:: verilog

 //////////////////////////////////////////////////////////////////////////////////
 //                                                                              //
 //                                                                              //
 //  Author: meisq                                                               //
 //          msq@qq.com                                                          //
 //          ALINX(shanghai) Technology Co.,Ltd                                  //
 //          heijin                                                              //
 //     WEB: http://www.alinx.cn/                                                //
 //     BBS: http://www.heijin.org/                                              //
 //                                                                              //
 //////////////////////////////////////////////////////////////////////////////////
 //                                                                              //
 // Copyright (c) 2017,ALINX(shanghai) Technology Co.,Ltd                        //
 //                    All rights reserved                                       //
 //                                                                              //
 // This source file may be used and distributed without restriction provided    //
 // that this copyright statement is not removed from the file and that any      //
 // derivative work contains the original copyright notice and the associated    //
 // disclaimer.                                                                  //
 //                                                                              //
 //////////////////////////////////////////////////////////////////////////////////
 
 //================================================================================
 //   Description:  pwm model
 //   pwm out period = frequency(pwm_out) * (2 ** N) / frequency(clk);
 //
 //================================================================================
 //  Revision History:
 //  Date          By            Revision    Change Description
 //--------------------------------------------------------------------------------
 //  2017/5/3     meisq          1.0         Original
 //********************************************************************************/
 `timescale 1ns / 1ps
 module ax_pwm
 #(
 parameter N = 32 //pwm bit width 
 )
 (
     input         clk,
     input         rst,
     input[N - 1:0]period,
     input[N - 1:0]duty,
     output        pwm_out 
     );
  
 reg[N - 1:0] period_r;
 reg[N - 1:0] duty_r;
 reg[N - 1:0] period_cnt;
 reg pwm_r;
 assign pwm_out = pwm_r;
 always@(posedge clk or posedge rst)
 begin
     if(rst==1)
     begin
         period_r <= { N {1'b0} };
         duty_r <= { N {1'b0} };
     end
     else
     begin
         period_r <= period;
         duty_r   <= duty;
     end
 end
 
 always@(posedge clk or posedge rst)
 begin
     if(rst==1)
         period_cnt <= { N {1'b0} };
     else
         period_cnt <= period_cnt + period_r;
 end
 
 always@(posedge clk or posedge rst)
 begin
     if(rst==1)
     begin
         pwm_r <= 1'b0;
     end
     else
     begin
         if(period_cnt >= duty_r)
             pwm_r <= 1'b1;
         else
             pwm_r <= 1'b0;
     end
 end
 
 endmodule

As you can see, this PWM module requires 2 parameters "period" and "duty" to control the frequency and duty cycle. "period" is the step value, which is the value the counter adds each cycle. Duty is the duty cycle value. We need to design some registers to control these parameters, which requires the use of the AXI bus, where the PS reads and writes registers through the AXI bus.

PWM Frequency = :math:`\frac{period}{2\hat{}N} \times clk frequency` (unit: Hz)

PWM Duty Cycle = 1 - :math:`\frac{duty + 1}{2\hat{}N}`

Vivado Project Setup
--------------------

Save the "ps_hello" project as a new project named "custom_pwm_ip"

Create Custom IP
~~~~~~~~~~~~~~~~

1) Click the menu "Tools->Create and Package IP..."

.. image:: images/07_media/image1.png
      
2) Select "Next"

.. image:: images/07_media/image2.png
      
3) Select to create a new AXI4 peripheral

.. image:: images/07_media/image3.png
      
4) Enter "ax_pwm" for the name, "alinx pwm" for the description, and then select a suitable location to store the IP

.. image:: images/07_media/image4.png
      
5) The following parameters can specify the interface type, number of registers, etc. No modification is needed here; use the AXI Lite Slave interface with 4 registers.

.. image:: images/07_media/image5.png
      
6) Click "Finish" to complete the IP creation

.. image:: images/07_media/image6.png
      
7) The newly created IP can be seen in the "IP Catalog"

.. image:: images/07_media/image7.png
      
8) At this point, the IP only has simple register read/write functionality. We need to modify the IP. Select the IP, right-click "Edit in IP Packager"

.. image:: images/07_media/image8.png
      
9) A dialog box pops up where you can fill in the project name and path. Use the defaults here and click "OK"

.. image:: images/07_media/image9.png
      
10) Vivado opens a new project

.. image:: images/07_media/image10.png
      
11) Add the core code for the PWM functionality

.. image:: images/07_media/image11.png
      
12) When adding code, select to copy the code to the IP directory

.. image:: images/07_media/image12.png
      
13) Modify "ax_pwm_v1_0.v" to add a PWM output port

.. image:: images/07_media/image13.png
      
14) Modify "ax_pwm_v1_0.v" to add the PWM port instantiation in the instantiation of "ax_pwm_V1_0_S00_AXI"

.. image:: images/07_media/image14.png
      
15) Modify the "ax_pwm_v1_0_s00_AXI.v" file to add the PWM port. This file is the core code implementing the AXI4 Lite Slave

.. image:: images/07_media/image15.png
      
16) Modify the "ax_pwm_v1_0_s00_AXI.v" file to instantiate the PWM core functionality code, using registers slv_reg0 and slv_reg1 to control the PWM module parameters.

.. image:: images/07_media/image16.png
      
17) Double-click the "component.xml" file

.. image:: images/07_media/image17.png
      
18) In the "File Groups" option, click "Merge changes from File Groups Wizard"

.. image:: images/07_media/image18.png
      
19) In the "Customization Parameters" option, click "Merge changes from Customization Parameters Wizard"

.. image:: images/07_media/image19.png
      
20) Click "Re-Package IP" to complete the IP modification

.. image:: images/07_media/image20.png
      
Add Custom IP to Project
~~~~~~~~~~~~~~~~~~~~~~~~

1) Search for "pwm" and add "ax_pwm_v1.0"

.. image:: images/07_media/image21.png
      
2) Click "Run Connection Automation"

.. image:: images/07_media/image22.png
      
3) Export the PWM port

.. image:: images/07_media/image23.png
      
.. image:: images/07_media/image24.png
      
4) Save the design and Generate Output Products

.. image:: images/07_media/image25.png
      
5) Add an XDC file to assign pins, assigning the pwm_0 output port to PL LED1 to create a breathing light

::

 set_property IOSTANDARD LVCMOS33 [get_ports pwm_0]
 set_property PACKAGE_PIN M14 [get_ports pwm_0]

.. image:: images/07_media/image26.png
      
1) Compile to generate the bit file and export hardware

.. image:: images/07_media/image27.png
         
Vitis Software Development and Debugging
-----------------------------------------

1) Launch Vitis, create a new APP, and select the "Hello World" template

.. image:: images/07_media/image28.png
            
2) The previous examples all used Xilinx IPs, for which Xilinx mostly provides a set of APIs. For this custom IP, we need to develop our own. First, let's look at the resources in the APP directory. You can find an ax_pwm.h file, which contains macro definitions for reading and writing the custom IP registers

.. image:: images/07_media/image29.png
      
3) Find the "xparameters.h" file in the BSP. This is a very important file where you can find the register base address of the custom IP.

.. image:: images/07_media/image30.png
      
4) With the register read/write macros and the base address of the custom IP, we can start writing code to test the custom IP. We first write to register AX_PWM_S00_AXI_SLV_REG0_OFFSET to control the PWM output frequency, then write to register AX_PWM_S00_AXI_SLV_REG1_OFFSET to control the PWM output duty cycle.

.. code:: c

 #include <stdio.h>
 #include "platform.h"
 #include "xil_printf.h"
 #include "ax_pwm.h"
 #include "xil_io.h"
 #include "xparameters.h"
 #include "sleep.h"
 
 unsigned int duty;
 
 int main()
 {
     init_platform();
 
     print("Hello World\n\r");
 
 //pwm out period = frequency(pwm_out) * (2^N) / frequency(clk);
 AX_PWM_mWriteReg(XPAR_AX_PWM_0_S00_AXI_BASEADDR, AX_PWM_S00_AXI_SLV_REG0_OFFSET, 17179);//200hz
 //duty = (2^N) * (1 - (duty cycle)) - 1
 while (1) {
 for (duty = 0x8fffffff; duty < 0xffffffff; duty = duty + 100000) {
 AX_PWM_mWriteReg(XPAR_AX_PWM_0_S00_AXI_BASEADDR, AX_PWM_S00_AXI_SLV_REG1_OFFSET, duty);
 usleep(100);
 }
 }
 
     cleanup_platform();
     return 0;
 }

1) By running the code, we can see that PL LED1 displays a breathing light effect.

2) Through debugging, let's examine the registers

.. image:: images/07_media/image31.png
      
7) Enter the debug state, press "F6" for single-step execution.

.. image:: images/07_media/image32.png
      
8) The "Memory" window can be viewed through the menu

.. image:: images/07_media/image33.png
      
9) Add a monitor address "0x43c00000"

.. image:: images/07_media/image34.png
      
.. image:: images/07_media/image35.png
      
10) Single-step execution, observe the changes

.. image:: images/07_media/image36.png
      
Experiment Summary
------------------

Through this experiment, we have mastered more Vitis debugging techniques and the core content of ARM + FPGA development, which is the data interaction between ARM and FPGA.
