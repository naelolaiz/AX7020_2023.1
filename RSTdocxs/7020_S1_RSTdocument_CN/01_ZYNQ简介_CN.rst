ZYNQ Introduction
=================

The highlight of the Zynq series is that the FPGA contains a complete ARM processing subsystem (PS). Every Zynq series processor includes a Cortex-A9 processor, and the entire system is built around the processor. The processing subsystem integrates a memory controller and a large number of peripherals,
allowing the Cortex-A9 core in the Zynq-7000 to operate completely independently of the programmable logic (PL). This means that if the programmable logic section (PL) is not being used, the ARM processor subsystem can still work independently. This is fundamentally different from previous FPGAs, as it is processor-centric.

Zynq consists of two major functional blocks: the PS section and the PL section. Simply put, these are the ARM SoC section and the FPGA section. The PS integrates two ARM
Cortex™-A9 processors, AMBA® interconnect, internal memory, external memory interfaces, and peripherals. These peripherals mainly include USB bus interfaces, Ethernet interfaces, SD/SDIO interfaces, I2C bus interfaces, CAN bus interfaces, UART interfaces, GPIO, etc.

.. image:: images/01_media/image1.png
      
Overall Block Diagram of the ZYNQ Chip

PS: Processing System, the ARM SoC portion that is independent of the FPGA.

PL: Programmable Logic, the FPGA portion.

*The PL section is the same as the 7 series. The corresponding 7 series products can be found in the DS190 document.*

.. image:: images/01_media/image2.png
      
PS and PL Interconnection Technology
-------------------------------------

As the first product to tightly integrate a high-performance ARM
Cortex-A9 series processor with a high-performance FPGA on a single chip, ZYNQ requires the design of high-speed communication and data exchange paths between the ARM processor and the FPGA to leverage the performance advantages of both the ARM processor and the FPGA. Therefore, designing an efficient data exchange path between PL and PS is of paramount importance in ZYNQ chip design and is one of the key factors in the success of a product design. In this section, we will mainly introduce the connection between PS and PL, allowing users to understand the interconnection technology between PS and PL.

In practice, we often do not need to do much work on the connection itself. After we add an IP core, the system will automatically use the AXI interface to connect our IP core to the processor, and we only need to make minor adjustments.

AXI stands for Advanced eXtensible
Interface. It is an interface protocol introduced by Xilinx starting from the 6 series FPGAs, which mainly describes the data transfer method between master and slave devices. It continues to be used in ZYNQ, with the version being AXI4, so we often see AXI4.0. All internal devices in ZYNQ have AXI interfaces. In fact, AXI is part of the AMBA (Advanced Microcontroller Bus
Architecture) specification proposed by ARM. It is a high-performance, high-bandwidth, low-latency on-chip bus, also used to replace the previous AHB and APB buses. The first version of AXI (AXI3) was included in AMBA 3.0 released in 2003, and the second version of AXI (AXI4) was included in AMBA 4.0 released in 2010.

The AXI protocol mainly describes the data transfer method between master and slave devices. The master and slave devices establish a connection through handshake signals. When the slave device is ready to receive data, it asserts the READY signal. When the master device's data is ready, it asserts and maintains the VALID signal, indicating that the data is valid. Data transfer only begins when both the VALID and READY signals are active. When both signals remain active, the master device continues to transfer the next data. The master device can deassert the VALID signal, or the slave device can deassert the READY signal to terminate the transfer. The AXI protocol is shown in the figure: at T2, the slave device's READY signal becomes active; at T3, the master device's VALID signal becomes active, and data transfer begins.

.. image:: images/01_media/image3.png
      
AXI Handshake Timing Diagram

In ZYNQ, three types of buses are supported: AXI-Lite, AXI4, and AXI-Stream. Through Table 5-1, we can see the characteristics of these three AXI interfaces.

+----------------------+----------------------+------------------------+
| Interface Protocol   | Characteristics      | Application Scenarios  |
+======================+======================+========================+
| AXI4-Lite            | Address/single data  | Low-speed peripherals  |
|                      | transfer             | or control             |
+----------------------+----------------------+------------------------+
| AXI4                 | Address/burst data   | Bulk address-based     |
|                      | transfer             | transfers              |
+----------------------+----------------------+------------------------+
| AXI4-Stream          | Data only, burst     | Data stream and media  |
|                      | transfer             | stream transfers       |
+----------------------+----------------------+------------------------+

AXI4-Lite:

Lightweight with a simple structure, suitable for small batch data and simple control scenarios. It does not support burst transfers; only one word (32-bit) can be read or written at a time. It is mainly used for accessing low-speed peripherals and peripheral control.

AXI4:

The interface is similar to AXI-Lite, but with an added burst transfer capability, allowing continuous read/write operations to a range of addresses in one transaction. In other words, it has burst functionality for data read/write operations.

