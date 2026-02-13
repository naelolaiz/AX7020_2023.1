Using AXI GPIO on the PL Side
================================

**The Vivado project for this experiment is "ps_axi_gpio".**

Some people may wonder why we are talking about GPIO and LED lights again, finding it too tedious. However, GPIO is a fundamental operation of ZYNQ. This tutorial aims to share various methods with everyone, including PS-side MIO, EMIO, PL-side AXI GPIO, covering both input and output directions, as well as basic PS and PL operations. So please be patient and keep learning.

Previously, we discussed how to use PS-side EMIO to light up PL-side LED lights, but there was no interaction with the PL side. This chapter introduces another control method. In ZYNQ, you can use AXI GPIO to control PL-side LED lights through the AXI bus. It also introduces the use of PL-side buttons.

The biggest question when using ZYNQ is how to combine PS and PL together. In other SOC chips, there are usually GPIOs available. This experiment uses an AXI GPIO IP core to let the PS side control PL-side LED lights through the AXI bus. Although the experiment is simple, it helps us understand how PL and PS are combined.

Principle Introduction
----------------------

An AXI GPIO module has two GPIOs, namely GPIO and GPIO2, which are channel1 and channel2 respectively, and they are bidirectional IOs.

.. image:: images/05_media/image1.png
      
AXI GPIO Structure

FPGA Engineer's Work
--------------------

The following is the responsibility of the FPGA engineer.

Creating the Vivado Project
---------------------------

1) Open "ps_hello" and save it as a Vivado project named "ps_axi_gpio", indicating that PS controls GPIO through the AXI bus

.. image:: images/05_media/image2.png
      
.. image:: images/05_media/image3.png
      
2) Double-click xx.bd to open the block design

.. image:: images/05_media/image4.png
      
Adding AXI GPIO
~~~~~~~~~~~~~~~

3) Add an AXI GPIO IP core

.. image:: images/05_media/image5.png
      
4) Double-click the newly added "axi_gpio_0" to configure its parameters

.. image:: images/05_media/image6.png
      
5) Select "All Outputs", since we are controlling LEDs here and only need output. Set "GPIO Width" to 4 to control 4 LEDs, then click OK. If you want to use channel2, you need to enable "Enable Dual Channel", which enables GPIO2.

.. image:: images/05_media/image7.png
      
6) Click "Run Connection Automation" to complete partial automatic wiring

.. image:: images/05_media/image8.png
      
7) Select the ports to be automatically connected. Select all here and click OK

.. image:: images/05_media/image9.png
      
8) Click "Optimize Routing" to optimize the layout. You can also see that two additional modules have been added. One is the Processor System Reset module, which is a synchronous reset module that provides reset signals within the same clock domain. The AXI Interconnect module is an AXI bus interconnect module used for cross-interconnection of AXI modules.

.. image:: images/05_media/image10.png
      
In this application, we can see that the ZYNQ GP port M_AXI_GP0 is used, where M stands for master. This interface is used to access PL-side data, and in most applications, it is used to configure registers of PL-side modules.

.. image:: images/05_media/image11.png
      
The reset signal is provided by the ZYNQ reset output. It is best to add a reset module for each clock domain. You can search and add them based on the module names shown below.

.. image:: images/05_media/image12.png
      
9) Modify the GPIO port name

.. image:: images/05_media/image13.png
      
10) Change the name to leds

.. image:: images/05_media/image14.png
      
11) Add another AXI GPIO to connect to PL-side buttons

.. image:: images/05_media/image15.png
      
12) Configure the GPIO parameters: all inputs, width of 1, and enable interrupts

.. image:: images/05_media/image16.png
      
13) Use automatic connection

.. image:: images/05_media/image17.png
      
14) Change the port name to keys

.. image:: images/05_media/image18.png
      
15) Since the interrupt comes from the PL side, we need to configure the ZYNQ processor interrupt here. Check IRQ_F2P

.. image:: images/05_media/image19.png
      
16) Connect ip2intc_irpt to IRQ_F2P

.. image:: images/05_media/image20.png
      
17) Save the design, click on xx.bd, right-click and select Generate Output Products

