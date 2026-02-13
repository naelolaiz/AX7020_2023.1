Using FreeRTOS on ZYNQ
======================

**The Vivado project for this experiment is "freeos_test".**

Many ZYNQ learners are FPGA developers who are not very familiar with Linux, so it is recommended to start with a real-time operating system or bare-metal development, which also offers greater flexibility. This chapter explains how to set up the FreeRTOS real-time operating system runtime environment, without delving deeply into the specific usage of FreeRTOS. This experiment uses the FreeRTOS Hello World as an example and implements two LEDs blinking continuously at different intervals.

This experiment is based on the "Dual-Core AMP Usage" project, and no modifications to the hardware environment are required.

Vitis Program Development
-------------------------

1. Create a new project and select freertos10_xilinx as the OS Platform

.. image:: images/11_media/image1.png
      
2. This experiment selects FreeRTOS Hello World as an example

.. image:: images/11_media/image2.png
      
The Hello World example creates two tasks: a send task and a receive task, where the receive task has a higher priority than the send task. A queue is also created, through which the send task sends data to the queue and the receive task reads data from the queue and prints it. The example originally sets up a timer, but in this experiment the timer is removed to allow the send and receive tasks to run continuously.

.. image:: images/11_media/image3.png
      
3. On this basis, add LED blinking tasks for both the PS and PL sides, with the PS side blinking interval set to 100ms and the PL side blinking interval set to 1S

.. image:: images/11_media/image4.png
      
Board Verification
------------------

1. Configure the download interface and download the program

.. image:: images/11_media/image5.png
      
2. Open PuTTY, data is being printed continuously

.. image:: images/11_media/image6.png
      
3. At the same time, you can see the PS side and PL side LEDs blinking on the development board, which intuitively demonstrates multi-task parallel processing.

(For the AX7015 board: PS_LED and PL_LED4; for the AX7021 board: LED1 and LED2; for the AX7020/AX7010 board: PS LED1 and PL LED1; for the AX7Z035/AX7Z100 board: LED1 and LED2)

Chapter Summary
---------------

Compared to the complexity of Linux, real-time operating systems such as FreeRTOS offer more flexible and convenient development, allowing more direct interaction with the underlying FPGA. However, FreeRTOS itself has a certain learning curve, and to become proficient, it is necessary to practice more with real-world projects.
