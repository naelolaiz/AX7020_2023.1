# Xilinx Zynq 7000 Series Development Board AX7020  
## Development Board Introduction
### Development Board Overview
This development board uses Xilinx's Zynq7000 series chip, model XC7Z020-2CLG400I,
with a 400-pin FBGA package. The ZYNQ7000 chip can be divided into the Processor System (PS)
and Programmable Logic (PL) parts. On the AX7020 development board, both the PS
and PL parts of the ZYNQ7000 are equipped with a rich set of external interfaces and peripherals for user convenience and functional verification. Additionally, the development board
integrates a Xilinx USB Cable downloader circuit, allowing users to download and debug the development board with just a USB cable.
### Key Features
  1. +5V power input, maximum 2A current protection; 
  2. Xilinx ARM+FPGA chip Zynq-7000 XC7Z020-2CLG400I   
  3. Two high-capacity 4Gbit (8Gbit total) high-speed DDR3 SDRAM, which can be used as data cache for the ZYNQ chip and also as memory for running the operating system;
  4. One 256Mbit QSPI FLASH, which can be used for storing system files and user data of the ZYNQ chip;   
  5. One 10/100M/1000M Ethernet RJ-45 interface, for Ethernet data exchange with computers or other network devices;  
  6. One HDMI video input/output interface, capable of 1080P video image transmission; 
  7. One high-speed USB2.0 HOST interface, for connecting peripherals such as mouse, keyboard, and USB drive;
  8. One high-speed USB2.0 OTG interface, for OTG communication with PC or USB devices; 
  9. One USB UART interface, for serial communication with PC or external devices;
  10. One RTC real-time clock, equipped with a battery holder, battery model CR1220.
  11. One EEPROM 24LC04 with IIC interface;
  12. 6 user LEDs, 2 PS-controlled, 4 PL-controlled;
  13. 7 buttons, 1 CPU reset button, 2 PS control buttons, 4 PL control buttons;
  14. One 33.333MHz active crystal oscillator on board providing a stable clock source for the PS system, and one 50MHz active crystal oscillator providing an additional clock for PL logic;
  15. One 12-pin expansion port (2.54mm pitch), for expanding the MIO of ZYNQ's PS system;
  16. One USB JTAG port for debugging and downloading to the ZYNQ system via USB cable and onboard JTAG circuit. One Micro SD card slot (on the back of the development board), for storing operating system images and file systems.
  17. Two 40-pin expansion ports (2.54mm pitch), for expanding the IO of ZYNQ's PL part. Can be connected to expansion modules such as 7-inch TFT module, camera module, and AD/DA module;

# AX7020 Document Tutorial Link
https://ax7020-20231-v101.readthedocs.io/zh-cn/latest/7020_S1_RSTdocument_CN/00_%E5%85%B3%E4%BA%8EALINX_CN.html

 Note: You can switch between Chinese and English languages at the footer at the end of the document

# AX7020 Examples
## Example Description
This project contains the factory examples for the development board, supporting most peripherals on the board.
## Development Environment and Requirements
* Vivado 2023.1
* AX7020 development board
## Creating a Vivado Project
* Download the latest ZIP package.
* Create a new project folder.
* Unzip the downloaded ZIP package into this project folder.


There are two ways to create a Vivado project, as follows:
### Using Vivado tcl console to create a Vivado project
1. Open the Vivado software and use the **cd** command to enter the "**auto_create_project**" directory and press Enter
```
cd \<archive extracted location\>/vivado/auto_create_project
```
2. Enter **source ./create_project.tcl** and press Enter
```
source ./create_project.tcl
```

### Using bat to create a Vivado project
1. In the "**auto_create_project**" folder, there is a "**create_project.bat**" file, right-click to open it in edit mode, and modify the vivado path to your local vivado installation path, save and close.
```
CALL E:\XilinxVitis\Vivado\2023.1\bin\vivado.bat -mode batch -source create_project.tcl
PAUSE
```
2. Double-click the bat file under Windows.


For more information, please visit [ALINX website](https://www.alinx.com)