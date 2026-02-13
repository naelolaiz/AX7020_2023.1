HDMI Clock Display Experiment
==============================

**The Vivado project for this experiment is "hdmi_rtc_char".**

This chapter builds upon the character display experiment. By reading the DS1302 chip datasheet, we learn about the DS1302 operation timing and related registers, and then design a program to display the DS1302 RTC time via HDMI, similar to a digital clock.

Experiment Principle
---------------------

The RTC (Real-Time Clock) provides the system with a reliable time source. Even when the power is off, the RTC can continue running on battery power. The RTC transmits 8-bit data (BCD encoded) to the FPGA via an SPI-like bus. The data includes seconds, minutes, hours, date, day, month, and year. In this experiment, we will read the hours, minutes, and seconds data from the RTC and display the time on the screen.

Hardware Introduction
----------------------

The RTC design on the development board uses the low-power real-time clock chip DS1302 from DALLAS. The VCC2 of DS1302 is the main power supply, and VCC1 is the backup power supply. When the main power is off, the battery can maintain continuous clock operation. The DS1302 is connected to an external 32.768kHz crystal oscillator to provide the oscillation source for the RTC circuit. The schematic of the RTC section is shown below:

.. image:: images/19_media/image1.png
      
DS1302 Timing and Control
---------------------------

Write Data Timing
~~~~~~~~~~~~~~~~~~

The interface is similar to an SPI interface, but the difference is that its data interface is bidirectional. The timing diagram for the DS1302 chip write operation is as follows. The first byte is the "register access address", and the second byte is the "write data". During write operations, data is valid on the "rising edge", and additionally, the CE (/RST) signal must be pulled high. (Data is sent starting from the LSB, i.e., from the least significant bit to the most significant bit.)

.. image:: images/19_media/image2.png
      
DS1302 Write Timing

Read Data Timing
~~~~~~~~~~~~~~~~~

The read timing is largely similar to the write timing. The difference is that the second byte involves a "read data" action. At the beginning of reading the second byte, the SCLK signal outputs data on the falling edge, and data can be read on the rising edge. The CE (/RST) signal must also be pulled high. (The first byte of data is output starting from the LSB, and the second byte of data is read in starting from the LSB.)

.. image:: images/19_media/image3.png
      
DS1302 Read Timing

Command Format and Registers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whether it is a read or write operation, in the timing diagram, the first byte is always the "register access address", and this byte has its own format.

.. image:: images/19_media/image4.png
      
BIT 7 is fixed. BIT 6 indicates whether to access the register itself or the RAM space. BIT 5 to BIT 1 represent the address of the register or RAM space. BIT 0 indicates whether the operation on the register is a write or a read.

The figure below shows the DS1302 register addresses and data formats.

.. image:: images/19_media/image5.png
      
Program Design
---------------

.. image:: images/19_media/image6.png

DS1302 Read/Write Design
~~~~~~~~~~~~~~~~~~~~~~~~~~

By analyzing the DS1302 read/write timing, it can be seen that it is similar to SPI timing, except that data output and input are time-division multiplexed. The SPI master state machine design mainly handles the read and write of one byte of SPI data. Since it is full-duplex, one byte is read while writing one byte simultaneously. First, in the idle state "IDLE", upon receiving a write request, it enters the "DCLK_IDLE" state. This state holds the SPI clock edge transition for a certain period to control the SPI clock cycle, then enters the SPI clock edge transition state. For one byte, there are 16 data edges in total (rising and falling edges combined). At the last data edge, it enters the "LAST_HALF_CYCLE" state to hold the last edge for a certain period, then enters the acknowledge state to complete one write request.

.. image:: images/19_media/image7.png
      
SPI Master Module State Diagram

The spi_master module simulates an SPI clock, which toggles when the state machine enters the 'DCLK_EDGE' state.

.. code:: verilog

 //SPI clock edge counter
 always@(posedge sys_clk or posedge rst)
 begin
 if(rst)
 clk_edge_cnt <= 5'd0;
 else if(state == DCLK_EDGE)
 clk_edge_cnt <= clk_edge_cnt + 5'd1;
 else if(state == IDLE)
 clk_edge_cnt <= 5'd0;
 end

