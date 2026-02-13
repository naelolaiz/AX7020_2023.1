DMA Loopback Test
==================

**The Vivado project for this experiment is "dma_loopback".**

This chapter introduces an important functional module, DMA (Direct Memory
Access), which is an interface technology that allows external devices to exchange data directly with system memory without going through the CPU. To read peripheral data into memory or transfer memory data to peripherals, it is generally done through CPU control, such as polling or interrupt methods, as in the BRAM experiment discussed earlier.

Although the interrupt method can improve CPU utilization, there are still efficiency issues. For bulk data transfers, using DMA can solve efficiency and speed problems. The CPU only needs to provide the address and length to the DMA, and the DMA can then take over the bus to access memory. After the DMA completes its work, it notifies the CPU and relinquishes bus control.

In this chapter, we use the DMA example provided by Vitis with slight modifications. After the DMA finishes its work, it issues a completion interrupt to notify the CPU, allowing the CPU to read memory data.

.. image:: images/15_media/image1.png
      
Experiment Description
----------------------

Refer to DMA document PG021

1. Let's first understand the AXI DMA module. This module uses three types of buses: AXI4-Lite is used for register configuration, AXI4 Memory Map is used for memory interaction. Within this module, there are two separate interfaces: AXI4 Memory Map Read and AXI4 Memory Map Write, also called M_AXI_MM2S and M_AXI_S2MM respectively - one for reading and one for writing. It is important to understand and not confuse these. The AXI4 Stream interface is used for reading and writing to peripherals, where AXI4 Stream Master (MM2S) is used for writing to peripherals, and AXI4-Stream Slave(S2MM) is used for reading from peripherals. It also supports Scatter/Gather functionality, but this experiment will not cover it, leaving it for users to explore. (MM2S stands for Memory Map to Stream, S2MM stands for Stream to Memory Map).

..

   AXI Memory Map data width supports 32, 64, 128, 256, 512, 1024 bits

   AXI Stream data width supports 8, 16, 32, 64, 128, 256, 512, 1024 bits

.. image:: images/15_media/image2.png
      
2. This experiment uses direct register mode. The register description is shown in the figure below, mainly divided into two parts: one is MM2S, which includes Control Register, Status Register, Source Address, and Transfer Length; the other is S2MM, which similarly includes Control Register, Status Register, Destination Address, and Buffer Length. Note that Source Address and Destination Address here refer to memory addresses.

.. image:: images/15_media/image3.png
      
.. image:: images/15_media/image4.png
      
1. The following is the MM2S_DMACR control register description. The most important bit is Bit0, Run/Stop, which starts or stops the DMA.

Other details will not be discussed here.

.. image:: images/15_media/image5.png
      
.. image:: images/15_media/image6.png
      
There are several interrupts that can be configured here: IOC_IrqEn enables the completion interrupt, Dly_IrqEn enables the delay interrupt, and Err_IrqEn enables the error interrupt.

.. image:: images/15_media/image7.png
      
4. MM2S_DMASR is the status register description. Bits 12, 13, and 14 are interrupt status bits, and writing 1 clears the interrupt.

.. image:: images/15_media/image8.png
      
.. image:: images/15_media/image9.png
      
5. MM2S_DA and MM2S_LENGTH represent the address and length settings. The S2MM registers are similar to MM2S and will not be discussed further. The function of this experiment is for MM2S to read data from DDR3, write it to the AXI Stream Data FIFO, then read from the FIFO and write back to DDR3, implementing a loopback test. The IOC_Irq in S2MM_DMACR needs to be enabled, which is the write-to-memory completion interrupt. The functional block diagram is shown below:

.. image:: images/15_media/image10.png

Hardware Environment Setup
--------------------------

1. Based on the "ps_hello" project. In this experiment, the HP interface is needed for high-speed DDR3 access:

.. image:: images/15_media/image11.png
      
Configure as follows:

.. image:: images/15_media/image12.png
      
2. Enable the PL-PS interrupt interface to connect the DMA interrupt

.. image:: images/15_media/image13.png
      
3. Set the clock to 100MHz for the PL-side AXI clock

.. image:: images/15_media/image14.png
      
4. Click "+" to add the DMA module.

.. image:: images/15_media/image15.png
      
5. The DMA settings are as follows. Width of Buffer Length Register refers to the width of the LENGTH register. For example, 23 bits means a maximum transfer of 2^26 = 67,108,864 bytes. Here we use the default setting of 14. Enable both read and write channels, but do not enable Allow Unaligned Transfers. If not enabled and Memory Map Data Width is set to 32, then addresses must be 0x0, 0x4, 0x8, 0xC, etc., aligned to 4 bytes. Max Burst Size can be set to 2, 4, 8, 16, 32, 64, 128, 256.

.. image:: images/15_media/image16.png
      
1. The AXI Stream Data FIFO settings are as follows: set the depth to 1024, TDATA Width to 4 bytes (i.e., 32 bits), and enable the TKEEP and TLAST signals.

