FPGA On-Chip FIFO Read/Write Test Experiment
=============================================

**The Vivado project for this experiment is "fifo_test".**

FIFO is a very important module in FPGA applications, widely used for data buffering, cross-clock domain data processing, and more. Mastering FIFO is key to FPGA design, and the ability to use FIFO flexibly is an essential skill for any FPGA engineer. This chapter mainly introduces how to perform read/write tests using the FIFO IP provided by XILINX.

Experiment Principle
--------------------

FIFO: First In, First Out means that data written first is read out first, and data written later is read out later. Xilinx has already provided the FIFO IP core in VIVADO. We only need to instantiate a FIFO through the IP core and write to or read from the FIFO according to its read/write timing.

In fact, FIFO is built on top of RAM with many additional features. The typical structure of a FIFO is shown below. It is mainly divided into read and write sections, along with status signals such as empty and full signals, as well as data count status signals. The biggest difference from RAM is that FIFO has no address lines and cannot perform random address data reads. What is random data reading? It means being able to read data at any arbitrary address. FIFO, on the other hand, does not support random reads, which has the advantage of not requiring frequent address line control.

.. image:: images/09_media/image1.png
      
Although the user cannot see the address lines, there are still address operations inside the FIFO to control the RAM read/write interface. The address operations during read and write are shown in the figure below, where the depth value represents the maximum number of data entries a FIFO can store. In the initial state, both read and write addresses are 0. After writing one data entry to the FIFO, the write address increments by 1. After reading one data entry from the FIFO, the read address increments by 1. At this point, the FIFO status is empty, because one data entry was written and one was read out.

.. image:: images/09_media/image2.png
      
You can think of a FIFO as a water tank, where the write channel adds water and the read channel drains water. If water is continuously added and drained, and the adding speed is faster than the draining speed, then the FIFO will eventually become full. If water continues to be added when it is already full, it will overflow. If the draining speed is faster than the adding speed, the FIFO will eventually become empty. Therefore, managing the timing and speed of adding and draining water to ensure the tank always has water is a challenging task. This involves checking the empty and full status and choosing the right time to write or read data.

According to the read and write clocks, FIFOs can be classified as synchronous FIFO (same read and write clocks) and asynchronous FIFO (different read and write clocks). Synchronous FIFO control is relatively simple and will not be discussed further. This experiment mainly introduces asynchronous FIFO control, where the read clock is 75MHz and the write clock is 100MHz. In the experiment, we will use the integrated logic analyzer (ILA) in VIVADO to observe the FIFO read/write timing and the data read from the FIFO.

Creating the Vivado Project
----------------------------

Adding the FIFO IP Core
~~~~~~~~~~~~~~~~~~~~~~~~

Before adding the FIFO IP, first create a new project named fifo_test, then add the FIFO IP to the project as follows:

1. Click on IP Catalog as shown in the figure below. In the interface that appears on the right side, search for fifo, find FIFO Generator, and double-click to open it.

.. image:: images/09_media/image3.png
      
2. In the configuration page that pops up, you can choose whether the read and write clocks are separate or shared. Generally, we use FIFO for data buffering, and the clock speeds on both sides are usually different. Therefore, independent clocks are the most commonly used option. Here we select "Independent Clocks Block RAM", then click "Next" to go to the next configuration page.

.. image:: images/09_media/image4.png
      
3. Switch to the Native Ports tab, select a data width of 16; set the FIFO depth to 512. In actual use, you can set these values according to your needs. Read Mode has two options: one is Standard FIFO, which is the commonly seen FIFO where data lags behind the read signal by one clock cycle; the other is First Word Fall Through, a data pre-fetch mode, abbreviated as FWFT mode. In this mode, the FIFO pre-fetches a data entry, so when the read signal is valid, the corresponding data is also valid. We will first conduct the Standard FIFO experiment.

.. image:: images/09_media/image5.png
      
4. Switch to the Data Counts tab, enable Write Data Count (how many data entries have been written to the FIFO) and Read Data Count (how many data entries are available to read from the FIFO), so we can monitor the amount of data inside the FIFO through these two values. Click OK, then Generate to generate the FIFO IP.