.. image:: images/05_media/image21.png
      
18) In the generated Verilog file, you can see there are ports named "leds_tri_o" and "keys_tri_i". You need to assign pins to them. When binding pins, use the pin names from this file as the reference.

.. image:: images/05_media/image22.png
      
XDC File for PL Pin Constraints
-------------------------------

1. Create a new XDC constraint file

.. image:: images/05_media/image23.png
      
2. Name the file led

.. image:: images/05_media/image24.png
      
3. Add the following content to led.xdc. The port names must match the top-level file ports

::

 set_property IOSTANDARD LVCMOS33 [get_ports {leds_tri_o[3]}]
 set_property IOSTANDARD LVCMOS33 [get_ports {leds_tri_o[2]}]
 set_property IOSTANDARD LVCMOS33 [get_ports {leds_tri_o[1]}]
 set_property IOSTANDARD LVCMOS33 [get_ports {leds_tri_o[0]}]
 set_property PACKAGE_PIN M14 [get_ports {leds_tri_o[0]}]
 set_property PACKAGE_PIN M15 [get_ports {leds_tri_o[1]}]
 set_property PACKAGE_PIN K16 [get_ports {leds_tri_o[2]}]
 set_property PACKAGE_PIN J16 [get_ports {leds_tri_o[3]}]
 
 set_property IOSTANDARD LVCMOS33 [get_ports {keys_tri_i[0]}]
 set_property PACKAGE_PIN N15 [get_ports {keys_tri_i[0]}]

4. Generate the bit file

.. image:: images/05_media/image25.png
      
5. Export hardware: File > Export > Export Hardware

.. image:: images/05_media/image26.png
         
6. Since PL is used, select "Include bitstream" and click "OK"

Software Engineer's Work
------------------------

The following is the responsibility of the software engineer.

Vitis Programming
-----------------

Using AXI GPIO to Light Up PL-Side LEDs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1) Create a platform. Refer to the "PS Timer Interrupt Experiment" chapter for the creation process

.. image:: images/05_media/image27.png
      
2) When facing an unfamiliar AXI GPIO, how do we control it? We can try the built-in examples provided by Vitis

3) Double-click "BSP in platform.spr", find "axi_gpio_0". You can click "Documentation" to view the related documentation, which we won't demonstrate here. Click "Import Examples"

.. image:: images/05_media/image28.png
      
4) In the pop-up dialog, there are multiple examples. You can roughly guess their functions from the names. Select the first one "xgpio_example"

.. image:: images/05_media/image29.png
      
5) You can see that the example is quite simple. With just a few lines of code, it completes the AXI GPIO operation

.. image:: images/05_media/image30.png
      
Many GPIO-related API functions are used inside. You can learn the details through the documentation, or select a function and press "F3" to view its specific definition. If you still cannot understand how to use AXI GPIO with this information, it means you need to strengthen your C language fundamentals.

In fact, these functions are all operating GPIO registers. There are not many AXI GPIO registers. The main ones are the data registers GPIO_DATA and GPIO2_DATA for the two channels, the direction control registers GPIO_TRI and GPIO2_TRI for the two channels, the global interrupt enable register GIER, the IP interrupt enable register IP IER, and the interrupt status register ISR. For specific functions, refer to the AXI GPIO document pg144.

.. image:: images/05_media/image31.png
      
For example, when entering the function that sets the GPIO direction, you can see that it writes data to the GPIO_TRI register of GPIO to control the direction.

.. image:: images/05_media/image32.png
      
Other functions can be studied in the same way.

Download and Debug
~~~~~~~~~~~~~~~~~~

1) First, compile the APP project. The compilation method has been introduced in previous examples. Although Vitis can provide some examples, some of them need to be modified by yourself. We won't modify this simple LED example. Try running it and you may find it does not achieve the expected result, and may even show some errors. After downloading, you can see that PL
   LED1 on the development board blinks rapidly.

.. image:: images/05_media/image33.png
      
2) Modify the code to make all 4 LEDs blink

.. image:: images/05_media/image34.png
      
