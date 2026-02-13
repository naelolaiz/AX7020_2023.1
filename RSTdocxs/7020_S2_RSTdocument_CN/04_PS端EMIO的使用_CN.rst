Using PS EMIO
=============

**The experiment Vivado project is "ps_emio".**

Previously, we introduced the experiment of using the PS to light up LEDs. But what if you want to use the PS to control LEDs on the PL side? One approach is to control PL-side LEDs through EMIO, and the other is through the AXI
GPIO IP. This chapter describes how to use EMIO to control the on/off state of PL-side LEDs. It also introduces how to use EMIO to connect PL-side buttons to control PL-side LEDs.

Principle Introduction
----------------------

As previously introduced, the PS-side MIO structure is shown below. From the figure, we can see that BANK0 and BANK1 have 54 MIOs. BANK2 and BANK3 have 64 EMIOs. This chapter uses EMIO to control PL-side LEDs.

.. image:: images/04_media/image1.png
      
FPGA Engineer's Tasks
---------------------

The following is the content handled by the FPGA engineer.

Creating the Vivado Project
---------------------------

1. Based on the ps_hello project, save it as a new project named ps_emio, open the ZYNQ configuration, and check the GPIO EMIO option.

.. image:: images/04_media/image2.png
      
2. In the MIO configuration, set the EMIO bit width to 5 bits, because there are four LEDs on the PL side and one button on the PL side. After configuration, click OK.

.. image:: images/04_media/image3.png
      
3. Right-click on the additional GPIO_0 port and select Make External to export the port signal.

.. image:: images/04_media/image4.png
      
4. Rename the pin to emio.

.. image:: images/04_media/image5.png
      
The modification result. Save the design.

.. image:: images/04_media/image6.png
      
5. Right-click on xx.bd and select Generate Output Products to regenerate the output files.

.. image:: images/04_media/image7.png
      
6. After completion, the top-level file will be updated with new pins. The following step is to bind these pins.

.. image:: images/04_media/image8.png
      
XDC File to Constrain PL Pins
------------------------------

7. Create a new XDC file to bind PL-side pins.

.. image:: images/04_media/image9.png
      
Set the file name to emio.

.. image:: images/04_media/image10.png
      
8. Add the following content to emio.xdc. The port names must match the top-level file ports exactly.

::
   
 set_property IOSTANDARD LVCMOS33 [get_ports {emio_tri_io[*]}]
 #pl led
 set_property PACKAGE_PIN M14 [get_ports {emio_tri_io[0]}]
 set_property PACKAGE_PIN M15 [get_ports {emio_tri_io[1]}]
 set_property PACKAGE_PIN K16 [get_ports {emio_tri_io[2]}]
 set_property PACKAGE_PIN J16 [get_ports {emio_tri_io[3]}]
 #pl key
 set_property PACKAGE_PIN N15 [get_ports {emio_tri_io[4]}]

1. Generate the bit file.

.. image:: images/04_media/image11.png
      
10. Export the hardware. Since the PL is used, select "Include bitstream" and click "OK".

.. image:: images/04_media/image12.png
         
Software Engineer's Tasks
-------------------------

The following is the content handled by the software engineer.

Vitis Programming
-----------------

Using EMIO to Light PL-side LEDs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Open the Vitis software and create a new project named emio_led.

.. image:: images/04_media/image13.png
      
2. The code is similar to the PS-side MIO LED lighting operation. Since MIO numbers are 0~53, EMIO numbers start from 54. Only the following modifications are needed.

.. image:: images/04_media/image14.png
      
3. Download and configure.

.. image:: images/04_media/image15.png
      
You can now see the PL-side LEDs blinking.

Using EMIO to Implement PL-side Button Interrupt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Control the on/off state of PL-side LEDs through PL-side buttons.

1) Create a new project named emio_key with the hello world template, copy the example program, save and compile.

.. image:: images/04_media/image16.png
      
1. Port the MIO button interrupt program from the PS-side MIO usage chapter, and change the button number to 58 and the LED number to 54. Save and regenerate the elf.

.. image:: images/04_media/image17.png
      
2. Download the program.

.. image:: images/04_media/image18.png
      
1. Observe the experimental result. Press the PL-side button to control the on/off state of the PL-side LED.

..

   The AX7020/AX7010 development board silkscreen label is PL KEY1;

   PL-side LED location: The AX7020/AX7010 development board silkscreen label is PL LED1;

Programming the Flash
---------------------

Previously, we introduced how to generate a flash program without an FPGA loading file (for details, refer to the chapter "Experience ARM, Bare-metal Output Hello World"). This chapter generates the FPGA loading file, and here we demonstrate how to generate a flash program.

Same as before, click on system and right-click Build Project.

.. image:: images/04_media/image19.png
      
.. image:: images/04_media/image20.png
      
The software will automatically add three files: the first is the boot program fsbl.elf, the second is the FPGA bitstream, and the third is the application program xx.elf. The download method is the same as before and will not be repeated here.

Common Pin Binding Errors
-------------------------

1. In the block design, for example in the figure below, the GPIO module pin names are set to leds and keys. Many people naturally assume they can bind pins in the XDC file using these names.

.. image:: images/04_media/image21.png
            
If you open the top-level file, you will find that the pin names are different. Be sure to check carefully and use the pin names from the top-level file.

.. image:: images/04_media/image22.png
            
Otherwise, you will encounter the following unbound pin errors.

.. image:: images/04_media/image23.png
            
2. If you write the XDC file manually, pay close attention to spaces. This is also a very common error.

.. image:: images/04_media/image24.png
            
Chapter Summary
---------------

This chapter further explored the use of PS-side EMIO. Although EMIO is connected to PL-side pins, the usage in Vitis remains the same. From this example, we can also see that once there is a connection to the PL side, a bitstream needs to be generated, even though almost no logic is produced.
