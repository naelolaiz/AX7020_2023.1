PS and PL Data Interaction via BRAM
====================================

**The Vivado project for this experiment is "bram_test".**

Sometimes the CPU needs to exchange small amounts of data with the PL, which can be achieved through the BRAM module, i.e., Block RAM. This chapter uses the Zynq GP
Master interface to read and write BRAM on the PL side, enabling interaction with the PL. In this experiment, a custom FPGA program is added and configured via the AXI4 bus to notify it when to read and write BRAM.

The following is the schematic of this experiment. The CPU reads BRAM data through the AXI BRAM Controller. The CPU only configures the registers of the custom PL BRAM Controller and does not read or write data through it.

.. image:: images/13_media/image1.png
      
Hardware Environment Setup
--------------------------

Based on the "ps_hello" project, save a copy of the project and configure ZYNQ interrupts.

1. First, add the AXI BRAM Controller module for PS-side BRAM control. Double-click to open the configuration, connect the AXI bus, which can be used to read and write the BRAM module. Set the AXI mode to AXI4 and the data width to 32 bits. The memory depth is not set here; it needs to be set in the Address Editor. Set the number of BRAM ports to 1 for connecting to PORTA of the dual-port RAM. Disable the ECC function.

.. image:: images/13_media/image2.png
      
Since the AXI4 bus uses byte addressing and the BRAM data width is also set to 32 bits, both having 32-bit data width, when mapping to BRAM addresses, 4-byte addressing is required, meaning the last two bits are removed. The figure below shows the mapping relationship between the BRAM controller and BRAM.

.. image:: images/13_media/image3.png
      
2. Add the BRAM module. The BRAM settings are as follows. There are two mode options: standalone mode, which allows free configuration of RAM data width and depth; and BRAM Controller mode, where the address lines and data ports default to 32 bits. Since this experiment uses a BRAM controller, BRAM Controller mode is selected. The memory type is set to dual-port RAM, with one port connected to the BRAM controller and the other to the PL RAM controller.

.. image:: images/13_media/image4.png
            
3. Add the custom PL RAM controller pl_ram_ctrl. Its function is simple: after the start signal is asserted, it begins reading BRAM data. The read data can be observed through the ILA logic analyzer. After the PL RAM controller finishes reading BRAM, it starts writing data to BRAM. After writing is complete, it asserts the intr signal (interrupt signal), and the CPU can then read the BRAM data. Connect the PL controller signals to PORTB of the BRAM. The custom IP is located in the ip_repo folder.

.. image:: images/13_media/image5.png
      
To add a custom IP to the IP library, click IP Catalog, then right-click on Vivado Repository and select Add Repository.

.. image:: images/13_media/image6.png
      
Navigate to the folder containing the custom IP and click Select.

.. image:: images/13_media/image7.png
      
A window pops up, select the IP and click OK.

.. image:: images/13_media/image8.png
      
You can see that the newly added IP now appears.

.. image:: images/13_media/image9.png
      
4. Connect the BRAM_PORTA of the AXI BRAM Controller to PORTA of the BRAM, and connect the BRAM_PORT of pl_bram_ctrl to PORTB of the BRAM. Connect the interrupt signal intr of the pl_bram_ctrl module to the interrupt port of ZYNQ. Then click Run Connection Automation.

.. image:: images/13_media/image10.png
      
5. In the Address Editor, select the BRAM addressing size. For example, setting a 4K space allows addressing a BRAM space with 1K depth.

.. image:: images/13_media/image11.png
      
Adding Logic Analyzer in Block Design
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

6. Here is another method for adding a logic analyzer. Select the BRAM_PORT bus and the intr interrupt, then right-click and select Debug.

.. image:: images/13_media/image12.png
      
7. You can see small debug icons appear on the bus. Click Run Connection Automation to auto-connect.

.. image:: images/13_media/image13.png
      
An ILA module is automatically added with one bus interface and one signal interface.

.. image:: images/13_media/image14.png
      
8. Save the design, then click Generate Bitstream to generate the bit file and export the Hardware information.

.. image:: images/13_media/image15.png
      
Vitis Program Development
-------------------------

1. The program design flow is as follows: Input the start address and length -> CPU writes BRAM data through the BRAM controller -> Notify the PL controller to read BRAM data -> After PL finishes reading internally, it writes data to the same location (initial data is provided by the CPU) -> After writing is complete, assert the write_end signal to trigger a GPIO interrupt -> Interrupt reads BRAM data and prints the results.

2. After entering Vitis, create a new project in Vitis. The program is already prepared. The program is relatively simple. First, set up the interrupt configuration.\ |image1|

3. In the While loop, the start address and length need to be input, then the bram_write function is called.

.. image:: images/13_media/image17.png
      
4. In the bram_read_write() function, data is first written through the BRAM controller with an initial value of TEST_START_VAL. Then the PL RAM controller parameters are configured, including length, start address, initial data, and the start signal. The function also checks whether the test length exceeds the BRAM controller address range. If it does, an error is reported and the address and length need to be re-entered.

.. image:: images/13_media/image18.png
            
5. In the interrupt service routine, the BRAM controller reads the BRAM data and prints it.

.. image:: images/13_media/image19.png
      
Experimental Results
--------------------

1. Open PuTTY.

.. image:: images/13_media/image20.png
      
2. Download the program through Run Configurations. Make sure to check Program FPGA, then click Run.

.. image:: images/13_media/image21.png
      
3. Open Hardware Manager, set the interrupt signal as the trigger signal, select rising edge trigger, and click the start button. You can see that hw_ila_1 changes to the Waiting for trigger state.

.. image:: images/13_media/image22.png
      
4. In the serial port software, enter the start address. Since the BRAM addressing is 1K, it can be set from 0 to 1023, and the length can be set from 1 to 1024. Note that the start address plus length should not exceed 1024, as this would exceed the addressing space.

.. image:: images/13_media/image23.png
      
5. The input data is in decimal. Press Enter after finishing the input.

.. image:: images/13_media/image24.png
      
6. Open the ILA logic analyzer. You can see that it has been triggered. First, the PL controller reads data from BRAM, followed by writing data. The red data represents the BRAM data read by the PL, which is exactly the data written by the CPU, starting from 12 with a total of 10 values. The yellow data represents the data written by the PL, starting from 1 with a total of 10 values, which matches the BRAM data read by the CPU shown above.

.. image:: images/13_media/image25.png
      
7. The interrupt signal status can also be observed.

.. image:: images/13_media/image26.png
      
8. If the range is exceeded, an error message is printed and valid information needs to be re-entered.

.. image:: images/13_media/image27.png
      
Chapter Summary
---------------

This concludes the experiment on low-bandwidth data interaction between PS and PL via BRAM. The two sides communicate data through the GP port, enabling small-batch data exchange.

Key concepts covered include the use of the logic analyzer, interrupt usage, and custom IP.

.. |image1| image:: images/13_media/image16.png