Register-Based Implementation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you find the API functions provided by Xilinx too cumbersome and inefficient, you can also control LEDs by directly operating registers.

For example, below we created a new axi_led project and modified helloworld.c as follows.

.. image:: images/05_media/image35.png
      
.. image:: images/05_media/image36.png
      
The base address GPIO_BASEADDR defined here can be found in xx.xsa

.. image:: images/05_media/image37.png
      
Since we only enabled channel1, we defined the following register addresses

.. image:: images/05_media/image38.png
      
Directly operating registers in this way is more efficient than calling Xilinx API functions, and it is more intuitive, which is very helpful for understanding how the program runs. However, for large projects, this approach becomes more complex to use. The choice mainly depends on personal needs.

AXI GPIO PL-Side Button Interrupt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The interrupt in the previous timer interrupt experiment belongs to the PS internal interrupt. In this experiment, the interrupt comes from the PL side. The PS can receive up to 16 interrupt signals from the PL, all triggered by rising edge or high level.

.. image:: images/05_media/image39.png
      
1) As with the previous tutorials, when unfamiliar with Vitis programming, we try to use built-in Vitis examples and modify them. Select "xgpio_intr_tapp_example"

.. image:: images/05_media/image40.png
      
2) After importing the example, there are undefined errors. We need to modify some code. You can go back to the Vivado project to see that the button's AXI GPIO module is called axi_gpio_1, along with its offset address

.. image:: images/05_media/image41.png
      
Therefore, you can find its device id in xparameters.h

.. image:: images/05_media/image42.png
      
.. image:: images/05_media/image43.png
      
.. image:: images/05_media/image44.png
      
3) Then modify the GPIO and interrupt number macro definitions as follows

.. image:: images/05_media/image45.png
      
4) Modify the test delay time to give us enough time to press the button

.. image:: images/05_media/image46.png
      
.. _下载调试-1:

Download and Debug
~~~~~~~~~~~~~~~~~~

1) Save the file, compile the project, open the serial terminal, and download the program. If no button is pressed, the serial port displays "No button pressed." If you press the "PL KEY1" button, it displays "Successfully ran Gpio Interrupt Tapp Example".

.. image:: images/05_media/image47.png
      
Experiment Summary
------------------

Through this experiment, we learned that PS can control PL through the AXI bus, but this barely demonstrates the advantages of ZYNQ. For controlling LED lights, whether it is ARM or FPGA, both can easily accomplish it. But what if we replace LEDs with serial ports? Controlling 100 serial communication channels, 8 Ethernet interfaces, and other applications - I believe no other SOC can accomplish such functions. Only ZYNQ can, and this is the difference between ZYNQ and ordinary SOCs.

The PL side can send interrupt signals to the PS, which improves the efficiency of data interaction between PL and PS. Interrupt handling is needed in applications that require large data volumes and low latency.

By the end of this chapter, we have covered how to use PS-side MIO, EMIO, and PL-side GPIO of ZYNQ, including input, output, and interrupt handling. These are the most fundamental operations, and everyone should think more and understand them clearly.

Knowledge Sharing
-----------------

1) After the design is complete, you can see in the Address Editor that address spaces have been allocated for AXI peripherals. The offset address and space size can be modified.

.. image:: images/05_media/image48.png
      
However, there are restrictions on modifying the offset address. For details, refer to the System Address chapter of the UG585 document. AXI peripherals are connected to the M_AXI_GP0 port and can be modified within the address space from 4000_0000 to 7FFF_FFFF.

.. image:: images/05_media/image49.png
      
2) When using a module, supporting documentation is needed for development. But how do you find these documents? For example, for Xilinx IPs, open the module configuration, click Documentation in the upper left corner, then click Product Guide. If DocNav was installed when Vivado was installed, it will redirect and open the document.

.. image:: images/05_media/image50.png
      
.. image:: images/05_media/image51.png
      
.. image:: images/05_media/image52.png
      
This feature requires an internet connection, as DocNav loads documents from the website. You can click the download button to save them locally.

Another method is to search for and download materials by module name on the Xilinx official website (the page may change over time)

.. image:: images/05_media/image53.png
      