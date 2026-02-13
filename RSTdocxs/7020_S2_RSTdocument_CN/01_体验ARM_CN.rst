Experience ARM, Bare-Metal "Hello World" Output
================================================

**The Vivado project for this experiment is "ps_hello".**

**Starting from this chapter, the work is carried out collaboratively by FPGA engineers and software developers.**

The previous experiments were all conducted on the PL side, and you can see that there is no difference from the ordinary FPGA development flow. The main advantage of ZYNQ is the rational combination of FPGA and ARM, which places higher demands on developers. Starting from this chapter, we will begin using ARM, which is what we call the PS. In this chapter, we will use a simple serial port print to experience the features of Vivado,
Vitis, and the PS side.

The previous experiments were all the work of FPGA engineers. Starting from this chapter, there is a division of labor. The FPGA engineer is responsible for building the Vivado project and providing the hardware to the software developer, who can then develop applications on this basis. A good division of labor also helps advance the project. If a software developer wants to do everything, it may take a lot of time and effort to learn FPGA knowledge. Transitioning from software thinking to hardware thinking is a rather painful process. If you are purely learning and have the time, that is a different story. Having professionals do professional work is a very good choice.

Hardware Introduction
---------------------

From the schematic, we can see that the ZYNQ chip is divided into PL and PS. The IO allocation on the PS side is relatively fixed and cannot be arbitrarily assigned, and there is no need to assign pins in the Vivado software. Although this experiment only uses the PS, a Vivado project still needs to be created to configure the PS pins. Although the ARM on the PS side is a hard core, it must also be added to the project in ZYNQ before it can be used. The previous chapters introduced code-based projects; this chapter introduces the graphical method of creating projects in ZYNQ.

FPGA Engineer Work Content
---------------------------

The following describes the content that the FPGA engineer is responsible for.

Vivado Project Creation
-----------------------

1) Create a project named "ps_hello". The creation process will not be repeated here; refer to the "PL 'Hello
   World' LED Experiment".

2) Click "Create Block Design" to create a Block design, which is a graphical design

.. image:: images/01_media/image1.png
      
3) The "Design name" is not modified here and remains the default "design_1". This can be modified as needed, but the name should be as short as possible, otherwise there will be issues when compiling under Windows.

.. image:: images/01_media/image2.png
      
4) Click the "Add IP" shortcut icon

.. image:: images/01_media/image3.png
      
5) Search for "zynq" and double-click "ZYNQ7 Processing System" in the search results list

.. image:: images/01_media/image4.png
      
6) Double-click "processing_system7_0" in the Block diagram to configure the relevant parameters

.. image:: images/01_media/image5.png
      
7) The first interface that appears is the architecture diagram of the ZYNQ hard core. You can clearly see its structure. You can refer to the ug585 document, which contains a detailed introduction to ZYNQ. The green parts in the diagram are configurable modules. You can click to enter the corresponding editing interface, and you can also enter the editing interface from the left window. The functions of each window are introduced below.

.. image:: images/01_media/image6.png
      
8) Next is the PS-PL
Configuration interface. This interface is mainly for configuring the interface between PS and PL, primarily AXI interfaces. These interfaces can extend AXI peripheral devices on the PL side, so if PL needs to exchange data with PS, it must follow the AXI bus protocol. Xilinx provides a large number of AXI interface IP
cores. Keep the defaults here; they will be configured in later chapters. This chapter does not interact with the PL side, so keep the defaults.

.. image:: images/01_media/image7.png
      
9) Then we enter the PS peripheral configuration stage. When first encountering ZYNQ, you may be very confused seeing the dense peripherals, not knowing where to start. Here is an explanation: many of the PS-side peripherals in ZYNQ are multiplexed. The same pin numbers can be configured with different functions. For example, pins 16-27 in the figure below can be configured as Enet0, or as SD0, SD1, but can only be configured as one peripheral. For instance, if configured as Enet0, you cannot select SD0 or SD1.
As for how to choose, it is determined by the schematic and PCB. You can select by checking the schematic or user manual.

.. image:: images/01_media/image8.png
      
.. image:: images/01_media/image9.png
      
