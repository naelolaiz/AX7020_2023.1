PS UART Read/Write Control
============================

**The Vivado project for this experiment is "ps_uart".**

In previous experiments, you may have noticed print messages being output, mainly by calling "xil_printf" or "printf". But what are they printed through? We recall that a serial port was configured before printing messages. Yes, it is indeed the serial port, but how do these functions call the serial port? In fact, we can see it in the definition of the "xil_printf" function — note that the outbyte function is what calls UART for printing.

.. image:: images/08_media/image1.png
      
By entering the outbyte function, we can see that it calls the PS UART function, which enables display through the serial port.

.. image:: images/08_media/image2.png
      
Besides printing messages, what if we want to use UART for data transmission? This chapter introduces the read/write control of the PS UART. In this experiment, a string of characters is sent out every 1 second. If data is received, an interrupt is generated, and the received data is sent back out.

The Vivado project is based on "ps_hello".

UART Module Introduction
------------------------

The following is the block diagram of the UART module. Both TxFIFO and RxFIFO are 64 bytes.

.. image:: images/08_media/image3.png
      
The following figure shows the four modes of UART.

.. image:: images/08_media/image4.png
      
You can use remote loopback
mode to test whether the physical circuit is functioning properly, using the API function XUartPs_SetOperMode.

.. image:: images/08_media/image5.png
      
Software Engineer's Tasks
-------------------------

The following is the content that software engineers are responsible for.

Vitis Program Development
-------------------------

1. The experiment flow is as follows:

**Main program flow:**

UART initialization —— Set UART mode —— Set data format —— Set interrupt —— Send UART data and check if data is received —— If received, send the received data; if not, wait 1 second, then continue sending data

**Interrupt program flow:**

Interrupt initialization —— Set receive FIFO trigger interrupt register to 1, meaning an interrupt is triggered upon receiving one data byte —— Enable receive trigger interrupt REMPTY and receive FIFO empty interrupt RTRIG

**Interrupt service routine:**

Check whether the status register indicates trigger or empty —— Clear the corresponding interrupt —— In trigger state, read RxFIFO data; in empty state, set the receive flag ReceivedFlag to 1

2. First, let's look at the registers. They are mainly divided into configuration registers for configuring UART mode and baud rate; interrupt register configuration; and transmit and receive register configuration.

.. image:: images/08_media/image6.png
      
3. Set the trigger level register, 6 bits, range 1~63.

.. image:: images/08_media/image7.png
      
4. The transmit part is relatively simple — just write data to TxFIFO. The receive part requires interrupts. Here, Intrpt_en_reg and Intrpt_dis_reg are mainly used. These two interrupt registers are generally used in pairs: enable the corresponding interrupt and disable the remaining interrupts.

.. image:: images/08_media/image8.png
      
You need to set the receive RTRIG interrupt, which is the trigger interrupt. Before that, set the trigger value, meaning when the number of data bytes in RxFIFO reaches the trigger value, an interrupt is generated. At the same time, enable the REMPTY empty interrupt to check whether it is empty.

.. image:: images/08_media/image9.png
      
5. In the interrupt service routine, read the value of the status register to determine whether the receive has a trigger or is empty.

.. image:: images/08_media/image10.png
      
.. image:: images/08_media/image11.png
      
6. In the main function, set the mode by directly calling functions. Set it to normal mode, with the data format configured as baud rate 115200, 8-bit data, no parity, and 1 stop bit. UartFormat is defined in uart_parameter.h.

.. image:: images/08_media/image12.png
      
.. image:: images/08_media/image13.png
      
7. The interrupt controller initialization can refer to the key interrupt method, as the usage is similar.

8. In the main function, set the trigger level to 1, and enable the trigger and empty interrupts.

.. image:: images/08_media/image14.png
      
9. The data send and receive functions are based on the UARTPS XUartPs_Send and XUartPs_Rev functions, but they would enable certain interrupts that did not meet expectations, so modifications were made.

.. image:: images/08_media/image15.png
      
A maximum 2000-byte buffer is set in the receive buffer, which can be modified as needed.

|image1|\ |image2|

10. In the interrupt service routine, the ReceivedBufferPtr pointer address and ReceivedByteNum are incremented by the number of bytes received. If the FIFO is empty, ReceivedFlag is set to 1. At the same time, data is written to the interrupt status register to clear the interrupt.

.. image:: images/08_media/image18.png
      
.. image:: images/08_media/image19.png
      
UG585 UART section — clearing interrupts

11. In the main function, clear ReceivedFlag and ReceivedByteNum to zero, and reset the ReceivedBufferPtr pointer.

.. image:: images/08_media/image20.png
      
12. In the UART send function, check whether TxFIFO is full; if not, continue sending until the count reaches NumBytes.

.. image:: images/08_media/image21.png
      
13. In the UART receive function, check whether the receive RxFIFO is empty; if not, continue reading data. NumBytes is the number of data bytes to read, but if the receive FIFO becomes empty before the count reaches this value, the function will also exit.

.. image:: images/08_media/image22.png
      
14. In addition to writing your own program, you can also import module examples from the BSP in platform.spr and refer to the programs provided by Xilinx for easier learning.

.. image:: images/08_media/image23.png
      
Board Verification
------------------

1. Next, download the program.

.. image:: images/08_media/image24.png
      
2. Open the serial port debugging tool in the project directory.

.. image:: images/08_media/image25.png
      
3. Set the parameters as shown below, open the serial port, and you will see the print messages.

.. image:: images/08_media/image26.png
      
4. Enter data in the send area and click manual send to see the data in the receive area.

.. image:: images/08_media/image27.png
      
Summary
-------

This chapter covered UART transmission and reception, as well as the use of interrupts. We hope that you develop good habits of reading documentation and understanding the principles, which will greatly improve your understanding of the system.

.. |image1| image:: images/08_media/image16.png
.. |image2| image:: images/08_media/image17.png
