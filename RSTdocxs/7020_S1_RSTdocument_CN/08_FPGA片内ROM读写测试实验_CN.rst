FPGA On-Chip ROM Read/Write Test Experiment
=============================================

**The Vivado project for this experiment is "rom_test".**

The FPGA itself is based on SRAM architecture, and the program disappears after power-off. So how can we implement a ROM using an FPGA? We can use the internal RAM resources of the FPGA to implement a ROM, but it is not a true ROM in the traditional sense. Instead, the initialized values are written into the RAM each time the device is powered on. This experiment will introduce how to use the on-chip ROM of the FPGA and how to perform data read operations on the ROM.

Experiment Principle
--------------------

Xilinx has already provided us with ROM IP cores in VIVADO. We only need to instantiate a ROM through the IP core and read the data stored in the ROM according to the ROM read timing. In the experiment, we can use the integrated online logic analyzer (ILA) in VIVADO to observe the ROM read timing and the data read from the ROM.

Program Design
--------------

Creating the ROM Initialization File
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Since it is a ROM, we must prepare the data in advance, and then directly read the pre-stored data from the ROM when the FPGA is actually running. The on-chip ROM of Xilinx FPGA supports initialization data configuration. As shown in the figure below, we can create a file named rom_init.coe. Note that the file extension must be ".coe", and the file name can be chosen freely.

.. image:: images/08_media/image1.png
      
The content format of the ROM initialization file is very simple, as shown in the figure below. The first line defines the data format, where 16 indicates that the ROM data format is hexadecimal. From line 3 to line 34, these are the initialization data for this 32*8bit ROM. Each line of data is followed by a comma, and the last line of data ends with a semicolon.

.. image:: images/08_media/image2.png
      
After completing the rom_init.coe file, save it. Next, we will start designing and configuring the ROM IP core.

Adding the ROM IP Core
~~~~~~~~~~~~~~~~~~~~~~

Before adding the ROM IP, first create a new project named rom_test, then add the ROM IP to the project as follows:

1. Click on IP Catalog as shown in the figure below. In the interface that pops up on the right side, search for rom, find Block Memory Generator, and double-click to open it.

.. image:: images/08_media/image3.png
      
2. Change the Component Name to rom_ip. Under the Basic tab, change the Memory Type to Single Prot ROM.

.. image:: images/08_media/image4.png
      
3. Switch to the Port A Options tab. Change the ROM bit width Port A Width to 8, change the ROM depth Port A Depth to 32, set the Enable Port Type to Always, and uncheck Primitives Output Register.

.. image:: images/08_media/image5.png
      
4. Switch to the Other Options tab, check Load Init File, click Browse, and select the previously created .coe file.

.. image:: images/08_media/image6.png
      
5. Click OK, then click Generate to generate the IP core.

.. image:: images/08_media/image7.png
      
Writing the ROM Test Program
-----------------------------

The ROM program design is very simple. In the program, we only need to change the ROM address on each clock cycle, and the ROM will output the internally stored data at the current address. We instantiate an ILA to observe the changes in address and data. The instantiation and program design of the ROM IP are as follows:

.. code:: verilog

 `timescale 1ns / 1ps
 
 module rom_test(
 	input sys_clk,	//50MHz clock
 	input rst_n		//Reset, active low
     );
 
 wire [7:0] rom_data;	  //ROM read data
 reg	 [4:0] rom_addr;      //ROM input address 
 
 //Generate ROM address to read data
 always @ (posedge sys_clk or negedge rst_n)
 begin
     if(!rst_n)
         rom_addr <= 10'd0;
     else
         rom_addr <= rom_addr+1'b1;
 end        
 //Instantiate ROM
 rom_ip rom_ip_inst
 (
     .clka   (sys_clk    ),      //inoput clka
     .addra  (rom_addr   ),      //input [4:0] addra
     .douta  (rom_data   )       //output [7:0] douta
 );
 //Instantiate logic analyzer
 ila_0 ila_m0
 (
     .clk    (sys_clk),
     .probe0 (rom_addr),
 	.probe1 (rom_data)
 );
 
 endmodule

Pin Assignment

::

 ############## clock and reset define##################
 create_clock -period 20 [get_ports sys_clk]
 set_property IOSTANDARD LVCMOS33 [get_ports {sys_clk}]
 set_property PACKAGE_PIN U18 [get_ports {sys_clk}]
 
 set_property IOSTANDARD LVCMOS33 [get_ports {rst_n}]
 set_property PACKAGE_PIN N15 [get_ports {rst_n}]

Simulation
----------

The simulation results are as follows. They meet expectations. Similar to RAM data reading, the data also lags behind the address by one clock cycle.

.. image:: images/08_media/image8.png
      
Board Verification
------------------

Using address 0 as the trigger condition, we can see that the read data is consistent with the simulation results.

.. image:: images/08_media/image9.png
      