PS-Side Peripheral Schematic

PS-Side Peripheral Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1)  From the schematic, we can find that the serial port is connected to MIO48-MIO49 of the PS, so enable UART1 (MIO48 MIO49) in the "Peripheral I/O Pins" option. The PS-side MIO is divided into two Banks: Bank 0, which corresponds to BANK500 in the schematic, with voltage selected as "LVCMOS 3.3V"; Bank 1, which corresponds to BANK501 in the schematic, with voltage selected as "LVCMOS 1.8V".\ **If the Bank1 voltage standard is not configured, the serial port may not be able to receive data**\ .

.. image:: images/01_media/image10.png
      
1)  Configure QSPI. QSPI can serve as the boot storage device for ZYNQ. ZYNQ can load ARM and FPGA by reading the boot files stored in QSPI. From the schematic, we select Quad SPI Flash as Single SS 4bit IO

.. image:: images/01_media/image11.png
      
12) Configure Ethernet. The PS side has an Ethernet interface. According to the schematic, select Ethernet 0 to MIO16-MIO27

.. image:: images/01_media/image12.png
      
MDIO is the Ethernet PHY register configuration interface. Select MDIO and configure it to MIO52-MIO53

.. image:: images/01_media/image13.png
      
13) Configure USB0 to MIO28-MIO39

.. image:: images/01_media/image14.png
      
14) In addition to QSPI boot for ZYNQ, there is also SD card boot mode. Select SD 0, configure to MIO40-MIO45, and select Card Detection MIO47 for detecting SD card insertion.

.. image:: images/01_media/image15.png
      
15) Enable GPIO MIO so that the PS can control the remaining unassigned MIOs as GPIO

.. image:: images/01_media/image16.png
      
Select MIO46 in GPIO MIO as the USB PHY reset

.. image:: images/01_media/image17.png
      
At this point, the peripheral configuration is complete.

MIO Configuration
~~~~~~~~~~~~~~~~~

Change the voltage standard of Enet0 to HSTL 1.8V and the Speed to fast. These parameters are very important; if not modified, the network may not work. Keep other parts at their defaults.

.. image:: images/01_media/image18.png
      
Clock Configuration
~~~~~~~~~~~~~~~~~~~

16) In the "Clock Configuration" tab, we can configure the PS clock input frequency. The default here is 33.333333, which matches the board, so no modification is needed. The CPU frequency defaults to 666.666666MHz, which we also leave unchanged. The PS can also provide 4 clock outputs to the PL side with configurable frequencies, but we do not need them here, so keep the defaults. The clocks for PS-side peripherals can also be configured, but we keep the defaults here.

.. image:: images/01_media/image19.png
      
DDR3 Configuration
~~~~~~~~~~~~~~~~~~

17) In the "DDR Configuration" tab, you can configure the PS-side DDR parameters. For AX7010, configure the DDR3 model as "MT41J128M16 HA-125"; for AX7020, configure the DDR3 model as "MT41J256M16 RE-125".\ **The DDR3 model here is not the actual DDR3 model on the board, but the model with the closest parameters**\ . For "Effective DRAM Bus Width", select "32 Bit"

.. image:: images/01_media/image20.png
      
AX7010 DDR3 Configuration

.. image:: images/01_media/image21.png
      
AX7020 DDR3 Configuration

Keep other parts at their defaults and click OK. At this point, the ZYNQ core configuration is complete.

1)  Click "Run Block Automation". The Vivado software will automatically complete some port export work

.. image:: images/01_media/image22.png
      
19) Click "OK" with the defaults

.. image:: images/01_media/image23.png
      
20) After clicking "OK", we can see that the PS side exports some pins, including DDR and FIXED_IO. DDR is the DDR3 interface signal, and FIXED_IO contains some fixed interfaces on the PS side, such as the input clock, PS-side reset signal, MIO, etc.

.. image:: images/01_media/image24.png
      
21) Connect FCLK_CLK0 to M_AXI_GP0_ACLK, and press Ctrl+S to save the design

.. image:: images/01_media/image25.png
      