.. image:: images/15_media/image17.png
      
7. Run automatic connection

.. image:: images/15_media/image18.png
      
Continue with automatic connection

.. image:: images/15_media/image19.png
      
8. Connect the FIFO's S_AXIS and M_AXIS to the DMA (AXIS is short for AXI Stream), then continue to click Run Connection Automation

.. image:: images/15_media/image20.png
      
9. Add Concat, and connect the MM2S and S2MM interrupt outputs to IRQ_F2P

.. image:: images/15_media/image21.png
      
10. The final connections are shown in the figure below

.. image:: images/15_media/image22.png
      
11. Select the FIFO's S_AXI, M_AXI, and count signals, right-click and select Debug to add an ILA logic analyzer for observing data changes.

.. image:: images/15_media/image23.png
      
.. image:: images/15_media/image24.png
      
12. After automatic connection, open the ILA configuration

.. image:: images/15_media/image25.png
      
Change Number of Probes to 4, adding two Probe interfaces

.. image:: images/15_media/image26.png
      
Connect the two newly added Probes to the DMA interrupt outputs

.. image:: images/15_media/image27.png
      
13. Save the design and generate the bitstream

.. image:: images/15_media/image28.png
      
Vitis Program Development
-------------------------

1. The program for this experiment is a modification of the simple_poll example. You can learn about module usage by importing examples in the BSP.

.. image:: images/15_media/image29.png
      
2. Set MAX_PKT_LEN, which is the length in bytes, TEST_START_VALUE as the starting data value, and NUMBER_OF_TRANSFERS as the number of test iterations.

.. image:: images/15_media/image30.png
      
3. Define the transmit and receive arrays

.. image:: images/15_media/image31.png
      
4. In the XAxiDma_Setup function, enable the S2MM IOC interrupt and disable all MM2S interrupts. An interrupt will be issued after S2MM finishes receiving data.

.. image:: images/15_media/image32.png
      
5. In the XAxiDma_Setup function, after initializing TxBufferPtr, the data in the Cache needs to be flushed to memory. This is very important because the DMA needs to access DDR3, and the CPU interacts with DDR3 through the Cache. Data is temporarily stored in the Cache and may not have been actually flushed to DDR3. If an external device (i.e., the DMA) wants to read the DDR3 values, the Cache data must be flushed to DDR3 so that the DMA can read the correct values. Call the Xil_DCacheFlushRange function, providing the memory address and length.

.. image:: images/15_media/image33.png
      
6. Enable the MM2S channel and S2MM channel.

.. image:: images/15_media/image34.png
      
7. The interrupt setup method is the same as in previous examples

.. image:: images/15_media/image35.png
      
8. In the interrupt service routine, first clear the interrupt. Since the data in DDR3 has been updated but the Cache data has not, the CPU needs to call the Xil_DCacheInvalidateRange function to invalidate the Cache data before reading from DDR3, so that the CPU can read the correct data from DDR3. The memory address and length also need to be provided.

.. image:: images/15_media/image36.png
      
9. The CPU then reads the data from DDR3 for comparison to verify data correctness.

.. image:: images/15_media/image37.png
      
Program Verification
--------------------

1. Select Debug As, use Debug mode, and click Debug

.. image:: images/15_media/image38.png
      
2. Open the ILA, set the trigger condition to the rising edge of axi_dma_0_s2mm_introut, and click Run

.. image:: images/15_media/image39.png
      
3. Return to the Vitis Debug interface, no need to set breakpoints, click Resume

.. image:: images/15_media/image40.png
      
4. At this point, you can see that the ILA has been triggered and you can observe the captured data.

.. image:: images/15_media/image41.png
      
5. In the serial port debugging tool, you can see the print information showing that interrupts occurred twice and the test was successful

.. image:: images/15_media/image42.png
      
6. You can also observe memory information in the Vitis debugger. Set breakpoints as shown in the figure below, placing a breakpoint in the interrupt service function

.. image:: images/15_media/image43.png
      
7. Re-run Run Configurations, then click the Resume button to run to the breakpoint. Add TxBufferPtr and RxBufferPtr in the Memory window to observe and compare the data

.. image:: images/15_media/image44.png
      
Chapter Summary
---------------

This chapter covers many topics, including using DMA for memory access, using DMA interrupts, observing data with the ILA logic analyzer, and handling Cache when the CPU reads and writes memory. Readers are encouraged to practice more and become proficient in using DMA.

As discussed earlier, accessing PS-side DDR through the HP port via the AXI bus is one method of data exchange between PS and PL. The DMA in this chapter is another method of PS and PL data exchange. Essentially, both methods are the same - they both access PS-side DDR. The difference is that one is implemented in PL-side code, which gives users more flexibility and control, but the drawback is that it requires writing code, which can be difficult for those unfamiliar with FPGA. The DMA approach gives control mainly to the PS side, where the PS configures DMA reads and writes. The advantage is that it is more intuitive, but it requires a solid software background.
