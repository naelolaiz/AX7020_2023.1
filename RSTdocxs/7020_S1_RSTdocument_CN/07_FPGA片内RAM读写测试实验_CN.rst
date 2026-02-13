FPGA On-Chip RAM Read/Write Test Experiment
=============================================

**The Vivado project for this experiment is "ram_test".**

RAM is a commonly used fundamental module in FPGAs, widely employed for data buffering, and it also serves as the basis for ROM and FIFO. This experiment will introduce how to use the internal RAM of an FPGA and perform data read/write operations on the RAM.

Experiment Principle
--------------------

Xilinx has already provided us with a RAM IP core in VIVADO. We only need to instantiate a RAM through the IP core and write/read data stored in the RAM according to the RAM's read/write timing. In the experiment, through the online logic analyzer (ILA) integrated in VIVADO, we can observe the RAM's read/write timing and the data read from the RAM.

Create Vivado Project
---------------------

Before adding the RAM IP, first create a new project called ram_test, then add the RAM IP to the project as follows:

1. Click on IP Catalog as shown in the figure below. In the interface that pops up on the right side, search for ram, find Block Memory Generator, and double-click to open it.

.. image:: images/07_media/image1.png
      
2. Change the Component Name to ram_ip. Under the Basic tab, change the Memory Type to Simple Dual Port RAM, which is a pseudo dual-port RAM. Generally speaking, "Simple Dual Port RAM" is the most commonly used, because it has two ports with independent input and output signals.

.. image:: images/07_media/image2.png
      
3. Switch to the Port A Options tab. Change the RAM bit width Port A Width to 16, which is the data width. Change the RAM depth Port A Depth to 512; depth refers to how many data entries can be stored in the RAM. Change the Enable Port Type to Always Enable.\ |image1|

4. Switch to the Port B Options tab. Change the RAM bit width Port B Width to 16, and change the Enable Port Type to Always Enable. Of course, you can also use Use ENB Pin, which acts as a read enable signal. Uncheck Primitives Output Register; its function is to add a register to the output data, which can effectively improve timing, but the read data will lag behind the address by two clock cycles. In many cases, this feature is not enabled, keeping the data lagging behind the address by one clock cycle.

.. image:: images/07_media/image4.png
      
5. In the Other Options tab, unlike ROM, there is no need to initialize the RAM data here. We can write data in the program, so keep the default configuration and click OK directly.

.. image:: images/07_media/image5.png
      
6) Click "Generate" to generate the RAM IP.

.. image:: images/07_media/image6.png
      
RAM Port Definitions and Timing
--------------------------------

The port descriptions of the Simple Dual Port RAM module are as follows:

+-----------------+-------------+-------------------------------------+
| Signal Name     | Direction   | Description                         |
+=================+=============+=====================================+
| clka            | in          | Port A clock input                  |
+-----------------+-------------+-------------------------------------+
| wea             | in          | Port A write enable                 |
+-----------------+-------------+-------------------------------------+
| addra           | in          | Port A address input                |
+-----------------+-------------+-------------------------------------+
| dina            | in          | Port A data input                   |
+-----------------+-------------+-------------------------------------+
| clkb            | in          | Port B clock input                  |
+-----------------+-------------+-------------------------------------+
| addrb           | in          | Port B address input                |
+-----------------+-------------+-------------------------------------+
| doutb           | out         | Port B data output                  |
+-----------------+-------------+-------------------------------------+

RAM data writing and reading are both performed on the rising edge of the clock. When writing data through Port A, the wea signal needs to be asserted high, while simultaneously providing the address and the data to be written. The figure below shows the timing diagram for writing data into the RAM.

.. image:: images/07_media/image7.png
      
**RAM Write Timing**

Port B cannot write data; it can only read data from the RAM. You only need to provide the address, and generally valid data can be captured in the next clock cycle.

.. image:: images/07_media/image8.png
      
**RAM Read Timing**

Writing the Test Program
------------------------

Now let's write the RAM test program. To test the RAM functionality, we write a series of consecutive data to Port A of the RAM, writing only once, and read it out from Port B, using the logic analyzer to view the data. The code is as follows:

