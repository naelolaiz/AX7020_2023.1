PS Timer Interrupt Experiment
==============================

**The Vivado project for this experiment is "ps_timer" (saved from the "ps_hello" project).**

Many SOCs have built-in timers, and so does the PS of ZYNQ. Developers must be aware of what peripherals are inside the ZYNQ and what features these peripherals have. Therefore, it is recommended to frequently read the Xilinx document UG585. The timer used in this chapter's experiment is the CPU Private Timer.

.. image:: images/02_media/image1.png
      
ZYNQ Timer Architecture Diagram

Open the "ps_hello" project and save it as a new project named "ps_timer".

Interrupt Introduction
----------------------

Referring to the interrupt section of UG585, Zynq interrupts can be roughly divided into three parts: 1) SGI (Software Generated Interrupts), with 16 ports; 2) PPI (CPU Private Peripheral Interrupts), with 5 ports; 3) SPI (Shared Peripheral Interrupts), from 44 PS-side IO peripherals and 16 PL-side interrupts. The middle part is the GIC (General Interrupt Controller), used to enable, disable, mask, and set priorities for interrupts.

.. image:: images/02_media/image2.png
      
Below is the interrupt controller block diagram. The main controller parts are ICC and ICD. ICD connects to SGI and PPI, ICD connects to SPI. You can configure the registers of both to control interrupts.

.. image:: images/02_media/image3.png
      
SGI Interrupts (Software Generated Interrupts), 16 IRQ ID numbers in total

.. image:: images/02_media/image4.png
      
PPI Interrupts, CPU Private Interrupts, 5 IRQ ID numbers in total

.. image:: images/02_media/image5.png
      
.. image:: images/02_media/image6.png
      
SPI Interrupt section, 60 IRQ ID numbers in total

.. image:: images/02_media/image7.png
      
.. image:: images/02_media/image8.png
      
Interrupt Register Introduction
-------------------------------

Interrupts can be well controlled using Xilinx's API functions. If you are interested, you can delve deeper into the interrupt registers to gain a better understanding of the mechanism.

.. image:: images/02_media/image9.png
      
ICDICFR:
Configuration register, used to configure the trigger mode, either level-triggered or edge-triggered. There are 6 registers in total, each 32 bits wide. Every two bits represent one interrupt, 32*6/2=96 interrupt numbers, covering all interrupts.

ICDICFR0: IRQ ID#0~#15

ICDICFR1: IRQ ID#16~#31

ICDICFR2: IRQ ID#32~#47

ICDICFR3: IRQ ID#48~#63

ICDICFR4: IRQ ID#64~#79

ICDICFR5: IRQ ID#80~#95

For SPI interrupts: 0b01: high-level triggered, 0b11: rising-edge triggered

ICDIPR:
Interrupt priority register, used to set priority. There are 24 registers in total, with every 8 bits representing one interrupt number, totaling 96 interrupt numbers.

ICDIPTR: CPU selection register, 24 registers, every 8 bits represent one interrupt number, 96 in total

0bxxxxxxx1: CPU interface 0

0bxxxxxx1x: CPU interface 1

ICDICER: Interrupt disable register, 3 registers, every 1 bit represents one interrupt number, 96 in total

ICDISER: Interrupt enable register, 3 registers, every 1 bit represents one interrupt number, 96 in total

For the remaining registers, you can study the mpcore section in the register table of UG585.

.. image:: images/02_media/image10.png
      
Software Engineer's Work
------------------------

The following is the content that the software engineer is responsible for.

Vitis Program Development
-------------------------

Creating a Platform Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1) Click Tools -> Launch Vitis

.. image:: images/02_media/image11.png
      
2) Unlike the previous Hello World experiment, we only create a Platform project

.. image:: images/02_media/image12.png
         
3) Enter the project name and click Next

.. image:: images/02_media/image13.png
         
4) Click "Create a new platform hardware (XSA)". The software already provides some board hardware platforms, but for

5) our own hardware platform, we can select browse

.. image:: images/02_media/image14.png
         
6) Select the XSA file

.. image:: images/02_media/image15.png
         
Keep the defaults and click Finish

.. image:: images/02_media/image16.png
      
7) Open platform.spr and expand BSP

.. image:: images/02_media/image17.png
      
8) Find the timer driver and click Import Examples

.. image:: images/02_media/image18.png
      
9) Fortunately, there is a timer interrupt example. How do we know this example is an interrupt example? It is guessed from the "intr" keyword. So, having a solid foundation is important, otherwise you won't even be able to find example programs.

.. image:: images/02_media/image19.png
      
10) The example project is now imported here

.. image:: images/02_media/image20.png
      
Next, we read the code and then modify it. Of course, you may not fully understand the code right away, and can only improve through repeated practice in future applications.

11) This experiment designs a timer that triggers an interrupt once every second, prints information, and ends after 30 seconds. From the UG585 document, we learn that the timer clock frequency is half of the CPU frequency. First, we need to modify the counter maximum value to half the CPU frequency, which is the timer clock frequency value. This way, an interrupt will occur once every second.

.. image:: images/02_media/image21.png
      
.. image:: images/02_media/image22.png
      
The macro definition for CPU frequency can be found in xparameters.h

.. image:: images/02_media/image23.png
      
12) Change the count number from 3 to 30

.. image:: images/02_media/image24.png
      
13) Add print information and save the file

.. image:: images/02_media/image25.png
      
14) Build Project to compile

.. image:: images/02_media/image26.png
      
15) Let's understand the usage of the interrupt controller. It mainly consists of several steps: initialize the interrupt controller, \ *GIC initialization, interrupt exception, interrupt service function registration, enable interrupt in the interrupt controller, enable peripheral interrupt, enable interrupt exception*\ . Two steps need attention: \ *enabling interrupt in the interrupt controller*
means enabling the corresponding interrupt based on the interrupt number. For example, the Timer introduced in this chapter is a private timer with interrupt number 29, which is an operation in the interrupt controller GIC. The subsequent \ *enabling peripheral interrupt*
refers to enabling the interrupt within the peripheral itself. Under normal circumstances, it is not enabled. Once enabled, the interrupt can be generated and passed to the interrupt controller GIC. This approach can be referenced in future experiments.

.. image:: images/02_media/image27.png
      
.. image:: images/02_media/image28.png
      
Download and Debug
------------------

1) Open the PuTTY serial terminal

2) The method for downloading and debugging the program has been explained in previous tutorials and will not be repeated here

3) As expected, the serial port outputs a message every second

.. image:: images/02_media/image29.png
      
Experiment Summary
------------------

In this experiment, by simply modifying the Vitis example program, we completed the timer and interrupt application. Although the operation seems simple, it contains a wealth of knowledge. We need to thoroughly understand the principles of timers and interrupts. This fundamental knowledge is a necessary condition for learning ZYNQ well.
