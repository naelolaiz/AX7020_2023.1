ADDA Test Experiment
====================

**The Vivado project for this experiment is "an108_adda_hdmi_test".**

This experiment practices using ADC and DAC. The ADDA module used in this experiment is model AN108, with a maximum ADC sampling rate of 32MHz and 8-bit precision, and a maximum DAC sampling rate of 125MHz with 8-bit precision. In this experiment, the DAC outputs a sine wave, which is then sampled by the ADC and displayed on an HDMI monitor.

.. image:: images/23_media/image1.png
      
ADDA Module

.. image:: images/23_media/image2.png
      
Expected Experiment Result

Hardware Introduction
---------------------

.. image:: images/23_media/image3.png
      
Digital-to-Analog Conversion (DA) Circuit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As shown in the hardware block diagram, the DA circuit consists of a high-speed DA chip, a 7th-order Butterworth low-pass filter, an amplitude adjustment circuit, and a signal output interface.

The high-speed DA chip we use is the AD9708 from Analog Devices. The AD9708 is an 8-bit, 125MSPS DA conversion chip with a built-in 1.2V reference voltage and differential current output. The internal block diagram of the chip is shown below.

.. image:: images/23_media/image4.png
      
After the differential output of the AD9708 chip, a 7th-order Butterworth low-pass filter with a bandwidth of 40MHz is connected to prevent noise interference. The frequency response is shown below.

.. image:: images/23_media/image5.png
      
The filter parameters are shown below.

.. image:: images/23_media/image6.png
      
After the filter, we use two high-performance AD8056 op-amps with 145MHz bandwidth to convert from differential to single-ended and provide amplitude adjustment, maximizing the overall circuit performance. The amplitude adjustment uses a 5K potentiometer, and the final output range is -5V to 5V (10Vpp).

Note:\ **Due to the limited precision of the circuit components, the final output may have some error. The waveform amplitude may not reach 10Vpp, or waveform clipping may occur. These are all normal situations.**\ 

Analog-to-Digital Conversion (AD) Circuit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As shown in the hardware block diagram, the AD circuit consists of a high-speed AD chip, an attenuation circuit, and a signal input interface.

The high-speed AD chip we use is the AD9280 from Analog Devices, an 8-bit chip with a maximum sampling rate of 32MSPS. The internal block diagram is shown below.

.. image:: images/23_media/image7.png
      
Based on the configuration shown below, we set the AD voltage input range to 0V to 2V.

.. image:: images/23_media/image8.png
      
Before the signal enters the AD chip, we built an attenuation circuit using an AD8056 chip, with an interface input range of -5V to +5V (10Vpp). After attenuation, the input range meets the AD chip's input range (0 to 2V). The conversion formula is as follows:

.. image:: images/23_media/image9.png
      
When the input signal Vin = 5(V), the signal input to the AD is Vad = 2(V);

When the input signal Vin = -5(V), the signal input to the AD is Vad = 0(V);

Program Design
--------------

The program design for this experiment is basically similar to the AN706 waveform display experiment, except that the ADDA module has a single-channel AD, so only one channel of waveform acquisition is overlaid. Additionally, the FPGA generates sine wave data through a ROM IP and outputs it to the DA chip for DA conversion, producing a sine wave analog signal. Users only need to connect the AD and DA ports of the module with a BNC cable to form a loop. This way, the HDMI monitor will display the DA sine wave signal.

.. image:: images/23_media/image10.png

The ad9280_sample module mainly handles the 8-bit AD data acquisition and conversion of the ad9280. It collects 1280 data points each time, then waits for a period before collecting the next 1280 data points.

