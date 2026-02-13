PL Read/Write PS-Side DDR Data
===================================

**The experimental Vivado project is "pl_read_write_ps_ddr".**

Efficient interaction between PL and PS is the top priority in zynq 7000 soc development. We often need to send large amounts of data from the PL side to the PS side for real-time processing, or send PS-side processing results to the PL side for real-time processing. Conventionally, we would think of using DMA, but the various protocols are very cumbersome and lack flexibility. This lesson explains how to directly read and write PS-side DDR data through the AXI bus, which involves the AXI4 protocol, Vivado FPGA debugging, and more.

Using ZYNQ HP Ports
--------------------

HP in zynq 7000 SOC stands for High-Performance Ports. As shown in the figure below, there are 4 HP ports in total. HP ports are AXI Slave devices, and we can achieve high-bandwidth data exchange through these 4 HP interfaces.

.. image:: images/12_media/image1.png
         
Use the "ps_hello" project as a base and save it as a new project.

1. In the Vivado interface, the HP configuration is shown below (HP0~HP3). It includes enable control and data width selection, with options of 32 or 64-bit width. Disable the M AXI GP0 interface, as it is not needed in this experiment.

.. image:: images/12_media/image2.png
      
In our experiment, we enable HP0 configured with 64-bit width, using a 150MHz clock. The HP bandwidth is 150MHz \* 64bit, which provides sufficient bandwidth for applications such as video processing and ADC data acquisition.

.. image:: images/12_media/image3.png
      
2. As shown in the figure below, after configuring the HP port, the ZYNQ will have an additional AXI Slave port named S_AXI_HP0. However, these ports follow the AXI3 standard, while we commonly use the AXI4 protocol. Here we add one AXI Interconnect IP for protocol conversion (AXI3<->AXI4).

.. image:: images/12_media/image4.png
      
3. Export the FCLK_CLK0 clock and S00_AXI, and modify their names.

.. image:: images/12_media/image5.png
      
4. Right-click on a blank area and select Create Port to add a clock input interface.

.. image:: images/12_media/image6.png
      
.. image:: images/12_media/image7.png
      
5. Add a reset module and connect the clock as follows.

.. image:: images/12_media/image8.png
      
6. Click the S00_AXI interface and select the clock interface as axi_hp_clk.

.. image:: images/12_media/image9.png
      
7. Double-click the S00_AXI pin to configure it, and change it to an AXI4 interface.

.. image:: images/12_media/image10.png
      
Double-click again to open the configuration and set the AXI Burst length to the maximum of 256. If set simultaneously with the previous step, it may not succeed.

.. image:: images/12_media/image11.png
      
.. image:: images/12_media/image12.png
      
8. Export the reset pin, modify its name, and connect the corresponding signals.

.. image:: images/12_media/image13.png
      
9. Change the associated clock of the reset to axi_hp_clk.

.. image:: images/12_media/image14.png
      
10. In the Address Editor, click to automatically assign the address space.

.. image:: images/12_media/image15.png
      
11. Save the design and press F6 to validate the design.

.. image:: images/12_media/image16.png
      
12. Generate Output Products to generate the output files.

13. Add other HDL files.

.. image:: images/12_media/image17.png
      
PL-Side AXI Master
-------------------

AXI4 is relatively complex, but SOC developers must master it. For ZYNQ developers, it is recommended to modify existing template code as a starting point. For detailed information on the AXI protocol, refer to Xilinx UG761 AXI Reference Guide. Here we provide a brief overview.

AXI4 uses a READY/VALID handshake communication mechanism, where the master and slave modules perform handshakes on the data and address channels used for the operation before data communication begins. The main operation involves the transfer sender A waiting for the READY signal from the transfer receiver B, after which A sends the data along with the VALID signal to B simultaneously. This is a typical handshake mechanism.

.. image:: images/12_media/image18.jpeg
         
The AXI bus is divided into five channels:

-  Read address channel, containing ARVALID, ARADDR, ARREADY signals;

-  Write address channel, containing AWVALID, AWADDR, AWREADY signals;

-  Read data channel, containing RVALID, RDATA, RREADY, RRESP signals;

-  Write data channel, containing WVALID, WDATA, WSTRB, WREADY signals;

-  Write response channel, containing BVALID, BRESP, BREADY signals;

-  System channel, containing ACLK, ARESETN signals;