.. image:: images/09_media/image6.png
      
FIFO Port Definitions and Timing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

+------------------+-------+-------------------------------------------+
| Signal Name      | Dir   | Description                               |
+==================+=======+===========================================+
| rst              | in    | Reset signal, active high                 |
+------------------+-------+-------------------------------------------+
| wr_clk           | in    | Write clock input                         |
+------------------+-------+-------------------------------------------+
| rd_clk           | in    | Read clock input                          |
+------------------+-------+-------------------------------------------+
| din              | in    | Write data                                |
+------------------+-------+-------------------------------------------+
| wr_en            | in    | Write enable, active high                 |
+------------------+-------+-------------------------------------------+
| rd_en            | in    | Read enable, active high                  |
+------------------+-------+-------------------------------------------+
| dout             | out   | Read data                                 |
+------------------+-------+-------------------------------------------+
| full             | out   | Full signal                               |
+------------------+-------+-------------------------------------------+
| empty            | out   | Empty signal                              |
+------------------+-------+-------------------------------------------+
| rd_data_count    | out   | Number of data entries available to read   |
+------------------+-------+-------------------------------------------+
| wr_data_count    | out   | Number of data entries written             |
+------------------+-------+-------------------------------------------+

Both FIFO data writing and reading operate on the rising edge of the clock. When the wr_en signal is high, data is written to the FIFO. When the almost_full signal is active, it indicates that only one more data entry can be written to the FIFO. Once one data entry is written, the full signal goes high. If wr_en remains active when full is asserted, meaning data continues to be written to the FIFO, the overflow signal becomes active, indicating an overflow.

.. image:: images/09_media/image7.png
      
**Standard FIFO Write Timing**

When the rd_en signal is high, data is read from the FIFO, and the data becomes valid on the next clock cycle. The valid signal indicates valid data. almost_empty indicates that there is one more data entry to read. When one more data entry is read, the empty signal becomes active. If reading continues, the underflow signal becomes active, indicating an underflow, and the read data at this point is invalid.

.. image:: images/09_media/image8.png
      
**Standard FIFO Read Timing**

From the FWFT mode read timing diagram, it can be seen that when the rd_en signal is active, the valid data D0 is already prepared and valid on the data bus, without being delayed by one more clock cycle. This is the key difference from the Standard FIFO.

.. image:: images/09_media/image9.png
      
**FWFT FIFO Read Timing**

For detailed information about FIFO, please refer to the pg057 document, which can be downloaded from the Xilinx official website.

FIFO Test Program Development
------------------------------

We design based on an asynchronous FIFO, using a PLL to generate two clocks of 100MHz and 75MHz for the write clock and read clock respectively, meaning the write clock frequency is higher than the read clock frequency.