+-------------+-------+------------------------------------------------+
| Signal Name | Dir   | Description                                    |
+=============+=======+================================================+
| sys_clk     | in    | Clock input                                    |
+-------------+-------+------------------------------------------------+
| rst         | in    | Asynchronous reset input, active high          |
+-------------+-------+------------------------------------------------+
| nCS         | out   | SPI chip select signal, equals nCS_ctrl        |
+-------------+-------+------------------------------------------------+
| DCLK        | out   | SPI serial clock                               |
+-------------+-------+------------------------------------------------+
| MOSI        | out   | SPI serial data output                         |
+-------------+-------+------------------------------------------------+
| MISO        | in    | SPI serial data input                          |
+-------------+-------+------------------------------------------------+
| CPOL        | in    | Clock Polarity, SPI clock polarity             |
|             |       |                                                |
|             |       | 0: Idle state is 0                             |
|             |       |                                                |
|             |       | 1: Idle state is 1                             |
+-------------+-------+------------------------------------------------+
| CPHA        | in    | Clock Phase, SPI clock phase                   |
|             |       |                                                |
|             |       | 0: Sample on the first edge                   |
|             |       |                                                |
|             |       | 1: Sample on the second edge                  |
+-------------+-------+------------------------------------------------+
| nCS_ctrl    | in    | nCS control                                    |
+-------------+-------+------------------------------------------------+
| clk_div     | in    | SPI clock frequency control                    |
|             |       |                                                |
|             |       | SPI clock = sys clock / (2*(2+clk_div))        |
|             |       |                                                |
|             |       | clk_div minimum value is 0; when 0,            |
|             |       | SPI clock is 1/4 of the system clock           |
+-------------+-------+------------------------------------------------+
| wr_req      | in    | Write one byte request                         |
+-------------+-------+------------------------------------------------+
| wr_ack      | out   | Write acknowledge, active high                 |
+-------------+-------+------------------------------------------------+
| data_in     | in    | Data                                           |
+-------------+-------+------------------------------------------------+
| data_out    | out   | Returned data, valid on write acknowledge      |
+-------------+-------+------------------------------------------------+

SPI Master Port Description

The ds1302_io module handles DS1302 register read/write control. The state machine is shown in the figure below.

In the "S_IDLE" idle state, upon receiving a register read/write request, it enters the "S_CE_HIGH" state to pull CE high, then enters either the read (S_READ) or write (S_WRITE) state based on the request type.

In the "S_WRITE" state, the next state transitions to the write address state "S_WRITE_ADDR", then to the write data state "S_WRITE_DATA" to complete writing one register, and finally acknowledges and pulls CE low.

In the "S_READ" state, the next state transitions to the read address state "S_READ_ADDR", then to the read data state "S_READ_DATA" to complete reading one register, and finally acknowledges and pulls CE low.

.. image:: images/19_media/image8.png
      
ds1302_io State Machine

+---------------+--------+---------------------------------------------+
| Signal Name   | Dir    | Description                                 |
+===============+========+=============================================+
| clk           | in     | Clock input                                 |
+---------------+--------+---------------------------------------------+
| rst           | in     | Asynchronous reset input, active high       |
+---------------+--------+---------------------------------------------+
| ds1302_ce     | out    | DS1302 CE, active high                      |
+---------------+--------+---------------------------------------------+
| ds1302_sclk   | out    | DS1302 serial clock                         |
+---------------+--------+---------------------------------------------+
| ds1302_io     | inout  | DS1302 data                                 |
+---------------+--------+---------------------------------------------+
| cmd_read      | in     | Read register request, address must be      |
|               |        | ready when the request is issued            |
+---------------+--------+---------------------------------------------+
| cmd_write     | in     | Write register request, address and data    |
|               |        | must be ready when the request is issued    |
+---------------+--------+---------------------------------------------+
| cmd_read_ack  | out    | Read register acknowledge, read data is     |
|               |        | valid upon acknowledge                      |
+---------------+--------+---------------------------------------------+
| cmd_write_ack | out    | Write register acknowledge                  |
+---------------+--------+---------------------------------------------+
| read_addr     | in     | Read register address                       |
+---------------+--------+---------------------------------------------+
| write_addr    | in     | Write register address                      |
+---------------+--------+---------------------------------------------+
| read_data     | out    | Data read out                               |
+---------------+--------+---------------------------------------------+
| write_data    | in     | Write register data                         |
+---------------+--------+---------------------------------------------+

ds1302_io Ports

The ds1302 module mainly handles the read/write control of time registers. The state machine is relatively simple.

.. image:: images/19_media/image9.png
      
ds1302 Module State Machine