*Note: DDR and FIXED_IO are PS-side pins. PS_PORB is the PS-side power-on reset signal and cannot be used for PL-side reset. Do not bind the PL-side reset to this pin number. Remember this!!!*

.. image:: images/01_media/image26.png
      
22) Select the Block design, right-click "Create HDL Wrapper..." to create a Verilog or VHDL file that generates the HDL top-level file for the block design.

.. image:: images/01_media/image27.png
      
23) Keep the default options and click "OK"

.. image:: images/01_media/image28.png
      
24) Expanding the design, you can see that the PS is used as a regular IP.

.. image:: images/01_media/image29.png
      
25) Select the block design, right-click "Generate Output Products". This step generates the block output files, including IP, instantiation templates, RTL source files, XDC constraints, third-party synthesis source files, etc., for subsequent operations.

.. image:: images/01_media/image30.png
      
26) Click "Generate"

.. image:: images/01_media/image31.png
      
27) It is not that PS-side pins do not need to be bound; rather, the IP-generated output files already contain the XDC file for PS-side pin assignments. In IP Sources, Block Designs design\_

28) 1Synthesis, you can see the processor's XDC file, which binds the PS-side IO. Therefore, there is no need to create a new XDC to bind these pins.

.. image:: images/01_media/image32.png
      
29) In the menu bar, go to "File -> Export -> Export Hardware..." to export the hardware information, which includes the PS-side configuration information.

.. image:: images/01_media/image33.png
      
30) In the dialog box that appears, click "OK". Since the experiment only uses the PS serial port and does not require PL participation, "Include bitstream" is not enabled here. The export path can be freely chosen. In this experiment, it is saved in a newly created folder called vitis under the project path. This folder can be created at any suitable location according to your needs and does not have to be under the Vivado project. The Vivado and Vitis software are independent.

.. image:: images/01_media/image34.png
      
.. image:: images/01_media/image35.png
      
At this point, you can see the xsa file in the newly created vitis folder. This file contains the Vivado hardware design information for use by software developers.

.. image:: images/01_media/image36.png
      
At this point, the FPGA engineer's work is complete for now.

Software Engineer Work Content
-------------------------------

The following is the content that the software engineer is responsible for.

Vitis Debugging
---------------

Creating an Application Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1) Vitis is a standalone software. We can open Vitis through Tools -> Launch Vitis

.. image:: images/01_media/image37.png
      
You can also double-click the Vitis software to open it

.. image:: images/01_media/image38.png
         
Select the previously created folder and click "Launch"

.. image:: images/01_media/image39.png
         
2) After launching Vitis, the interface is as follows. Click "Create Application Project". This option generates an APP project and a Platform project. The Platform project is similar to the hardware platform in previous versions, containing hardware support files and BSP.

.. image:: images/01_media/image40.png
         
3) Click Next

.. image:: images/01_media/image41.png
         
4) Click "Create a new platform hardware (XSA)". The software already provides hardware platforms for some boards, but for our own hardware platform, we can select "browse"

.. image:: images/01_media/image42.png
         
5) Select the previously generated xsa file and click Open

.. image:: images/01_media/image43.png
         
6) The "Generate boot components" option at the bottom, if checked, will cause the software to automatically generate the fsbl project. We generally keep it checked by default. Click Next

.. image:: images/01_media/image44.png
         
7) Enter "hello" for the project name, or fill in as needed. The CPU defaults to ps7_cortexa9_0, OS selects standalone. Click Next

.. image:: images/01_media/image45.png
         
.. image:: images/01_media/image46.png
         
8) Select Hello World as the template and click Finish

.. image:: images/01_media/image47.png
         
9) After completion, you can see that two projects have been generated: one is the hardware platform project, i.e., the Platform project mentioned earlier, and the other is the APP project

.. image:: images/01_media/image48.png
         
10) Expanding the Platform project, you can see that it contains the BSP project and the zynq_fsbl project (this project is the result of selecting Generate boot components). Double-click platform.spr to see the BSP project generated by the Platform, where you can configure the BSP. Software developers are familiar with BSP, which stands for Board Support Package, containing the driver files needed for development and used for application development. You can see that there are multiple BSPs under Platform, which is different from previous versions of Vitis. Among them, zynq_fsbl is the BSP for fsbl, and standalone on ps7_cortexa9_0 is the BSP for the APP project. You can also add BSPs in the Platform, which will be discussed in later examples.