.. code:: verilog

 `timescale 1ns / 1ps
 module fifo_test
 	(
 		input 		clk,		         //50MHz clock
 		input 		rst_n	             //Reset signal, active low	
 	);
 
 
 reg	 [15:0] 		w_data			;	   		//FIFO write data
 wire      			wr_en			;	   		//FIFO write enable
 wire      			rd_en			;	   		//FIFO read enable
 wire [15:0] 		r_data			;			//FIFO read data
 wire       			full			;  			//FIFO full signal 
 wire       			empty			;  			//FIFO empty signal 
 wire [8:0]  		rd_data_count	;  			//Number of data entries available to read	
 wire [8:0]  		wr_data_count	;  			//Number of data entries written
 	
 wire				clk_100M 		;			//PLL generated 100MHz clock
 wire				clk_75M 		;			//PLL generated 75MHz clock
 wire				locked 			;			//PLL lock signal, can be used as system reset, high level indicates locked
 wire				fifo_rst_n 		;			//FIFO reset signal, active low
 
 wire				wr_clk 			;			//Write FIFO clock
 wire				rd_clk 			;			//Read FIFO clock
 reg	[7:0]			wcnt 			;			//Write FIFO post-reset wait counter
 reg	[7:0]			rcnt 			;			//Read FIFO post-reset wait counter
 
 //Instantiate PLL, generate 100MHz and 75MHz clocks
 clk_wiz_0 fifo_pll
  (
   // Clock out ports
   .clk_out1(clk_100M),     	 	// output clk_out1
   .clk_out2(clk_75M),    		// output clk_out2
   // Status and control signals
   .reset(~rst_n), 			 	// input reset
   .locked(locked),       		// output locked
   // Clock in ports
   .clk_in1(clk)					// input clk_in1
   );      			
 
 assign fifo_rst_n 	= locked	;	//Assign PLL LOCK signal to FIFO reset signal
 assign wr_clk 		= clk_100M 	;	//Assign 100MHz clock to write clock
 assign rd_clk 		= clk_75M 	;	//Assign 75MHz clock to read clock
 
 
 /* Write FIFO state machine */
 localparam      W_IDLE      = 1	;
 localparam      W_FIFO     	= 2	; 
 
 reg[2:0]  write_state;
 reg[2:0]  next_write_state;
 
 always@(posedge wr_clk or negedge fifo_rst_n)
 begin 
 	if(!fifo_rst_n)
 		write_state <= W_IDLE;
 	else
 		write_state <= next_write_state;
 end
 
 always@(*)
 begin
 	case(write_state)
 		W_IDLE:
 			begin
 				if(wcnt == 8'd79)               //Wait for a certain time after reset, 60 cycles of the slowest clock in safety circuit mode
 					next_write_state <= W_FIFO;
 				else
 					next_write_state <= W_IDLE;
 			end
 		W_FIFO:
 			next_write_state <= W_FIFO;			//Stay in write FIFO state
 		default:
 			next_write_state <= W_IDLE;
 	endcase
 end
 //In IDLE state, i.e., after reset, the counter counts
 always@(posedge wr_clk or negedge fifo_rst_n)
 begin 
 	if(!fifo_rst_n)
 		wcnt <= 8'd0;
 	else if (write_state == W_IDLE)
 		wcnt <= wcnt + 1'b1 ;
 	else
 		wcnt <= 8'd0;
 end
 //In write FIFO state, write data to FIFO if not full
 assign wr_en = (write_state == W_FIFO) ? ~full : 1'b0; 
 //When write enable is active, increment write data value
 always@(posedge wr_clk or negedge fifo_rst_n)
 begin
 	if(!fifo_rst_n)
 		w_data <= 16'd1;
 	else if (wr_en)
 		w_data <= w_data + 1'b1;
 end
 
 /* Read FIFO state machine */
 
 localparam      R_IDLE      = 1	;
 localparam      R_FIFO     	= 2	; 
 reg[2:0]  read_state;
 reg[2:0]  next_read_state;
 
 ///Generate FIFO read data
 always@(posedge rd_clk or negedge fifo_rst_n)
 begin
 	if(!fifo_rst_n)
 		read_state <= R_IDLE;
 	else
 		read_state <= next_read_state;
 end
 
 always@(*)
 begin
 	case(read_state)
 		R_IDLE:
 			begin
 				if (rcnt == 8'd59)             	//Wait for a certain time after reset, 60 cycles of the slowest clock in safety circuit mode
 					next_read_state <= R_FIFO;
 				else
 					next_read_state <= R_IDLE;
 			end
 		R_FIFO:	
 			next_read_state <= R_FIFO ;			//Stay in read FIFO state
 		default:
 			next_read_state <= R_IDLE;
 	endcase
 end
 
 //In IDLE state, i.e., after reset, the counter counts
 always@(posedge rd_clk or negedge fifo_rst_n)
 begin 
 	if(!fifo_rst_n)
 		rcnt <= 8'd0;
 	else if (write_state == W_IDLE)
 		rcnt <= rcnt + 1'b1 ;
 	else
 		rcnt <= 8'd0;
 end
 //In read FIFO state, read data from FIFO if not empty
 assign rd_en = (read_state == R_FIFO) ? ~empty : 1'b0; 
 
 //-----------------------------------------------------------
 //Instantiate FIFO
 fifo_ip fifo_ip_inst 
 (
   .rst            (~fifo_rst_n    	),   // input rst
   .wr_clk         (wr_clk          	),   // input wr_clk
   .rd_clk         (rd_clk          	),   // input rd_clk
   .din            (w_data       	),   // input [15 : 0] din
   .wr_en          (wr_en        	),   // input wr_en
   .rd_en          (rd_en        	),   // input rd_en
   .dout           (r_data       	),   // output [15 : 0] dout
   .full           (full         	),   // output full
   .empty          (empty        	),   // output empty
   .rd_data_count  (rd_data_count	),   // output [8 : 0] rd_data_count
   .wr_data_count  (wr_data_count	)    // output [8 : 0] wr_data_count
 );
 
 //Write channel logic analyzer
 ila_m0 ila_wfifo (
 	.clk(wr_clk), 
 	.probe0(w_data), 	
 	.probe1(wr_en), 	
 	.probe2(full), 		
 	.probe3(wr_data_count)
 );
 //Read channel logic analyzer
 ila_m0 ila_rfifo (
 	.clk(rd_clk), 
 	.probe0(r_data), 	
 	.probe1(rd_en), 	
 	.probe2(empty), 		
 	.probe3(rd_data_count)
 );
  	
 endmodule

In the program, the PLL lock signal is used as the FIFO reset, and the 100MHz clock is assigned to the write clock while the 75MHz clock is assigned to the read clock.

.. image:: images/09_media/image10.png
      
One thing to note is that the FIFO is configured by default to use the safety circuit, which ensures that input signals reaching the internal RAM are synchronized. In this case, after an asynchronous reset, you need to wait for 60 cycles of the slowest clock. In this experiment, that means 60 cycles of the 75MHz clock, so the 100MHz clock needs approximately (100/75) x 60 = 80 cycles.

.. image:: images/09_media/image11.png
      
Therefore, in the write state machine, it waits for 80 cycles before entering the write FIFO state.

.. image:: images/09_media/image12.png
      
In the read state machine, it waits for 60 cycles before entering the read state.

.. image:: images/09_media/image13.png
      
If the FIFO is not full, data is continuously written to the FIFO.

.. image:: images/09_media/image14.png
      
If the FIFO is not empty, data is continuously read from the FIFO.

.. image:: images/09_media/image15.png
      
Two logic analyzers are instantiated, connected to the write channel and read channel signals respectively.

.. image:: images/09_media/image16.png
      
Simulation
----------

The following shows the simulation results. It can be seen that after the write enable wr_en becomes active, data writing begins with an initial value of 0001. From the start of writing to when empty becomes deasserted, a certain number of clock cycles are needed because internal synchronization processing is required. After empty is deasserted, data reading begins, and the read data lags behind rd_en by one clock cycle.

.. image:: images/09_media/image17.png
      
Later, it can be seen that if the FIFO becomes full, according to the program design, no more data is written to the FIFO, and wr_en goes low. Why does it become full? Because the write clock is faster than the read clock. If the write clock and read clock are swapped, meaning the read clock is faster, then a read-empty situation will occur. You can try this yourself.

.. image:: images/09_media/image18.png
      
If the FIFO Read Mode is changed to First Word Fall Through:

.. image:: images/09_media/image19.png
      
The simulation results are as follows. It can be seen that when rd_en is active, the data is also valid simultaneously, without a one-cycle delay.

.. image:: images/09_media/image20.png
      
On-Board Verification
---------------------

After generating the bit file and downloading it, two ILA instances will appear. First, let's look at the write channel. It can be seen that when the full signal is high, wr_en goes low, and no more data is written.

.. image:: images/09_media/image21.png
      
The read channel is also consistent with the simulation results.

.. image:: images/09_media/image22.png
      
If the rising edge of rd_en is used as the trigger condition, click run, and then press the reset button (which is mapped to PL KEY1), the following result will appear, consistent with the simulation. In Standard FIFO mode, the data lags behind rd_en by one clock cycle.

.. image:: images/09_media/image23.png
      