+-------+---+---------------------------------------------------------+
| Signal| D | Description                                             |
| Name  | i |                                                         |
|       | r |                                                         |
+=======+===+=========================================================+
| clk   | i | Clock input                                             |
|       | n |                                                         |
+-------+---+---------------------------------------------------------+
| rst   | i | Asynchronous reset input, active high                   |
|       | n |                                                         |
+-------+---+---------------------------------------------------------+
| ds13  | o | DS1302 CE, active high                                  |
| 02_ce | u |                                                         |
|       | t |                                                         |
+-------+---+---------------------------------------------------------+
| d     | o | DS1302 serial clock                                     |
| s1302 | u |                                                         |
| _sclk | t |                                                         |
+-------+---+---------------------------------------------------------+
| ds13  | i | DS1302 data                                             |
| 02_io | n |                                                         |
|       | o |                                                         |
|       | u |                                                         |
|       | t |                                                         |
+-------+---+---------------------------------------------------------+
| writ  | i | ds1302 write time request; when the request is issued,  |
| e_tim | n | time data write_second, write_minute, write_hour,       |
| e_req |   | write_date, write_month, write_week, write_year must    |
|       |   | be valid                                                |
+-------+---+---------------------------------------------------------+
| writ  | o | Write time request acknowledge                          |
| e_tim | u |                                                         |
| e_ack | t |                                                         |
+-------+---+---------------------------------------------------------+
| wr    | i | Write time: seconds, BCD encoded, 00-59                 |
| ite_s | n |                                                         |
| econd |   |                                                         |
+-------+---+---------------------------------------------------------+
| wr    | i | Write time: minutes, BCD encoded, 00-59                 |
| ite_m | n |                                                         |
| inute |   |                                                         |
+-------+---+---------------------------------------------------------+
| write | i | Write time: hours, BCD encoded, 00-23                   |
| _hour | n |                                                         |
+-------+---+---------------------------------------------------------+
| write | i | Write time: date, BCD encoded, 01-31                    |
| _date | n |                                                         |
+-------+---+---------------------------------------------------------+
| write | i | Write time: month, BCD encoded, 01-12                   |
| _     | n |                                                         |
| month |   |                                                         |
+-------+---+---------------------------------------------------------+
| write | i | Write time: day of week, BCD encoded, 01-07             |
| _week | n |                                                         |
+-------+---+---------------------------------------------------------+
| write | i | Write time: year, BCD encoded, 00-99                    |
| _year | n |                                                         |
+-------+---+---------------------------------------------------------+
| rea   | i | Read time request                                       |
| d_tim | n |                                                         |
| e_req |   |                                                         |
+-------+---+---------------------------------------------------------+
| rea   | o | Read time request acknowledge                           |
| d_tim | u |                                                         |
| e_ack | t |                                                         |
+-------+---+---------------------------------------------------------+
| r     | o | Read time: seconds, BCD encoded, 00-59                  |
| ead_s | u |                                                         |
| econd | t |                                                         |
+-------+---+---------------------------------------------------------+
| r     | o | Read time: minutes, BCD encoded, 00-59                  |
| ead_m | u |                                                         |
| inute | t |                                                         |
+-------+---+---------------------------------------------------------+
| read  | o | Read time: hours, BCD encoded, 00-23                    |
| _hour | u |                                                         |
|       | t |                                                         |
+-------+---+---------------------------------------------------------+
| read  | o | Read time: date, BCD encoded, 01-31                     |
| _date | u |                                                         |
|       | t |                                                         |
+-------+---+---------------------------------------------------------+
| read_m| o | Read time: month, BCD encoded, 01-12                    |
| onth  | u |                                                         |
|       | t |                                                         |
+-------+---+---------------------------------------------------------+
| read  | o | Read time: day of week, BCD encoded, 01-07              |
| _week | u |                                                         |
|       | t |                                                         |
+-------+---+---------------------------------------------------------+
| read  | o | Read time: year, BCD encoded, 00-99                     |
| _year | u |                                                         |
|       | t |                                                         |
+-------+---+---------------------------------------------------------+

ds1302 Module Ports

The ds1302_test module mainly performs CH status detection. CH is located at BIT 7 of the seconds register. After power-up, it first reads the time, checks the CH status of the seconds register. If it is high, it means the DS1302 is halted, and the state machine enters "S_WRITE_CH" to write 0 to CH and write an initial time. Then it continuously reads the time registers in a loop.

.. image:: images/19_media/image10.png
      
ds1302_test State Machine