+--------------+------+-------+--------------------------------------+
| Signal Name  | Dir  | Width | Description                          |
|              |      | (bit) |                                      |
+==============+======+=======+======================================+
| adc_clk      | in   | 1     | ADC system clock                     |
+--------------+------+-------+--------------------------------------+
| rst          | in   | 1     | Async reset, active high             |
+--------------+------+-------+--------------------------------------+
| adc_data     | in   | 8     | ADC data input                       |
+--------------+------+-------+--------------------------------------+
| adc_buf_wr   | out  | 1     | ADC data write enable                |
+--------------+------+-------+--------------------------------------+
| adc_buf_addr | out  | 12    | ADC data write address               |
+--------------+------+-------+--------------------------------------+
| adc_buf_data | out  | 8     | Unsigned 8-bit ADC data              |
+--------------+------+-------+--------------------------------------+

ad9280_sample Module Ports

The grid_display module mainly handles the overlay of grid lines on the video image. In this experiment, a color bar video is input, then a grid is overlaid and output. This grid area is provided for the subsequent waveform display module. The grid area is located at the display position from 9 to 1018 in the horizontal direction (left to right) and from 9 to 308 in the vertical direction (top to bottom).

.. image:: images/23_media/image11.png
      
+-------------+------+-------+----------------------------------------+
| Signal Name | Dir  | Width | Description                            |
|             |      | (bit) |                                        |
+=============+======+=======+========================================+
| pclk        | in   | 1     | Pixel clock                            |
+-------------+------+-------+----------------------------------------+
| rst_n       | in   | 1     | Async reset, active low                |
+-------------+------+-------+----------------------------------------+
| i_hs        | in   | 1     | Video horizontal sync input            |
+-------------+------+-------+----------------------------------------+
| i_vs        | in   | 1     | Video vertical sync input              |
+-------------+------+-------+----------------------------------------+
| i_de        | in   | 1     | Video data valid input                 |
+-------------+------+-------+----------------------------------------+
| i_data      | in   | 24    | Video data input                       |
+-------------+------+-------+----------------------------------------+
| o_hs        | out  | 1     | Video horizontal sync output with grid |
+-------------+------+-------+----------------------------------------+
| o_vs        | out  | 1     | Video vertical sync output with grid   |
+-------------+------+-------+----------------------------------------+
| o_de        | out  | 1     | Video data valid output with grid      |
+-------------+------+-------+----------------------------------------+
| o_data      | out  | 24    | Video data output with grid            |
+-------------+------+-------+----------------------------------------+

grid_display Module Ports

The wav_display module mainly handles the overlay display of waveform data. The module contains a dual-port RAM, where the write port is written by the ADC acquisition module and the read port is used by the display module. When the grid display area is active, each display line reads the AD data value stored in RAM and compares it with the Y coordinate to determine whether to display the waveform or not.

.. image:: images/23_media/image12.png
      
+--------------+------+-------+---------------------------------------+
| Signal Name  | Dir  | Width | Description                           |
|              |      | (bit) |                                       |
+==============+======+=======+=======================================+
| pclk         | in   | 1     | Pixel clock                           |
+--------------+------+-------+---------------------------------------+
| rst_n        | in   | 1     | Async reset, active low               |
+--------------+------+-------+---------------------------------------+
| wave_color   | in   | 24    | Waveform color, RGB                   |
+--------------+------+-------+---------------------------------------+
| adc_clk      | in   | 1     | ADC module clock                      |
+--------------+------+-------+---------------------------------------+
| adc_buf_wr   | in   | 1     | ADC data write enable                 |
+--------------+------+-------+---------------------------------------+
| adc_buf_addr | in   | 12    | ADC data write address                |
+--------------+------+-------+---------------------------------------+
| adc_buf_data | in   | 8     | ADC data, unsigned                    |
+--------------+------+-------+---------------------------------------+
| i_hs         | in   | 1     | Video horizontal sync input           |
+--------------+------+-------+---------------------------------------+
| i_vs         | in   | 1     | Video vertical sync input             |
+--------------+------+-------+---------------------------------------+
| i_de         | in   | 1     | Video data valid input                |
+--------------+------+-------+---------------------------------------+
| i_data       | in   | 24    | Video data input                      |
+--------------+------+-------+---------------------------------------+
| o_hs         | out  | 1     | Video horizontal sync output with grid|
+--------------+------+-------+---------------------------------------+
| o_vs         | out  | 1     | Video vertical sync output with grid  |
+--------------+------+-------+---------------------------------------+
| o_de         | out  | 1     | Video data valid output with grid     |
+--------------+------+-------+---------------------------------------+
| o_data       | out  | 24    | Video data output with grid           |
+--------------+------+-------+---------------------------------------+

