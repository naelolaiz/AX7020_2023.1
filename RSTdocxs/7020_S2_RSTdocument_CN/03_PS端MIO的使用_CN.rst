Using PS MIO
=============

**The Vivado project for this experiment is "ps_mio".**

This chapter introduces the operation of PS MIO. MIO is a basic peripheral IO that can be connected to peripherals such as SPI, I2C, UART, GPIO, etc. Through VIVADO software configuration, signals can be routed out via MIO, and signals can also be connected to PL pins via EMIO.

MIO has two BANKs. BANK0 has 16 pins, and BANK1 has 38 pins, for a total of 54 pins. The voltage of the two BANKs must be configured correctly.

.. image:: images/03_media/image1.png
      
This experiment demonstrates MIO operation by implementing PS LED blinking.

Principle Introduction
----------------------

First, let's understand the GPIO BANK distribution. In the GPIO chapter of the UG585 document, you can see that GPIO has 4 BANKs. Note the distinction from MIO BANKs.
BANK0 controls 32 signals, and BANK1 controls 22 signals, totaling 54 MIO pins, which correspond to PS peripheral interfaces such as SPI, I2C, USB, SD, etc.
BANK2 and BANK3 can control a total of 64 PL pins. Note that each group has three signals: input EMIOGPIOI, output EMIOGPIOO, and output enable EMIOGPIOTN, similar to a tri-state gate, totaling 192 signals. These can be connected to PL pins and controlled by the PS.

.. image:: images/03_media/image2.png
      
Vivado Project Setup
--------------------

This experiment is based on the "ps_hello" project saved as "ps_mio". To control the PS MIO, GPIO MIO needs to be enabled, which has already been configured previously.

.. image:: images/03_media/image3.png
            
1. Since there is no need to generate an FPGA bitstream file, directly click File > Export > Export Hardware without checking the include bitstream option to generate the Hardware information. A new Vitis directory will be generated at this point.

.. image:: images/03_media/image4.png
            
Software Engineer Tasks
-----------------------

The following is the content handled by software engineers.

Vitis Program Development
-------------------------

MIO Lighting PS LED
~~~~~~~~~~~~~~~~~~~~

According to the schematic, the LEDs of AX7020 and AX7010 are connected to PS MIO0 and MIO13. You can control the LEDs based on the corresponding MIO positions on the development board.

.. image:: images/03_media/image5.png
      
AX7020/AX7010 Schematic

1. Click Tools > Launch Vitis to enter Vitis

.. image:: images/03_media/image6.png
            
2. The process of creating a new platform project will not be repeated here. Please refer to the "PS Timer Interrupt Experiment" chapter.

.. image:: images/03_media/image7.png
      
3. The figure below shows the GPIO control block diagram. The experiment will use the output section registers: data register DATA, data mask registers MASK_DATA_LSW and MASK_DATA_MSW, direction control register DIRM, and output enable controller OEN.

.. image:: images/03_media/image8.png
      
4. Now let's look at the GPIO registers. You can open the Register Details at the bottom of the UG585 document and find the General Purpose I/O section.

.. image:: images/03_media/image9.png
      
5. Registers that may be used in the experiment:

Data mask register: For example, MIO 9 is in GPIO BANK0, and you can mask the other 31 bits in BANK0.

.. image:: images/03_media/image10.png
      
Direction register, controlling the data direction

.. image:: images/03_media/image11.png
      
Output enable register

.. image:: images/03_media/image12.png
      
Data register, valid data

.. image:: images/03_media/image13.png
      
The specific meanings of each register will not be explained one by one here. Please study them on your own.

6. When first writing code, you may not know where to start. We can import an example project provided by Xilinx. Open BSP, find ps7_gpio_0, and click Import Examples.

.. image:: images/03_media/image14.png
      
In the popup window, select "xgpiops_polled_example" and click OK.

.. image:: images/03_media/image15.png
      
A new APP project will appear.

.. image:: images/03_media/image16.png
      
7. This example project tests the input and output of PS MIO. Since the PS LEDs on the development board are MIO0 and MIO13, you need to modify Output_pin to 0 in the file to test the MIO0 LED.

.. image:: images/03_media/image17.png
      
Since we are only testing the LED, i.e., the output, we comment out the input function. Save the file.

.. image:: images/03_media/image18.png
      
8. Build the project

.. image:: images/03_media/image19.png
            
9. Run As > Launch on Hardware (Single Application Debug). After the download is complete, you can see PS_LED1 blinking rapidly 16 times.

.. image:: images/03_media/image20.png
            
You can also change it to MIO13 to observe the change of PS_LED2.

10. Although using the official example is convenient, its code looks rather bloated. We can learn from its approach and write a simplified version ourselves. Let's create a new APP project. You can right-click in the blank area and select New > Application Project. Modify it in the helloworld.c of ps_led_test. The program steps are actually quite simple: initialize GPIO, set direction, enable output, and control GPIO output value.

