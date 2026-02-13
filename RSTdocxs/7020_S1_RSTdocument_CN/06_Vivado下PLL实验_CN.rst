PLL Experiment in Vivado
=======================

**The Vivado project for this experiment is "pll_test".**

Many beginners are puzzled when they see that there is only one 50MHz clock input on the board — how can the clock be only 50MHz? What if we need to work at 100MHz or 150MHz?
In fact, most FPGA chips have integrated PLLs internally. Other vendors may not call them PLLs, but they have similar functional modules. Through a PLL, frequency multiplication and division can be performed to generate many other clocks. This experiment demonstrates the usage of PLLs and the method of using IP cores in Vivado by instantiating a PLL IP core.

Experiment Principle
--------------------

PLL (Phase-Locked Loop) is an important resource in FPGAs. Since a complex FPGA system often requires multiple clock signals with different frequencies and phases, the number of PLLs in an FPGA chip is an important indicator of the chip's capability. In FPGA design, the clock system is extremely important for high-speed design. A low-jitter,
low-latency system clock will increase the success rate of an FPGA design.

In this experiment, we will use a PLL to output a square wave to an expansion port on the development board (AX7020/AX7010 development board J11 PIN3) to demonstrate how to use a PLL in the Vivado software.

The 7-series FPGAs use dedicated Global and Regional IO and clock resources to manage various clock requirements in a design. Clock Management Tiles (CMTs) provide clock frequency synthesis, deskew, and jitter filtering functions.

Each CMT contains one MMCM (Mixed-Mode Clock Manager) and one PLL. As shown in the figure below, the inputs of a CMT can be BUFR, IBUFG, BUFG, GT, BUFH, or local routing (not recommended). The outputs need to be connected to BUFG or BUFH before use.

.. image:: images/06_media/image1.png
      
-  Mixed-Mode Clock Manager (MMCM)

The MMCM is used to generate different clock signals that have a defined phase and frequency relationship with a given input clock. The MMCM provides extensive and powerful clock management features.

The internal functional block diagram of the MMCM is shown below:

.. image:: images/06_media/image2.png
      
-  Digital Phase-Locked Loop (PLL)

The Phase-Locked Loop (PLL) is mainly used for frequency synthesis. A single PLL can generate multiple clock signals from one input clock signal.

The internal functional block diagram of the PLL is shown below:

.. image:: images/06_media/image3.png
      
To learn more about clock resources, it is recommended to read the Xilinx document "7 Series FPGAs Clocking Resources User Guide".

Creating the Vivado Project
---------------------------

In this experiment, we will demonstrate how to use the Xilinx PLL IP core to generate clocks with different frequencies, and output one of the clocks to an external FPGA IO. Below are the detailed steps of the program design.

1) Create a new project called pll_test, and click on IP Catalog under the Project Manager interface.

.. image:: images/06_media/image4.png
      
2) In the IP Catalog interface, select Clocking Wizard under FPGA Features and Design\\Clocking, and double-click to open the configuration interface.

.. image:: images/06_media/image5.png
      
3) By default, the name of this Clocking Wizard is clk_wiz_0, and we will not modify it here. In the first interface, Clocking Options, we select the PLL resource and set the input clock frequency to 50MHz.

.. image:: images/06_media/image6.png
      
4) In the Output Clocks interface, select four clock outputs clk_out1 through clk_out4, with frequencies of 200MHz, 100MHz, 50MHz, and 25MHz respectively. You can also set the output clock phase here; we will not change it and keep the default phase. Click OK to finish.

.. image:: images/06_media/image7.png
      
5) Click the Generate button in the pop-up dialog to generate the PLL IP design files.

.. image:: images/06_media/image8.png
      
6) At this point, a clk_wiz_0.xci IP will be automatically added to our pll_test project. Users can double-click it to modify the configuration of this IP.

.. image:: images/06_media/image9.png
      
Select the IP Sources tab, then double-click to open the clk_wiz_0.veo file. This file provides the instantiation template for this IP. We just need to copy the content within the box to our Verilog program to instantiate the IP.

.. image:: images/06_media/image10.png
      
7) Next, we write a top-level design file to instantiate this PLL IP. The pll_test.v code is as follows. Note that the PLL reset is active-high, meaning the PLL will remain in reset state and will not work when the reset is high — this is a point that many beginners overlook. Here we bind rst_n to a button, and since the button is active-low reset, we need to invert it when connecting to the PLL reset.

