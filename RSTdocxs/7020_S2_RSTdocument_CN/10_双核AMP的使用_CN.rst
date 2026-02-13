Using Dual-Core AMP
===================

**The Vivado project for this experiment is "dualcore_amp".**

The previous examples all used a single-core CPU. In some cases, such as multitasking and parallel processing, a dual-core CPU is needed. This chapter provides a brief introduction to using dual cores and implements the following functions:

1. CPU0 implements PS-side button interrupt to control PS-side LED on/off, and sends a software interrupt to CPU1, causing CPU1 to print a string from CPU0's memory space

2. CPU1 implements PL-side button interrupt to control PL-side LED on/off, and sends a software interrupt to CPU0, causing CPU0 to print a string from CPU1's memory space

3. Memory space partitioning and shared memory space usage

4. FSBL boot from Flash

The reference document is XAPP1079, bare-metal dual-core application. Terminology:

AMP:
Asymmetric Multi-Processing, where each CPU core runs an independent operating system or an independent instance of the same operating system

SMP:
Symmetric Multi-Processing, where a single operating system instance can manage all CPU cores, and applications are not bound to any specific core

BMP:
Bound Multi-Processing, where a single operating system instance can manage all CPU cores simultaneously, but each application is locked to a specific designated core.

Hardware Environment Setup
--------------------------

1. This experiment is based on the "ps_hello" example, with PL-side GPIO added. Add axi_gpio_0 configured as output, with a bit width of 1, connected to the PL-side LED.

.. image:: images/10_media/image1.png
      
2. Add axi_gpio_1, connected to the PL-side button, configured as input with a bit width of 1, and enable the interrupt

.. image:: images/10_media/image2.png
      
3. The result after connection is shown below. Connect the interrupt output of axi_gpio_1 to the CPU's IRQ_F2P port

.. image:: images/10_media/image3.png
      
4. Enable GPIO EMIO configuration (The AX7Z035 and AX7Z100 development boards are special cases as they do not have PS-side buttons and LEDs, so EMIO must be configured to control PL-side buttons and LEDs. Other boards do not need to enable this option)

.. image:: images/10_media/image4.png
      
Set the EMIO width to 2, one for controlling the button and one for controlling the LED

.. image:: images/10_media/image5.png
      
Export it and modify the name

.. image:: images/10_media/image6.png
      
5. Generate Outputs

.. image:: images/10_media/image7.png
      
6. Assign the button and LED pin constraints, then generate the bitstream

.. image:: images/10_media/image8.png
      
Vitis Program Development
-------------------------

CPU0 Vitis Project Creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Create a new project. Note that the CPU should be selected as ps7_cortexa9_0, which is CPU0

.. image:: images/10_media/image9.png
      
2. The code has been prepared for you: cpu0_app.c and share.h. share.h contains the shared memory structure, which will be discussed later.

.. image:: images/10_media/image10.png
      
3. Set the CPU0 access space in lscript.ld. For example, if DDR3 is 1 GByte, set the CPU0 space to half. Of course, this can be modified as needed.

.. image:: images/10_media/image11.png
      
CPU1 Vitis Project Creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Before creating the CPU1 APP project, we can first create a Domain based on CPU1, which is the so-called BSP. Click "+" in platform.spr

.. image:: images/10_media/image12.png
      
2. Enter the name, select ps7_cortexa9_1 as the Processor (i.e., CPU1), and click OK

.. image:: images/10_media/image13.png
      
3. When creating the CPU1 project, select the newly created Domain

.. image:: images/10_media/image14.png
      
4. The code has also been prepared: cpu1_app.c and share.h

.. image:: images/10_media/image15.png
      
5. Set the CPU1 memory space. Make sure it does not overlap with CPU0. The last 256 bytes of space are reserved for shared memory

.. image:: images/10_media/image16.png
      
6. Click on the CPU1 BSP settings

.. image:: images/10_media/image17.png
      
7. In the CPU1 BSP settings interface, add -DUSE_AMP=1 in the extra_compile_flags to enable dual-core operation

.. image:: images/10_media/image18.png
      
Build the APP projects for both CPU0 and CPU1

CPU0 Program Description
~~~~~~~~~~~~~~~~~~~~~~~~

1. In the cpu0_app.c file, a character array Cpu0_Data is defined and stored in the CPU0 access space. The pointer Cpu1Data is used to point to the character array in CPU1.

.. image:: images/10_media/image19.png
      
2. In the program, CPU0 needs to wake up CPU1. The relevant explanation can be found in the UG585 document. The first step is to write the CPU1 access memory base address to the 0xFFFFFFF0 address, which is 0x20000000 in this experiment. The second step is to wake up CPU1 using the SEV instruction and jump to the corresponding program.

