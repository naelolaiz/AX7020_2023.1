HDMI Character Display Experiment
===================================

**The Vivado project for this experiment is "hdmi_char".**

The HDMI output experiment explained the HDMI display principles and display methods. This experiment introduces how to use FPGA to implement character display, providing a deeper understanding of HDMI display methods through this experiment.

Experiment Principle
---------------------

This experiment uses a character conversion tool to convert characters into hexadecimal coe files, which are stored in a single-port ROM IP core. The converted data is then read from the ROM and displayed on the HDMI output.

Program Design
---------------

The character display routine adds an osd_display module on top of the HDMI display framework. The "osd_display" module is used to read the converted character information stored in the ROM IP core and display it in a specified area. The program block diagram is shown below:

.. image:: images/18_media/image1.png

1. The "timing_gen_xy" module defines two counters "x_cnt" and "y_cnt" based on the HDMI timing standard, and these two counters generate the "x" and "y" coordinates for the HDMI display. In the program, "vs_edge" and "de_falling" represent the vertical sync start signal and data valid end signal, respectively. The principle is shown in the figure below:

.. image:: images/18_media/image2.png
      
+--------------------+-------+----------------------------------------+
| Signal Name        | Dir   | Description                            |
+====================+=======+========================================+
| rst_n              | in    | Asynchronous reset input, active low   |
+--------------------+-------+----------------------------------------+
| clk                | in    | External clock input                   |
+--------------------+-------+----------------------------------------+
| i_hs               | in    | Horizontal sync signal                 |
+--------------------+-------+----------------------------------------+
| i_vs               | in    | Vertical sync signal                   |
+--------------------+-------+----------------------------------------+
| i_de               | in    | Data valid signal                      |
+--------------------+-------+----------------------------------------+
| i_data             | in    | color_bar data                         |
+--------------------+-------+----------------------------------------+
| o_hs               | out   | Output horizontal sync signal          |
+--------------------+-------+----------------------------------------+
| o_vs               | out   | Output vertical sync signal            |
+--------------------+-------+----------------------------------------+
| o_de               | out   | Output data valid signal               |
+--------------------+-------+----------------------------------------+
| o_data             | out   | Output data                            |
+--------------------+-------+----------------------------------------+
| x                  | out   | Generated X coordinate                 |
+--------------------+-------+----------------------------------------+
| y                  | out   | Generated Y coordinate                 |
+--------------------+-------+----------------------------------------+

timing_gen_xy module ports

1. The following introduces the ROM IP for storing character information. First, you need to generate a .coe file that can be recognized by XILINX FPGA.

First, locate the "FPGA Font Extraction" tool in the project folder.

.. image:: images/18_media/image3.png
      
Double-click the .exe file to open the tool.

   .. image:: images/18_media/image4.png
            
In the "Character Input" box of the extraction tool, enter the characters to be displayed. The font and character height can be customized. After setting up, click the "Convert" button. In the lower-left corner of the interface, you can see the converted character dot matrix size. The width and height of the dot matrix are needed in the program.

   .. image:: images/18_media/image5.png
            
The width and height of the dot matrix here are 144x32, which need to match the definitions in the osd_display program:

   .. image:: images/18_media/image6.png
            
Click the "Save" button to save the file to the source file directory of this routine. Note that in the save type dropdown, you should select Xilinx (\*.coe), then click the "Save" button.

.. image:: images/18_media/image7.png
      
Return to the character extraction tool interface. The following dialog box indicates that saving is complete. Click OK to exit.

.. image:: images/18_media/image8.png
      
Locate and open the generated .coe file, and you can see the following:

.. image:: images/18_media/image9.png
      
The process of instantiating the single-port ROM IP core has been introduced in the previous ROM usage section. Set it to Single Port
ROM

.. image:: images/18_media/image10.png
      
In the PortA Options tab, configure as follows:

.. image:: images/18_media/image11.png
      
Add the osd.coe file as shown below (locate the previously generated coe file), then click the "OK" button when done:

.. image:: images/18_media/image12.png
      
4. The osd_display module contains the timing_gen_xy module and the osd_rom module. The osd_rom stores character data. If the data is 1, the OSD area displays the foreground color red from the ROM (displaying the ALINX logo). If the data is 0, the OSD area displays the background color (color bars).

.. image:: images/18_media/image13.png
      
Set the area valid signal, which defines the area where characters are displayed. The starting coordinates are set to (9, 9), and the area size can be configured based on the area settings of the character generation tool.

.. image:: images/18_media/image14.png
      
Many people may not understand the ROM read address part — why it is [15:3], meaning one data word is read out every eight clock cycles. This is because one dot of a character represents only 1 bit, while the ROM storage data width is 8 bits. Therefore, eight cycles are needed to extract one data word, and each bit value is compared to convert one character dot into one pixel on the image.

.. image:: images/18_media/image15.png
      
+--------------------+-------+----------------------------------------+
| Signal Name        | Dir   | Description                            |
+====================+=======+========================================+
| rst_n              | in    | Asynchronous reset input, active low   |
+--------------------+-------+----------------------------------------+
| pclk               | in    | External clock input                   |
+--------------------+-------+----------------------------------------+
| i_hs               | in    | Horizontal sync signal                 |
+--------------------+-------+----------------------------------------+
| i_vs               | in    | Vertical sync signal                   |
+--------------------+-------+----------------------------------------+
| i_de               | in    | Data valid signal                      |
+--------------------+-------+----------------------------------------+
| i_data             | in    | color_bar data                         |
+--------------------+-------+----------------------------------------+
| o_hs               | out   | Output horizontal sync signal          |
+--------------------+-------+----------------------------------------+
| o_vs               | out   | Output vertical sync signal            |
+--------------------+-------+----------------------------------------+
| o_de               | out   | Output data valid signal               |
+--------------------+-------+----------------------------------------+
| o_data             | out   | Output data                            |
+--------------------+-------+----------------------------------------+

osd_display module ports

Experiment Results
-------------------

Connect the development board and the monitor. Refer to the "HDMI Output Experiment" tutorial for the connection method. Please note that connectors on the development board should not be hot-plugged while powered on. After downloading the experiment program, you can see characters displayed on the monitor with a color bar background. The development board serves as an HDMI output device and can only display through an HDMI display device. Do not attempt to display through a laptop's HDMI port, as laptops are also output devices.

.. image:: images/18_media/image16.png
      
AX7020/AX7010 Hardware Connection Diagram

.. image:: images/18_media/image17.png
      
The default character display position is at coordinates (9, 9). Users can also modify the pos_y and pos_x conditions below to display characters at any position on the screen:

.. image:: images/18_media/image18.png
      