.. image:: images/01_media/image49.png
         
1)  Clicking on the BSP, you can see the peripheral drivers included in the project. Among them, Documentation is the driver documentation provided by Xilinx, and Import Examples are example projects provided by Xilinx to accelerate learning.

.. image:: images/01_media/image50.png
      
12) Select the APP project, right-click Build Project, or click the "hammer" button in the menu bar to compile the project

.. image:: images/01_media/image51.png
      
13) You can see the compilation process in the Console

.. image:: images/01_media/image52.png
      
Compilation is complete, and the elf file is generated

.. image:: images/01_media/image53.png
      
14) Connect the JTAG cable to the development board and the UART USB cable to the PC

15) Use PuTTY software as the serial terminal debugging tool. PuTTY is a small portable software that does not require installation

.. image:: images/01_media/image54.png
      
16) Select Serial, fill in COM3 for Serial line, fill in 115200 for Speed. The COM3 serial port number should be filled in according to what is displayed in the Device Manager. Click "Open"

.. image:: images/01_media/image55.png
      
17) Before powering on, it is best to set the boot mode of the development board to JTAG mode

.. image:: images/01_media/image56.png
      
18) Power on the development board and prepare to run the program. The development board comes with a program from the factory. Here, you can select JTAG mode for the boot mode and then power on again. Select "hello", right-click, and you can see many options. This experiment uses "Run as", which runs the program. Under "Run as", there are many options. Select the first one, "Launch on Hardware (Single Application Debug)", to use system debugging and run the program directly.

.. image:: images/01_media/image57.png
      
19) At this point, observe the PuTTY software, and you can see the output "Hello World"

.. image:: images/01_media/image58.png
      
20) To ensure reliable system debugging, it is best to right-click "Run As -> Run Configuration..."

.. image:: images/01_media/image59.png
      
21) We can look at the configuration inside. Among them, "Reset entire system" is selected by default, which is different from previous versions of Vitis. If there is also a PL design in the system, you must also select "Program FPGA".

.. image:: images/01_media/image60.png
      
22) In addition to "Run As", you can also use "Debug As", which allows setting breakpoints and step-by-step execution

.. image:: images/01_media/image61.png
      
23) Enter Debug mode

.. image:: images/01_media/image62.png
      
24) Like other C language development IDEs, you can step through execution, set breakpoints, etc.

.. image:: images/01_media/image63.png
      
25) The IDE mode can be switched in the upper right corner

.. image:: images/01_media/image64.png
      
Program Flashing
----------------

An ordinary FPGA can generally boot from flash or be passively loaded. ZYNQ booting is led by ARM, including loading the FPGA program. ZYNQ booting generally requires at least two steps, which are also described in UG585:

Stage 0
:After power-on reset or warm reset, the processor first executes the code in BootRom. This is the initial boot setup. BootRom contains a piece of code that cannot be modified by the user. Of course, it is only executed in non-JTAG mode. The code includes the most basic drivers for NAND, NOR, Quad-SPI, SD, and PCAP. Another very important function is to move the stage
1 code to OCM, which is the FSBL code (First Stage Boot
Loader), with a space limit of 192KB.

Stage 1:
Next comes the most important step. After BootRom moves the FSBL to OCM, the processor begins executing the FSBL code. The FSBL mainly serves the following purposes:

-  Initialize PS-side configuration. These configurations are the ZYNQ core configurations set in the Vivado project, including initializing DDR, MIO, and SLCR registers. This mainly involves executing ps7_init.c and ps7_init.h. The execution effect of ps7_init.tcl is the same as ps7_init.c.

-  If there is a PL-side program, load the PL-side bitstream

-  Load the second stage bootloader or bare-metal application into DDR memory

-  Hand off to the second stage bootloader or bare-metal application

.. image:: images/01_media/image65.png
      
Stage 2: Second stage
bootloader is optional and is generally used when running an operating system, such as u-boot for Linux. It will not be introduced here. Later, we will use the PetaLinux tool to build a Linux system.

