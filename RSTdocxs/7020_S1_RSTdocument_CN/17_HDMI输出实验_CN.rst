HDMI Output Experiment
======================

**The Vivado project for this experiment is "hdmi_output_test".**

In the previous section, we introduced the LED blinking experiment, which was only meant to familiarize you with the basic Vivado development workflow. This chapter's experiment is more complex than the LED blinking experiment — we will generate color bars for HDMI output, which also serves as a foundation for later learning about display and video processing. This experiment does not involve the PS system. As the experiment design shows, a solid foundation in FPGA knowledge is required to make the best use of the ZYNQ chip.

Hardware Introduction
---------------------

The development board does not use an HDMI encoder chip. Instead, the FPGA's 3.3V differential IOs are directly connected to the HDMI connector,
and the FPGA encodes 24-bit RGB data and outputs TMDS differential signals.

.. image:: images/17_media/image1.png
      
TMDS Principles
~~~~~~~~~~~~~~~

HDMI uses the same transmission principle as DVI — TMDS (Transition Minimized Differential Signal), which minimizes transmission transitions using differential signaling.

The TMDS transmission system consists of two parts: the transmitter and the receiver. The TMDS transmitter receives 24-bit parallel data representing RGB signals from the HDMI interface (TMDS encodes the RGB primary colors of each pixel at 8 bits each, i.e., 8 bits for R, 8 bits for G, and 8 bits for B), then encodes and performs parallel-to-serial conversion on this data, and distributes the data representing the 3 RGB signals to independent transmission channels for output. The receiver accepts serial signals from the transmitter, decodes them and performs serial-to-parallel conversion, then sends the data to the display controller. At the same time, it also receives the clock signal for synchronization.

**TMDS Principles**

Each TMDS link includes 3 data channels for transmitting RGB signals and 1 channel for transmitting the clock signal. Each data channel uses an encoding algorithm to convert 8-bit video and audio data into transition-minimized, DC-balanced 10-bit data. This makes data transmission and recovery more reliable. The transition-minimized differential signal uses XOR and XNOR logic algorithms to convert the original 8-bit signal data into 10 bits. The first 8 bits of data are derived from the original signal through computation, the 9th bit indicates the type of operation performed, and the 10th bit is used for DC balancing.

Generally, the encoding format for HDMI transmission includes video data, control data, and data packets (which contain audio data and auxiliary information such as error correction codes). Each TMDS channel transmits 2-bit control data, 8-bit video data, or 4-bit data packets. During HDMI data transmission, the process can be divided into three phases: the video data transmission period, the control data transmission period, and the data island transmission period, corresponding to the three data types mentioned above.

The following describes the technologies used in TMDS:

1. Transition Minimization

 After encoding and DC balancing, 8-bit data becomes 10-bit minimized data. This appears to add redundant bits and demand higher bandwidth from the transmission link. However, in practice, the 10-bit data obtained through this algorithm is more reliably transmitted over longer coaxial cables. The figure below shows an example of encoding and parallel-to-serial conversion of 8-bit parallel RED data.

.. image:: images/17_media/image2.jpeg
   
      
Step 1: Send the 8-bit parallel RED data to the TMDS transmitter.

Step 2: Parallel-to-serial conversion.

Step 3: Perform transition minimization processing by adding the 9th bit, which is the encoding process. The 9th bit is called the encoding bit.

2. DC Balancing

DC balancing (DC-balanced) means ensuring zero DC offset in the channel during the encoding process. The method is to add a 10th bit after the original 9-bit data. This way, the transmitted data tends toward DC balance, reducing electromagnetic interference on the transmission line and improving transmission reliability.

3. Differential Signaling

TMDS differential transmission technology uses the voltage difference between 2 pins to transmit signals. The data value ("0" or "1") is determined by the polarity and magnitude of the voltage between the two pins. That is, 2 wires are used to transmit the signal — one wire carries the original signal, and the other carries the inverted signal. This way, the receiver can subtract the signal on one wire from the signal on the other to reject electromagnetic interference and obtain the correct signal.

As shown in the figure below:

.. image:: images/17_media/image3.jpeg
   
      
 Additionally, there is a Display Data Channel (DDC), which is a signal line used to read the Extended Display Identification Data (EDID) that describes the receiver display's capabilities such as resolution. Devices equipped with HDCP (High-bandwidth Digital Content
Protection) also use the DDC line for authentication key exchange between transmitting and receiving devices.

Video Timing Standards
~~~~~~~~~~~~~~~~~~~~~~

An HDMI display scans starting from the top-left corner of the screen, scanning point by point from left to right. After each line is scanned, the electron beam returns to the beginning of the next line on the left side of the screen. During this time, the CRT blanks the electron beam. At the end of each line, a horizontal sync signal is used for synchronization. When all lines have been scanned to form a frame, a vertical sync signal is used for vertical synchronization, and the scan returns to the top-left of the screen while vertical blanking occurs, beginning the next frame.

The time to complete one line scan is called the horizontal scan time, and its reciprocal is called the line frequency. The time to complete one frame (full screen) scan is called the vertical scan time, and its reciprocal is called the field frequency, i.e., the screen refresh rate. Common values include 60Hz, 75Hz, etc. The standard display field frequency is 60Hz.

Clock frequency: Taking 1024x768@59.94Hz (60Hz) as an example, each frame corresponds to 806 line periods, of which 768 are display lines. Each display line includes 1344 clock cycles, of which 1024 are the active display area. Therefore, the required pixel clock frequency is: 806 × 1344 × 60 ≈ 65MHz.

.. image:: images/17_media/image4.png
      
Video Timing

VGA scanning uses line scanning as the basic element, with multiple lines forming a frame. The figure below shows the timing of one line, where "Active" Video represents the valid pixels of a line. In most resolution standards, Top/Left Border and Bottom/Right Border are 0. "Blanking" is the synchronization time of a line. The "Blanking" time plus the "Active" Video time equals the total time for one line. "Blanking" is further divided into three segments: "Front Porch", "Sync", and "Back Porch".

.. image:: images/17_media/image5.png
      
Horizontal Sync Timing

The following are the timing parameters for 720p

|image1|\ 1280x720@60Hz Timing Parameters

Building the Vivado Project
---------------------------

This experiment will implement HDMI output display. Verilog is used to program and drive the HDMI output, displaying test color bar images on an HDMI monitor. The HDMI output display module is divided into 3 sub-modules: the clock module vidio_pll, the color bar generation module color_bar, and the VGA-to-DVI module rgb2dvi. The logic block diagram is as follows:

.. image:: images/17_media/image7.png

Adding the HDMI Encoder IP Core
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1) Create a new project named "hdmi_output_test"

Many people are familiar with VGA data, which is RGB data, while HDMI uses TMDS differential signals. RGB data is easy to work with in FPGA, so what we need to do is convert RGB data to HDMI TMDS differential signals. Therefore, we use the RGB to DVI IP (both DVI and HDMI use TMDS signals).

2) Copy the repo folder (this folder can be found in the provided example project) to the project directory. This folder contains the HDMI encoder IP provided by a third-party vendor.

.. image:: images/17_media/image8.png
      
3) Click "IP Catalog". By default, these IPs are all provided by Xilinx. Now we need to add third-party IPs or our own custom IPs.

.. image:: images/17_media/image9.png
      
4) Right-click and select "Add Repository..."

.. image:: images/17_media/image10.png
      
5) Select the path to the repo folder that was copied earlier.

.. image:: images/17_media/image11.png
      
6) A success message will indicate how many IPs were added.

.. image:: images/17_media/image12.png
      
7) Find "RGB to DVI Video Encoder(Source)" and double-click it.

.. image:: images/17_media/image13.png
      
8) In the pop-up window, keep the "Component Name" unchanged and leave other parameters as default, then click "OK".

.. image:: images/17_media/image14.png
      
9) A "Generate Output Products" window will appear, where "Number of jobs" refers to the number of threads — higher is faster.

.. image:: images/17_media/image15.png
      
10) You can now see an IP named rgb2dvi_0.

.. image:: images/17_media/image16.png
      
Adding the Pixel Clock PLL Module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To drive the HDMI encoder, a pixel clock and a 5x pixel clock are needed. The 5x pixel clock is used for 10:1 serialization.