The above two types both use memory-mapped control, meaning the ARM maps user-defined IPs to specific addresses for access. Reading and writing is like accessing on-chip RAM, which makes programming convenient and development relatively easy. The cost is higher resource usage, requiring additional read address lines, write address lines, read data lines, write data lines, and write response lines.

AXI4-Stream:

This is a continuous streaming interface that does not require address lines (much like a FIFO — you just keep reading or writing). For this type of IP, the ARM cannot use the memory-mapped approach described above (a FIFO has no concept of addresses). A conversion device is needed, such as an AXI-DMA module, to convert between memory-mapped and streaming interfaces. AXI-Stream is suitable for many scenarios: video stream processing, communication protocol conversion, digital signal processing, wireless communication, etc. Essentially, it builds data paths for numerical streams, from source (e.g., ARM memory, DMA, wireless receiver front-end, etc.) to sink (e.g., HDMI display, high-speed AD audio output, etc.), creating a continuous data flow. This interface is suitable for real-time signal processing.

AXI4 and AXI4-Lite interfaces contain 5 different channels:

-  Read Address Channel

-  Write Address Channel

-  Read Data Channel

-  Write Data Channel

-  Write Response Channel

Each channel is an independent AXI handshake protocol. The following two figures show the read and write models respectively:

.. image:: images/01_media/image4.png
      
AXI Read Data Channel

.. image:: images/01_media/image5.png
      
AXI Write Data Channel

Inside the ZYNQ chip, the AXI bus protocol is implemented in hardware, including 9 physical interfaces: AXI-GP0 to AXI-GP3, AXI-HP0 to AXI-HP3, and the AXI-ACP interface.

The AXI_ACP interface is an interface defined under the ARM multi-core architecture, known as the Accelerator Coherency Port, used to manage non-cached AXI peripherals such as DMA. The PS side is a Slave interface.

The AXI_HP interface is a high-performance/high-bandwidth AXI 3.0 standard interface. There are four in total, with PL modules connecting as master devices. It is mainly used for PL to access PS storage (DDR and On-Chip RAM).

The AXI_GP interface is a general-purpose AXI interface. There are four in total, including two 32-bit master device interfaces and two 32-bit slave device interfaces.

.. image:: images/01_media/image6.png
      
As can be seen, only two AXI-GP ports are Master Ports (master interfaces), while the remaining 7 ports are Slave
Ports (slave interfaces). Master interfaces have the authority to initiate read/write operations. The ARM can use the two AXI-GP master interfaces to actively access PL logic, essentially mapping the PL to certain addresses and reading/writing PL registers as if accessing its own memory. The remaining slave interfaces are passive interfaces that accept read/write operations from the PL.

Additionally, these 9 AXI interfaces differ in performance. The GP interfaces are 32-bit low-performance interfaces with a theoretical bandwidth of 600MB/s, while the HP and ACP interfaces are 64-bit high-performance interfaces with a theoretical bandwidth of 1200MB/s. One might ask, why aren't the high-performance interfaces designed as master interfaces so that ARM can initiate high-speed data transfers? The answer is that high-performance interfaces do not need the ARM
CPU to handle data movement — the real workhorse is the DMA controller located in the PL.

The ARM on the PS side has direct hardware support for AXI interfaces, while the PL needs to implement the corresponding AXI protocol using logic. Xilinx provides ready-made IPs in the Vivado development environment, such as AXI-DMA, AXI-GPIO, AXI-Datamover,
and AXI-Stream, all of which implement the corresponding interfaces. They can be used by simply adding them from the IP list in Vivado to achieve the desired functionality. The figure below shows various DMA
IPs in Vivado:

.. image:: images/01_media/image7.png
      
Below is a functional introduction to several commonly used AXI interface IPs:

AXI-DMA: Implements the conversion from PS memory to PL high-speed transfer channel AXI-HP<---->AXI-Stream

AXI-FIFO-MM2S: Implements the conversion from PS memory to PL general-purpose transfer channel AXI-GP<----->AXI-Stream

AXI-Datamover: Implements the conversion from PS memory to PL high-speed transfer channel AXI-HP<---->AXI-Stream, except this time it is entirely controlled by the PL, and the PS is completely passive.

AXI-VDMA: Implements the conversion from PS memory to PL high-speed transfer channel AXI-HP<---->AXI-Stream, specifically designed for two-dimensional data such as video and images.

AXI-CDMA: This is used by the PL to move data from one memory location to another without CPU intervention.

We will provide examples of how to use these IPs in later chapters. Sometimes, users need to develop their own custom IPs to communicate with the PS. In this case, a wizard can be used to generate the corresponding IP. User-defined IP cores can have AXI4-Lite, AXI4, AXI-Stream, PLB, and FSL interfaces. The latter two are not used because the ARM side does not support them.

With these official IPs and wizard-generated custom IPs, users do not need to understand AXI timing in great detail (unless they actually encounter issues), because Xilinx has encapsulated all AXI timing-related details, and users only need to focus on their own logic implementation.