wav_display Module Ports

The timing_gen_xy module is a sub-module of other modules that generates video image coordinates. The x coordinate increases from left to right, and the y coordinate increases from top to bottom.

+-------------+------+-------+----------------------------------------+
| Signal Name | Dir  | Width | Description                            |
|             |      | (bit) |                                        |
+=============+======+=======+========================================+
| clk         | in   | 1     | System clock                           |
+-------------+------+-------+----------------------------------------+
| rst_n       | in   | 1     | Async reset, active low                |
+-------------+------+-------+----------------------------------------+
| i_hs        | in   | 1     | Video horizontal sync input            |
+-------------+------+-------+----------------------------------------+
| i_vs        | in   | 1     | Video vertical sync input              |
+-------------+------+-------+----------------------------------------+
| i_de        | in   | 1     | Video data valid input                 |
+-------------+------+-------+----------------------------------------+
| i_data      | in   | 24    | Video data input                       |
+-------------+------+-------+----------------------------------------+
| o_hs        | out  | 1     | Video horizontal sync output           |
+-------------+------+-------+----------------------------------------+
| o_vs        | out  | 1     | Video vertical sync output             |
+-------------+------+-------+----------------------------------------+
| o_de        | out  | 1     | Video data valid output                |
+-------------+------+-------+----------------------------------------+
| o_data      | out  | 24    | Video data output                      |
+-------------+------+-------+----------------------------------------+
| x           | out  | 12    | X coordinate output                    |
+-------------+------+-------+----------------------------------------+
| y           | out  | 12    | Y coordinate output                    |
+-------------+------+-------+----------------------------------------+

timing_gen_xy Module Ports

Additionally, a ROM IP module is added in this example, which requires initialization data for the ROM IP. Here we only introduce how to use the waveform data generation tool. Find the tool in the software tools and drivers folder. Its icon is shown below:

.. image:: images/23_media/image13.png
      
1. Double-click the .exe file to open the tool. The interface is shown below:

.. image:: images/23_media/image14.png
      
2. You can select the waveform as needed. In this example, a sine wave is selected, and the data length and bit width are kept at default values.

.. image:: images/23_media/image15.png
      
3. Click the save button to save the generated data file to the project directory (pay attention to the file type being saved):

.. image:: images/23_media/image16.png
      
4. After saving, the following dialog box appears indicating a successful save. Click OK to close the tool.

.. image:: images/23_media/image17.png
            
Save the .coe file to the generated ROM IP core. This will not be repeated here.

Experiment Results
------------------

Connect the DAC output of AN108 to the ADC input using a BNC cable.\ **A dedicated shielded cable is used here. Using other cables may cause significant interference.**\ 

.. image:: images/23_media/image18.png
      
AN108 Connection Diagram

.. image:: images/23_media/image19.png
      
J11 Expansion Port

Adjust the frequency and amplitude of the signal generator. The AN108 input range is -5V to 5V. For easier observation of waveform data, it is recommended to set the signal input frequency between 200KHz and 1MHz. Observe the monitor output: the red waveform represents the ADC input; in the yellow grid, the top horizontal line represents 5V, the bottom horizontal line represents -5V, the middle horizontal line represents 0V, and each vertical line interval represents 10 sampling points.

.. image:: images/23_media/image2.png
      