1) In the "IP Catalog" window, search for the keyword "clock" and double-click "Clocking Wizard".

.. image:: images/17_media/image17.png
      
2) This time, give the component a name. Enter "video_clock" in the "Component Name" field, and set "clk_in1" to 50. The 50MHz here matches the crystal oscillator frequency on the PL side of the development board.

.. image:: images/17_media/image18.png
      
3) The output clock "clk_out1" is used as the video pixel clock. Enter 74.25 here, which is the pixel clock for the 1280x720@60 resolution. The pixel clock differs for each resolution, and a thorough understanding of video standards is needed to know the pixel clock for each video resolution. "clk_out2" is used for encoder serialization at 5 times the pixel clock. Enter 371.25 here, then click "OK" to generate the IP.

.. image:: images/17_media/image19.png
      
Adding the Color Bar Generator Module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

4) The color bar generator module is a piece of Verilog code used to generate video timing and 8 horizontal color bars. The existing code can be copied from the provided example project. The color_bar file defines parameters for different resolutions for user reference.

.. image:: images/17_media/image20.png
      
Add the video_define file, which contains the macro definitions for 1280x720.

|image2|\ |image3|

Adding the Top-Level Module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

5) The top module instantiates the color bar generator module, the HDMI encoder module, and the pixel clock generation module. Refer to the code in the provided example project.

.. image:: images/17_media/image23.png
      
Adding the XDC Constraint File
-------------------------------

Add the following XDC constraint file to the project. The constraint file includes pin assignments for the clock and HDMI-related signals.

.. image:: images/17_media/image24.png
      
::

 set_property PACKAGE_PIN U18 [get_ports {sys_clk}]
 set_property IOSTANDARD LVCMOS33 [get_ports {sys_clk}]
 create_clock -period 20.000 -waveform {0.000 10.000} [get_ports sys_clk]
 set_property IOSTANDARD TMDS_33 [get_ports TMDS_clk_n]
 set_property PACKAGE_PIN N18 [get_ports TMDS_clk_p]
 set_property IOSTANDARD TMDS_33 [get_ports TMDS_clk_p]
 set_property IOSTANDARD TMDS_33 [get_ports {TMDS_data_n[0]}]
 set_property PACKAGE_PIN V20 [get_ports {TMDS_data_p[0]}]
 set_property IOSTANDARD TMDS_33 [get_ports {TMDS_data_p[0]}]
 set_property IOSTANDARD TMDS_33 [get_ports {TMDS_data_n[1]}]
 set_property PACKAGE_PIN T20 [get_ports {TMDS_data_p[1]}]
 set_property IOSTANDARD TMDS_33 [get_ports {TMDS_data_p[1]}]
 set_property IOSTANDARD TMDS_33 [get_ports {TMDS_data_n[2]}]
 set_property PACKAGE_PIN N20 [get_ports {TMDS_data_p[2]}]
 set_property IOSTANDARD TMDS_33 [get_ports {TMDS_data_p[2]}]
 set_property PACKAGE_PIN V16 [get_ports hdmi_oen]
 set_property IOSTANDARD LVCMOS33 [get_ports hdmi_oen]

Download and Debug
------------------

Save the project and compile to generate the bit file. Connect the HDMI interface to an HDMI monitor. Note that this experiment uses 1280x720@60Hz, so please ensure your monitor supports this resolution.

.. image:: images/17_media/image25.png
      
.. image:: images/17_media/image26.png
      
AX7020/AX7010 Hardware Connection Diagram

The monitor will display the following image after downloading

.. image:: images/17_media/image27.png
      
Experiment Summary
------------------

This experiment provides an initial introduction to video display and involves video-related knowledge. This is not the focus of ZYNQ learning, but ZYNQ is widely used in the video processing field, and learners need a solid foundation of knowledge. In this experiment, only the PL is used to drive the HDMI output. We have learned the basics of using third-party custom IPs, and in later chapters, we will learn how to create custom IPs.

.. |image1| image:: images/17_media/image6.png
.. |image2| image:: images/17_media/image21.png
.. |image3| image:: images/17_media/image22.png