Generating FSBL
~~~~~~~~~~~~~~~~

FSBL is a second-level boot loader that completes MIO allocation, clock, PLL, DDR controller initialization, SD, QSPI controller initialization, searches for the bitstream through the boot mode to configure the FPGA, then searches for the user program to load into DDR, and finally hands off to the application for execution. For details, please refer to the ug821 document.

1) Since the "Generate boot components" option was selected during creation, the Platform already has the fsbl project imported and has generated the corresponding elf file.

.. image:: images/01_media/image66.png
      
2) Add the debug macro definition FSBL_DEBUG_INFO, which allows FSBL status information to be output during startup, aiding in debugging, but it will increase the boot time. Save the file. You can see that fsbl contains many peripheral files, including ps7_init.c, nand, nor, qspi, sd, etc. In fsbl's main.c, the first function to run is ps7_init. As for the subsequent work, you can read the code carefully. Of course, this fsbl template can also be modified according to your needs.

.. image:: images/01_media/image67.png
      
3) Rebuild the Project

.. image:: images/01_media/image68.png
      
4) Next, we can click on the system of the APP project, right-click and select Build Project

.. image:: images/01_media/image69.png
      
5) At this point, an additional Debug folder will appear, and the corresponding BOOT.BIN will be generated

.. image:: images/01_media/image70.png
      
6) Another method is to click on the system of the APP project, right-click and select Create Boot Image. In the pop-up window, you can see the path of the generated BIF file. The BIF file is the configuration file for generating the BOOT file. There is also the path of the generated BOOT.bin file. The BOOT.bin file is the boot file we need, which can be placed on an SD card for booting or written to QSPI Flash.

.. image:: images/01_media/image71.png
      
.. image:: images/01_media/image72.png
      
7) In the Boot image partitions list are the files to be combined. The first file must be the bootloader file, which is the fsbl.elf file generated above. The second file is the FPGA configuration file bitstream. In this experiment, since there is no FPGA bitstream, it does not need to be added. The third is the application, which in this experiment is hello.elf. Since there is no bitstream, only the bootloader and application are added in this experiment. Click Create Image to generate.

.. image:: images/01_media/image73.png
      
8) The BOOT.bin file can be found in the generated directory

.. image:: images/01_media/image74.png
      
SD Card Boot Test
~~~~~~~~~~~~~~~~~~

1) Format the SD card. It can only be formatted as FAT32; other formats cannot boot

.. image:: images/01_media/image75.png
      
2) Place the BOOT.bin file in the root directory

.. image:: images/01_media/image76.png
      
3) Insert the SD card into the SD card slot of the development board

4) Set the boot mode to SD card boot

.. image:: images/01_media/image56.png
      
5) Open PuTTY software, power on and boot. You can see the print information. The red box shows the FSBL boot information, and the yellow arrow shows the executed application helloworld

.. image:: images/01_media/image77.png
      
QSPI Boot Test
~~~~~~~~~~~~~~~

1) In the Vitis menu, go to Xilinx -> Program Flash

.. image:: images/01_media/image78.png
      
1) For Hardware Platform, select the latest one. For Image File, select the BOOT.bin to be programmed. For FSBL file, select fsbl.elf. Select Verify after flash to verify the flash after programming is complete.

.. image:: images/01_media/image79.png
      
2) Click Program and wait for programming to complete

.. image:: images/01_media/image80.png
      
3) Set the boot mode to QSPI, and boot again. You can see the same boot result as SD in PuTTY.

.. image:: images/01_media/image81.png
      
.. image:: images/01_media/image82.png
      
Programming QSPI in Vivado 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1) In HARDWARE MANAGER, select the device, right-click Add Configuration Memory Device

.. image:: images/01_media/image83.png
      
2) Select the manufacturer Winbond, select type qspi, select width x4-single. At this point, w25q128 appears. Select the model shown in the red box. The development board uses w25q256, but this does not affect programming.

.. image:: images/01_media/image84.png
      
3) Right-click to select the programming file

.. image:: images/01_media/image85.png
      