.. code:: verilog

 `timescale 1ns / 1ps
 //////////////////////////////////////////////////////////////////////////////////
 module ram_test(
 			input clk,		          	//50MHz clock
 			input rst_n	             	//Reset signal, active low	
 		);
 
 //-----------------------------------------------------------
 reg		[8:0]  		w_addr;	   		//RAM PORTA write address
 reg		[15:0] 		w_data;	   		//RAM PORTA write data
 reg 	      		wea;	    	//RAM PORTA enable
 reg		[8:0]  		r_addr;	  	 	//RAM PORTB read address
 wire	[15:0] 		r_data;			//RAM PORTB read data
 
 //Generate RAM PORTB read address
 always @(posedge clk or negedge rst_n)
 begin
   if(!rst_n) 
 	r_addr <= 9'd0;
   else if (|w_addr)			//Bitwise OR of w_addr, not equal to 0
     r_addr <= r_addr+1'b1;
   else
 	r_addr <= 9'd0;	
 end
 
 //Generate RAM PORTA write enable signal
 always@(posedge clk or negedge rst_n)
 begin	
   if(!rst_n) 
   	  wea <= 1'b0;
   else 
   begin
      if(&w_addr) 			//All bits of w_addr are 1, 512 data entries written, writing complete
         wea <= 1'b0;                 
      else               
         wea	<= 1'b1;        //RAM write enable
   end 
 end 
 
 //Generate RAM PORTA write address and data
 always@(posedge clk or negedge rst_n)
 begin	
   if(!rst_n) 
   begin
 	  w_addr <= 9'd0;
 	  w_data <= 16'd1;
   end
   else 
   begin
      if(wea) 					//RAM write enable active
 	 begin        
 		if (&w_addr)			//All bits of w_addr are 1, 512 data entries written, writing complete
 		begin
 			w_addr <= w_addr ;	//Hold address and data values, write to RAM only once
 			w_data <= w_data ;
 		end
 		else
 		begin
 			w_addr <= w_addr + 1'b1;
 			w_data <= w_data + 1'b1;
 		end
 	 end
   end 
 end 
 
 //-----------------------------------------------------------
 //Instantiate RAM	
 ram_ip ram_ip_inst (
   .clka      (clk          ),     // input clka
   .wea       (wea          ),     // input [0 : 0] wea
   .addra     (w_addr       ),     // input [8 : 0] addra
   .dina      (w_data       ),     // input [15 : 0] dina
   .clkb      (clk          ),     // input clkb
   .addrb     (r_addr       ),     // input [8 : 0] addrb
   .doutb     (r_data       )      // output [15 : 0] doutb
 );
 
 //Instantiate ILA logic analyzer
 ila_0 ila_0_inst (
 	.clk	(clk	), 
 	.probe0	(r_data	), 
 	.probe1	(r_addr	) 
 );
 
 	
 endmodule

In order to view the data values read from the RAM in real time, we added the ILA tool here to observe the data signal and address signal of RAM PORTB. For how to generate the ILA, please refer to the "PL 'Hello World' LED Experiment".

.. image:: images/07_media/image9.png
      
The program structure is as follows:

.. image:: images/07_media/image10.png
      
Pin Binding

::

 ############## clock and reset define##################
 create_clock -period 20 [get_ports clk]
 set_property IOSTANDARD LVCMOS33 [get_ports {clk}]
 set_property PACKAGE_PIN U18 [get_ports {clk}]
 
 set_property IOSTANDARD LVCMOS33 [get_ports {rst_n}]
 set_property PACKAGE_PIN N15 [get_ports {rst_n}]

Simulation
----------

For the simulation method, refer to the "PL 'Hello World' LED Experiment". The simulation results are as follows. From the figure, it can be seen that the data written to address 1 is 0002, and in the next clock cycle, i.e., at time 2, the valid data is read out.

.. image:: images/07_media/image11.png
      
Board Verification
------------------

Generate the bitstream and download the bit file to the FPGA. Next, we will use the ILA to observe whether the data read from the RAM matches the data we initialized.

In the Waveform window, set the r_addr address to 0 as the trigger condition. We can see that r_addr continuously increments from 0 to 1ff. As r_addr changes, r_data also changes. The r_data values are exactly the 512 data entries we wrote into the RAM. It should be noted here that when a new address appears on r_addr, the corresponding r_data takes two clock cycles to appear, meaning the data appears two clock cycles later than the address, which is consistent with the simulation results.

.. image:: images/07_media/image12.png
      
.. |image1| image:: images/07_media/image3.png
      