ACLK is the AXI bus clock, and ARESETN is the AXI bus reset signal, active low. The read/write data and read/write address signal widths are all 32-bit. READY and VALID are the corresponding channel handshake signals. The bits set to 1 in the WSTRB signal correspond to the valid data bytes in WDATA, and the WSTRB width is 32bit/8=4bit. BRESP and RRESP are the write response and read response signals respectively, both 2-bit wide, where 'h0 indicates success and other values indicate errors.

The read operation sequence is: the master and slave perform a read address channel handshake and transmit the address content, then perform a read data channel handshake and transmit the read content along with the read operation response, effective on the rising edge of the clock. As shown in the figure:

.. image:: images/12_media/image19.png
      
The write operation sequence is: the master and slave perform a write address channel handshake and transmit the address content, then perform a write data channel handshake and transmit the write content, and finally perform a write response channel handshake and transmit the write response data, effective on the rising edge of the clock. As shown in the figure:

.. image:: images/12_media/image20.png
      
When we are not proficient in writing certain FPGA code, we often reference others' code or use IP cores. Here, an AXI master code was found on GitHub at https://github.com/aquaxis/IPCORE/tree/master/aq_axi_vdma. This project is a custom VDMA implementation that contains a large amount of referenceable code. The aq_axi_master.v code is primarily used here for AXI master read/write operations. Referencing others' code can sometimes save a lot of time, but if you borrow code without understanding it, problems will be difficult to resolve. Please refer to the aq_axi_master.v code for details, with some modifications applied.

DDR Read/Write Data Verification
---------------------------------

With the AXI Master read/write interface available, a simple verification module was written. This verification module was originally used to verify DDR IP by writing incrementing data every 8 bits and then reading it back for comparison. Note the starting address and size of the PS-side DDR, as well as whether the address unit is byte or word. The AXI bus address unit is byte, while the test module address unit is word (where a word is not necessarily 4 bytes). The file name is mem_test.v.

Vivado Software Debugging Tips
-------------------------------

The AXI read/write verification module has only one error signal to indicate errors. If there are data errors, we want more precise information. Altera's Quartus II software has the Signal Tap tool, and Xilinx's ISE has the ChipScope tool. These are all embedded logic analyzers that are very helpful for debugging. Debugging in Vivado software is even more convenient. As shown in the figure below, click Set Up Debug to directly enter the debug configuration interface.

.. image:: images/12_media/image21.png
      
The specific method for adding debug signals has been covered in the "PL 'Hello World' LED Experiment" in course_s1. Please refer to that section.

Bind the error signal to a PL-side LED in the XDC file.

.. image:: images/12_media/image22.png
      
Power-On Verification
----------------------

After generating the bit file, export it to Vitis and run Vitis, as shown in the figure below. Because Vitis cannot find the hardware information after the project is moved, a new hardware platform was created: top_hw_platform_1. The top_hw_platform_0 here was generated during debugging. You can directly delete it along with its files, and then rename the remaining top_hw_platform_1 to top_hw_platform_0. We created a helloworld program in Vitis. Although we are only testing PL-side reading of PS-side DDR, if the PS is not running, the DDR controller will not be working either. So this simple helloworld program is just to get the DDR controller up and running. Note that you must download from Vitis; if you download the bit file directly from Vivado, it will not run correctly. We configure the run options as shown in the figure below:

.. image:: images/12_media/image23.png
      
After clicking Run, the system will reset and download the FPGA bit file. Then return to the Vivado interface and click the auto-connect target in the Program and Debug panel as shown below:

.. image:: images/12_media/image24.png
      
After automatically connecting to the hardware, you can see the devices connected via JTAG. Among them is a hw_ila_1 device, which is our debug device. After selecting it, you can click the yellow triangle button above to capture waveforms. If some signals are not fully displayed, you can click the "+" button next to the waveform to add them.

.. image:: images/12_media/image25.png
      
After clicking to capture waveforms, the result is shown in the figure below. If the error signal remains low and the read/write states are changing, it indicates that DDR data read/write operations are normal. Users can view other signals here to observe the data written to DDR and the data read from DDR.

.. image:: images/12_media/image26.png
      
Chapter Summary
----------------

The ZYNQ system is significantly more complex than a standalone FPGA or standalone ARM, requiring a higher level of foundational knowledge from developers. This chapter covers the AXI protocol, ZYNQ interconnect resources, and debugging techniques for Vivado and Vitis. These are just the basics, and this chapter merely serves as an introduction. Readers are encouraged to practice extensively and master these skills through continuous practice.