4) Select the file to be programmed and the fsbl file, and you can start programming. If the boot mode is not JTAG during programming, the software will give a warning. Therefore, it is recommended to set the boot mode to JTAG when programming QSPI

.. image:: images/01_media/image86.png
      
Using a Batch File to Quickly Program QSPI
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1) Create a new text file named program_qspi.txt, change the extension to bat, and fill in the following content. Among them, set XIL_CSE_ZYNQ_DISPLAY_UBOOT_MESSAGES=1 sets the display of uboot print information during the programming process.

..

   F:\\Xilinx_Vitis\\Vitis\\2023.1\\bin\\program_flash
   is our tool path, modify appropriately according to the installation path. -f
   is the file to be programmed, -fsbl is the fsbl file used for programming, -verify is the verification option.

::

 call F:\Xilinx_Vitis\Vitis\2023.1\bin\program_flash -f BOOT.bin    -offset 0 -flash_type qspi-x4-single  -fsbl fsbl.elf -verify
 pause

1) Place the BOOT.bin to be programmed, the fsbl, and the bat file together

.. image:: images/01_media/image87.png
      
2) After connecting the JTAG cable and powering on, double-click the bat file to program the flash.

.. image:: images/01_media/image88.png
      
Frequently Asked Questions
--------------------------

Flashing with Only PL-Side Logic
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Many people ask: if there is only PL-side logic and the PS side is not needed, how do you flash the program? Flashing an FPGA without ARM is not a problem, but for ZYNQ, PS-side cooperation is required to flash the program. So how do you flash the program for the previous "PL 'Hello World' LED Experiment"?

1. Based on this chapter, add the ZYNQ core on the PS side and configure it. The simplest method is to add the LED experiment's Verilog source files to this chapter's project, instantiate them to form a system, and generate the bitstream.

.. image:: images/01_media/image89.png
      
.. image:: images/01_media/image90.png
      
2. After generating the bitstream, export the hardware and select include bitstream

.. image:: images/01_media/image91.png
         
3. When generating BOOT.BIN, you still need an app project hello, solely for generating BOOT.BIN. By default, right-click Build Project on the system to generate BOOT.BIN that includes the bitstream.

.. image:: images/01_media/image92.png
      
Open the Create Boot Image interface and you can see that the file order in Boot Image Partitions is fsbl, bitstream, app. Note that the order must not be reversed. The BOOT.BIN generated this way can be tested for booting using the methods described earlier

.. image:: images/01_media/image93.png
      
In the course_s2 folder, we provide a project named led_qspi_sd for your reference.

Tips and Tricks
---------------

When frequently modifying source files and compiling, it is best to select the APP project for Build Project. In this case, only the elf file will be generated.

.. image:: images/01_media/image94.png
      
If you want to generate a BOOT.BIN file, you can select the system for compilation. In this case, both the elf and BOOT.BIN will be generated. The author suffered from this when first using the tool, selecting system every time for compilation, which resulted in waiting for BOOT.BIN generation every time, wasting time. Please take note of this.

.. image:: images/01_media/image95.png
      
Chapter Summary
---------------

This chapter, from the perspectives of both the FPGA engineer and the software engineer, introduces the classic ZYNQ development flow. The FPGA engineer's main work is to build the hardware platform and provide the hardware description file xsa to the software engineer, who then develops applications on this basis. This chapter is a simple example illustrating the collaboration between FPGA and software engineers. Later chapters will involve joint debugging between PS and PL, which is more complex and is the core part of ZYNQ development.

This chapter also introduces FSBL, boot file creation, SD card boot method, QSPI programming and boot method, and Vivado BOOT.BIN programming method. This chapter does not include an FPGA bitstream file; later applications will introduce how to create BOOT.BIN with an FPGA bitstream file.

Subsequent projects will be based on the configuration in this chapter, and the basic ZYNQ configuration will not be introduced again.

A journey of a thousand miles begins with a single step. After studying this chapter, you should have a basic understanding of ZYNQ development. Whether a tall building is stable depends on whether the foundation is solid. Although this chapter is relatively simple, there are many knowledge points for you to slowly digest. Keep going!!!