.. image:: images/03_media/image21.png
      
11. Select platform

.. image:: images/03_media/image22.png
      
12. Select Domain. The concept of Domain is similar to BSP.

.. image:: images/03_media/image23.png
      
13. Select Hello World as the template.

.. image:: images/03_media/image24.png
      
14. You can see that a new APP project has been added. It is still based on the BSP named standalone on ps7_cortexa9_0, which is a Domain, sharing the same BSP with the previous example project.

.. image:: images/03_media/image25.png
            
15. You can copy the example code to helloworld.c, save and Build Project.

.. image:: images/03_media/image26.png
            
16. The download method is the same as before, and you can see PS LED1 and LED2 start blinking.

MIO Button Interrupt
~~~~~~~~~~~~~~~~~~~~

The previous section introduced using MIO as output to control LEDs. Here we will discuss using MIO as button input to control LEDs.

1. Let's look at the GPIO structure diagram from the UG585 document. The interrupt registers are:

INT_MASK: Interrupt mask

INT_DIS: Interrupt disable

INT_EN: Interrupt enable

INT_TYPE: Interrupt type, setting level-sensitive or edge-sensitive

INT_POLARITY: Interrupt polarity, setting low level/falling edge or high level/rising edge

INT_ANY: Edge trigger mode, requires INT_TYPE to be set to edge-sensitive to use

When configuring the interrupt generation method, INT_TYPE, INT_POLARITY, and INT_ANY need to be used together. For specific register meanings, please refer to the UG585 Register Details section.

.. image:: images/03_media/image27.png
      
From the schematic, you can see that the PS buttons are connected to MIO50 and MIO51. This experiment uses MIO50.

|image1|\ |image2|

AX7020/AX7010 Schematic

2. This experiment is designed so that pressing the button turns the LED on, and pressing it again turns the LED off.

The main program design flow is as follows:

GPIO initialization -> Set button and LED direction -> Set interrupt generation method -> Set interrupt -> Enable interrupt controller -> Enable interrupt exception -> Enable GPIO interrupt -> Check KEY_FLAG value, if 1, write LED

Interrupt handling flow:

Query interrupt status register -> Check status -> Clear interrupt -> Set KEY_FLAG value

3. Create a new Vitis project

.. image:: images/03_media/image30.png
      
4. Define the PS button number as 50 and PS LED as 0

.. image:: images/03_media/image31.png
      
5. In the main function, set up the LED and button, and configure the button interrupt type to trigger on the rising edge. In this experiment, the rising edge of the button signal generates an interrupt.

.. image:: images/03_media/image32.png
      
6. The interrupt controller setup function IntrInitFuntions is based on the PS timer interrupt experiment, and the following statements set the interrupt priority and trigger mode, i.e., operating the ICDIPR and ICDICFR registers.

.. image:: images/03_media/image33.png
      
7. In the interrupt service routine GpioHandler, check the interrupt status register, clear the interrupt, and set the button flag to 1.

.. image:: images/03_media/image34.png
      
8. In the main function, check the button flag key_flag and write data to the LED.

.. image:: images/03_media/image35.png
      
9.  Build the project and download the program

10. Observe the experimental results. Pressing the PS button can control the PS LED on and off.

..

   AX7020/AX7010 development board silkscreen label is PS KEY1;

   PS LED location: AX7020/AX7010 development board silkscreen label is PS LED1;

Knowledge Sharing
-----------------

1. The include folder of the BSP in the platform contains various Xilinx header files. For example, the GPIO used in this chapter uses xgpiops.h. In this file, you can see various macro definitions that can be used when calling GPIO functions to improve readability.

.. image:: images/03_media/image36.png
      
It also contains the peripheral's built-in function declarations.

.. image:: images/03_media/image37.png
      
2. The xparameters.h header file defines the base addresses, device IDs, interrupts, etc. for each peripheral.

.. image:: images/03_media/image38.png
      
For example, the DEVICE_ID macro definition in the program is found in this file.

.. image:: images/03_media/image39.png
      
3. The libsrc folder contains peripheral function definitions and usage instructions.

.. image:: images/03_media/image40.png
      
4. In the lscript.ld file under the src folder, the available memory space, stack and heap sizes, etc. are defined and can be modified as needed.

.. image:: images/03_media/image41.png
      
5. Place the mouse cursor on a macro definition or function and press F3 to see where it is defined. You can also hold Ctrl and left-click to navigate to it. For example, clicking on DEVICE_ID below will navigate to xparameter.h.

.. image:: images/03_media/image42.png
      
.. image:: images/03_media/image43.png
      
Chapter Summary
---------------

This chapter introduced MIO input/output control and GPIO usage. We hope you now have a good understanding of these concepts. During the learning process, be sure to read the documentation frequently, and deepen your understanding by studying the module structure and register meanings. Reference document: UG585.

.. |image1| image:: images/03_media/image28.png
.. |image2| image:: images/03_media/image29.png