.. code:: verilog

 `timescale 1ns / 1ps
 module pll_test(
  input      sys_clk,            //system clock 50Mhz on board
 input       rst_n,             //reset ,low active
 output      clk_out           //pll clock output J8_Pin3
 
     );
     
 wire        locked;
 
 /////////////////////PLL IP call////////////////////////////
 clk_wiz_0 clk_wiz_0_inst
    (// Clock in ports
     .clk_in1(sys_clk),            // IN 50Mhz
     // Clock out ports
     .clk_out1(),                // OUT 200Mhz
     .clk_out2(),               // OUT 100Mhz
     .clk_out3(),              // OUT 50Mhz
     .clk_out4(clk_out),    // OUT 25Mhz 
     // Status and control signals 
     .reset(~rst_n),        // pll reset, high-active
     .locked(locked));     // OUT
 
 
 endmodule

In the program, we first instantiate clk_wiz_0, feeding the single-ended 50MHz clock signal into clk_in1 of clk_wiz_0, and assigning the clk_out4 output to clk_out.

**Note: The purpose of instantiation is to call the instantiated module in the upper-level module to implement the code functionality. In Verilog, the format for instantiating signals is as follows: the module name must match the name of the module being instantiated, such as clk_wiz_0 in the program, and the module signal names must also match, such as clk_in1, clk_out1, clk_out2, etc. The connection signals are passed between the TOP program and the modules. Connection signals between modules must not conflict with each other; otherwise, compilation errors will occur.**

.. image:: images/06_media/image11.png
      
1) After saving the project, pll_test automatically becomes the top file, and clk_wiz_0 becomes a sub-module of the pll_test file.

.. image:: images/06_media/image12.png
      
9) Add an XDC pin constraint file pll.xdc to the project. Refer to the "PL 'Hello World' LED Experiment" for the method of adding it, or you can directly copy the following content. Then compile and generate the bitstream.

::

 ############## clock and reset define##################
 create_clock -period 20 [get_ports sys_clk]
 set_property IOSTANDARD LVCMOS33 [get_ports {sys_clk}]
 set_property PACKAGE_PIN U18 [get_ports {sys_clk}]
 
 set_property IOSTANDARD LVCMOS33 [get_ports {rst_n}]
 set_property PACKAGE_PIN N15 [get_ports {rst_n}]
 ############## pll output define  J11 PIN3##################
 set_property IOSTANDARD LVCMOS33 [get_ports clk_out]
 set_property PACKAGE_PIN F17 [get_ports clk_out]

Simulation
----------

Add a vtf_pll_test simulation file. After running, the PLL's lock signal will go high, indicating that the PLL IP phase-locked loop has completed initialization. The clk_out will have a clock signal output at a frequency of half the input clock frequency, which is 25MHz. Refer to the "PL 'Hello World' LED Experiment" for the simulation method.

.. image:: images/06_media/image13.png
      
Board Verification
------------------

Compile the project and generate the pll_test.bit file, then download the bit file to the FPGA. Next, we can use an oscilloscope to measure the output clock waveform.

Connect the ground wire of the oscilloscope probe to the ground on the development board (AX7020/AX7010 development board J11 PIN1), and connect the signal end to the AX7020 development board J11 PIN3 (be careful when measuring to avoid the oscilloscope probe touching other pins, which could cause a short circuit between power and ground).

At this point, we can see the 25MHz clock waveform on the oscilloscope. The waveform amplitude is 3.3V with a 1:1 duty cycle. The waveform is displayed as shown below:

.. image:: images/06_media/image14.jpeg
      
If you want to output waveforms at other frequencies, you can change the clock output to clk_out2, clk_out3, or clk_out4 of clk_wiz_0. You can also modify the frequency of clk_out4 of clk_wiz_0 to your desired frequency. Note that since the clock output is derived from the multiplication and division coefficients applied by the PLL to the input clock signal, not all clock frequencies can be precisely generated by the PLL. However, the PLL will automatically calculate the closest achievable output clock frequency for you.

Additionally, note that if your oscilloscope has insufficient bandwidth and sampling rate, it may cause significant attenuation of the high-frequency components when measuring high-frequency clock signals, resulting in lower measured waveform amplitude.