+---------------+--------+--------------------------------------------+
| Signal Name   | Dir    | Description                                |
+===============+========+============================================+
| clk           | in     | Clock input                                |
+---------------+--------+--------------------------------------------+
| rst           | in     | Asynchronous reset input, active high      |
+---------------+--------+--------------------------------------------+
| ds1302_ce     | out    | DS1302 CE, active high                     |
+---------------+--------+--------------------------------------------+
| ds1302_sclk   | out    | DS1302 serial clock                        |
+---------------+--------+--------------------------------------------+
| ds1302_io     | inout  | DS1302 data                                |
+---------------+--------+--------------------------------------------+
| read_second   | out    | Time: seconds, BCD encoded, 00-59          |
+---------------+--------+--------------------------------------------+
| read_minute   | out    | Time: minutes, BCD encoded, 00-59          |
+---------------+--------+--------------------------------------------+
| read_hour     | out    | Time: hours, BCD encoded, 00-23            |
+---------------+--------+--------------------------------------------+
| read_date     | out    | Time: date, BCD encoded, 01-31             |
+---------------+--------+--------------------------------------------+
| read_month    | out    | Time: month, BCD encoded, 01-12            |
+---------------+--------+--------------------------------------------+
| read_week     | out    | Time: day of week, BCD encoded, 01-07      |
+---------------+--------+--------------------------------------------+
| read_year     | out    | Time: year, BCD encoded, 00-99             |
+---------------+--------+--------------------------------------------+

ds1302_test Ports

Character Overlay Design
~~~~~~~~~~~~~~~~~~~~~~~~~

Referring to the previous character overlay experiment, since the characters in the previous experiment were static, and this section requires dynamically displaying the RTC data, the content of a character display area needs to be variable. We need to create a character library containing digits 0-9 and the separator ":". Considering the large number of characters, placing them in a single ROM would make it difficult to access. Therefore, instead of instantiating a ROM, we use case statements to create the character library in char_repo.v. For example, the figure below shows the character library expression for the digit 0.

.. image:: images/19_media/image11.png
      
The character library data is generated by the "FPGA Font Extraction" software, with a dot matrix width x height of 16x32, which is 64 bytes.

.. image:: images/19_media/image12.png
      
In the program, char_addr_sel is used to select which character to use. 0-9 correspond to digits 0-9, and 10 corresponds to ":".

.. image:: images/19_media/image13.png
      
+----------------+--------+--------------------------------------------+
| Signal Name    | Dir    | Description                                |
+================+========+============================================+
| clk            | in     | Clock input                                |
+----------------+--------+--------------------------------------------+
| char_addr_sel  | in     | Select character, 0-9 for digits 0-9,     |
|                |        | 10 for ":"                                 |
+----------------+--------+--------------------------------------------+
| char_addr      | in     | Character data address                     |
+----------------+--------+--------------------------------------------+
| char_data      | out    | Character data                             |
+----------------+--------+--------------------------------------------+

char_repo Module Interface Signals

The rtc_osd.v module is used to overlay the RTC data onto the color bar, and sets the following parameters. Since one character width is 16 pixels, the spacing between two characters is set to 16.

.. image:: images/19_media/image14.png
      
Since the hours, minutes, seconds, and separators total 8 characters, eight valid display areas are generated.

.. image:: images/19_media/image15.png
      
The character selection signal is decoded based on the RTC data values.

.. image:: images/19_media/image16.png
      
+-------------------+-------+------------------------------------------+
| Signal Name       | Dir   | Description                              |
+===================+=======+==========================================+
| rst_n             | in    | Asynchronous reset input, active low     |
+-------------------+-------+------------------------------------------+
| pclk              | in    | External clock input                     |
+-------------------+-------+------------------------------------------+
| rtc_data          | In    | RTC data, 24-bit, hours/minutes/seconds  |
+-------------------+-------+------------------------------------------+
| i_hs              | in    | Horizontal sync signal                   |
+-------------------+-------+------------------------------------------+
| i_vs              | in    | Vertical sync signal                     |
+-------------------+-------+------------------------------------------+
| i_de              | in    | Data enable signal                       |
+-------------------+-------+------------------------------------------+
| i_data            | in    | color_bar data                           |
+-------------------+-------+------------------------------------------+
| o_hs              | out   | Output horizontal sync signal            |
+-------------------+-------+------------------------------------------+
| o_vs              | out   | Output vertical sync signal              |
+-------------------+-------+------------------------------------------+
| o_de              | out   | Output data enable signal                |
+-------------------+-------+------------------------------------------+
| o_data            | out   | Output data                              |
+-------------------+-------+------------------------------------------+

rtc_osd Module Signals

Experiment Result
------------------

After connecting the download cable and HDMI cable and downloading the program to the board, you can see that the HDMI display background shows color bars, and the time is displayed in the upper left corner, updating every second.

.. image:: images/19_media/image17.png
      
AX7020/AX7010 Hardware Connection Diagram

The button cell battery model is CR1220. When installing, make sure the positive side faces up. To remove it, use tweezers to push the yellow spring clip and the battery will pop out.

.. image:: images/19_media/image18.png
      
.. image:: images/19_media/image19.png
      