Strictly speaking, the AXI protocol is a point-to-point master-slave interface protocol. When multiple peripherals need to exchange data with each other, we need to add an AXI
Interconnect module, which is an AXI interconnect matrix that provides a switching mechanism to connect one or more AXI master devices to one or more AXI slave devices (somewhat similar to the switching matrix inside a network switch).

This AXI Interconnect IP core can support up to 16 master devices and 16 slave devices. If more interfaces are needed, additional IP cores can be added.

The basic connection modes of AXI Interconnect are as follows:

-  N-to-1 Interconnect

-  to-N Interconnect

-  N-to-M Interconnect (Crossbar Mode)

-  N-to-M Interconnect (Shared Access Mode)

.. image:: images/01_media/image8.png
      
Many-to-One Scenario

.. image:: images/01_media/image9.png
      
One-to-Many Scenario

.. image:: images/01_media/image10.png
      
Many-to-Many Read/Write Address Channels

.. image:: images/01_media/image11.png
      
Many-to-Many Read/Write Data Channels

The AXI interface devices inside ZYNQ are interconnected through the interconnect matrix approach, which ensures both efficient data transfer and flexible connectivity. Xilinx provides the axi_interconnect IP core in Vivado to implement this interconnect matrix, and we can simply instantiate it.

.. image:: images/01_media/image12.png
      
AXI Interconnect IP

Introduction to ZYNQ Chip Development Flow
-------------------------------------------

Since ZYNQ integrates the CPU and FPGA together, developers need to design ARM operating system applications and device drivers, as well as the FPGA hardware logic design. Development requires understanding the Linux operating system and system architecture, as well as building a hardware design platform between the FPGA and ARM systems. Therefore, ZYNQ development requires collaboration between software engineers and hardware engineers. This is what is referred to as "hardware-software co-design" in ZYNQ development.

The design and development of ZYNQ hardware and software systems require the following development environments and debugging tools:

Xilinx Vivado.

The Vivado Design Suite implements the design and development of the FPGA portion, including pin and timing constraints, compilation and simulation, and the design flow from RTL to bitstream. Vivado is not simply an upgrade of the ISE Design Suite, but a completely new design suite. It replaces all the important tools of the ISE Design Suite, such as Project Navigator, Xilinx Synthesis Technology, Implementation, CORE Generator, Constraint, Simulator, Chipscope Analyzer, FPGA Editor, and other design tools.

Xilinx SDK (Software Development Kit).
SDK is the Xilinx Software Development Kit (SDK). Based on the Vivado hardware system, the system automatically configures some important parameters, including tool and library paths, compiler options, JTAG and flash settings, debugger connections, and bare-metal board support packages (BSP). SDK also provides drivers for all supported Xilinx
IP hard cores. SDK supports co-debugging of IP hard cores (on the FPGA) and processor software. We can use high-level C or C++ languages to develop and debug ARM and FPGA systems and test whether the hardware system works properly. The SDK software is also included with the Vivado software and does not need to be installed separately.

ZYNQ development follows a hardware-first, software-second approach. The specific flow is as follows:

1) Create a new project in Vivado and add an embedded source file.

2) Add and configure basic peripherals for the PS and PL sections in Vivado, or add custom peripherals as needed.

3) Generate the top-level HDL file in Vivado, add constraint files, and then compile to generate the bitstream file (\*.bit).

4) Export the hardware information to the SDK software development environment. In the SDK environment, you can write debugging software to verify hardware and software, and use the bitstream file to independently debug the ZYNQ system.

5) Generate the FSBL file in SDK.

6) Generate u-boot.elf and bootloader images in a VMware virtual machine.

7) In SDK, generate a BOOT.bin file using the FSBL file, the bitstream file system.bit, and the u-boot.elf file.

8) Generate the Ubuntu kernel image file Zimage and the Ubuntu root file system in VMware. Additionally, drivers need to be written for FPGA custom IPs.

9) Place the BOOT, kernel, device tree, and root file system files onto the SD card, power on the development board, and the Linux operating system will boot from the SD card.

The above is a typical ZYNQ development flow, but ZYNQ can also be used solely as an ARM, in which case there is no need to worry about PL-side resources, and it is not much different from traditional ARM development. ZYNQ can also use only the PL section, but the PL configuration still needs to be done by the PS, meaning it is not possible to program PL-only firmware through the traditional Flash programming method.

Skills Required for Learning ZYNQ
----------------------------------

Learning ZYNQ is more demanding than learning traditional development tools such as FPGA, MCU, ARM, etc. Mastering ZYNQ is not something that can be achieved overnight.

Software Developers
~~~~~~~~~~~~~~~~~~~

-  Computer Architecture

-  C, C++ Languages

-  Computer Operating Systems

-  Tcl Scripting

-  Good English Reading Skills

Logic Developers
~~~~~~~~~~~~~~~~

-  Computer Architecture

-  C Language

-  Digital Circuit Fundamentals

-  Verilog, VHDL Languages

-  Good English Reading Skills