.. image:: images/10_media/image20.png
      
CPU1STARTMEM is the CPU1 base address set in lscript.ld

.. image:: images/10_media/image21.png
      
3. In the main function, the Xil_SetTlbAttributes function is first used to disable Cache for OCM access. The author believes that the 0xFFFFFFF0 address is within the OCM address range, and disabling Cache can reduce the consistency issues of maintaining OCM access between two CPUs. The author has tested that without this function, CPU1 does not work after Flash boot. Refer to the XAPP1079 document for details.

.. image:: images/10_media/image22.png
      
4. Next is the interrupt initialization and PS GPIO setup. Software interrupts use ID numbers 1 and 2.

.. image:: images/10_media/image23.png
      
Connect interrupt number 1 to the software interrupt service function.

.. image:: images/10_media/image24.png
      
5. In the while loop, the address and length of the character array are assigned to the shared structure. Here we should mention the shared memory structure: in share.h, the structure ShareMem is defined for passing information through shared memory.

.. image:: images/10_media/image25.png
      
.. image:: images/10_media/image26.png
      
Both cores agree on the shared address, so parameters can be passed.

.. image:: images/10_media/image27.png
      
The software interrupt with interrupt number 2 is triggered through the XScuGic_SoftwareIntr function. The third parameter of this function is the CPU number. Note that the CPU number is not simply 0, 1, 2, etc. Instead, each bit represents a CPU number. Refer to the ICDIPTR explanation in the UG585 register table for mpcore: 0bxxxxxxx1 targets CPU0, 0bxxxxxx1x targets CPU1. Therefore, in this program, the CPU1 number value is set to 0x2

.. image:: images/10_media/image28.png
      
6. In the while loop, when a software interrupt from CPU1 is detected, the string from CPU1's memory space is printed.

.. image:: images/10_media/image29.png
      
CPU1 Program Description
~~~~~~~~~~~~~~~~~~~~~~~~

1. The CPU1 program also has a character array. Cpu0Data points to the string address in CPU0's memory space.

.. image:: images/10_media/image30.png
      
2. In the main function, the OCM Cache is also disabled first

.. image:: images/10_media/image31.png
      
3. In the PLGpioSetup function, the button interrupt number needs to be bound to CPU1. The rest is similar to CPU0 and will not be repeated here.

.. image:: images/10_media/image32.png
      
On-Board Verification
---------------------

1. When downloading, make sure to enter the Run Configurations settings

.. image:: images/10_media/image33.png
      
2. Double-click Single Application Debug

.. image:: images/10_media/image34.png
      
3. Check the CPU1 checkbox, leave the rest as default, and click Run

.. image:: images/10_media/image35.png
      
4. Open the serial port software and test CPU0. Press the button to turn on the LED, indicating that CPU0 is running. At the same time, CPU1 receives the software interrupt set by CPU0 and prints the information. (For the AX7015 board: PS_KEY and PS_LED; for the AX7021 board: KEY1 and LED1; for the AX7020/AX7010 board: PS KEY1 and PS LED1; for the AX7Z035/AX7Z100 board: KEY1 and LED1)

.. image:: images/10_media/image36.png
      
5. Test CPU1. Press the button to turn on the PL-side LED, indicating that CPU1 is running. At the same time, CPU0 receives the software interrupt set by CPU1 and prints the information. (For the AX7015 board: PL_KEY and PL_LED4; for the AX7021 board: KEY2 and LED2; for the AX7020/AX7010 board: PL KEY1 and PL LED1; for the AX7Z035/AX7Z100 board: KEY2 and LED2)

.. image:: images/10_media/image37.png
      
QSPI Flash Boot
----------------

The method of generating BOOT.BIN is different from the previous Build Project generation; we need to configure it. Right-click on the CPU0 system and select Create Boot Image

.. image:: images/10_media/image38.png
      
Click Add to add the CPU1 elf file

.. image:: images/10_media/image39.png
      
Select datafile for the Partition type

.. image:: images/10_media/image40.png
      
The result after adding is shown below. Click Create Image

.. image:: images/10_media/image41.png
      
Chapter Summary
---------------

This chapter provided a brief introduction to using dual cores in bare-metal mode, including interrupt usage and inter-core communication. In this experiment, the length member of the shared memory structure was not used. You can try copying data between the two cores based on the length and address.

Note that the AX7010 has 512 MB of DDR3 memory, while the AX7020/AX7015/AX7021 has 1 GB of DDR3 memory. Therefore, pay attention to the differences when setting up the dual-core memory space. Refer to the provided examples for guidance.
