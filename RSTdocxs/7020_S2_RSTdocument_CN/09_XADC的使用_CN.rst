Using XADC
==========

**The Vivado project for this experiment is "ps_xadc".**

This chapter introduces the use of XADC. The XADC is embedded in the PS side, allowing the CPU or other hosts to connect to the XADC without using the PL side. The XADC has a maximum sampling rate of 1MSPS, a precision of 12 bits, and built-in voltage and temperature sensors that can monitor the chip's voltage and temperature information. As shown in the figure, the voltage sensors can monitor the chip's VCCINT, VCCAUX, VCCBRAM, etc. VP_0 and VN_0 are a pair of dedicated ADC analog input ports. VAUXP[*] and VAUXN[*] are also ADC input ports, but when not used as ADC inputs, they can serve as general-purpose IOs. On the AX7015/AX7021/AX7010/AX7020/AX7Z035/AX7Z100 development boards, these pins are not exposed. Therefore, this experiment mainly measures the values of the Temperature Sensor and the Supply Sensors.

.. image:: images/09_media/image1.png
      
This experiment introduces three methods to read sensor information values. First, a new Vivado project needs to be created. As before, save a copy based on the "ps_hello" project; the details will not be repeated here.

Reading XADC via Hardware
-------------------------

1. Open the project, connect the development board power supply and JTAG downloader, set the development board to JTAG mode, power on the board, click Open Hardware Manager, then click Auto Connect to discover the hardware.

.. image:: images/09_media/image2.png
      
2. Right-click on XADC and create a new Dashboard.

.. image:: images/09_media/image3.png
      
3. Modify the name and click OK.

.. image:: images/09_media/image4.png
      
4. Temperature information will be displayed by default.

.. image:: images/09_media/image5.png
      
5. Click + to add voltage values to the window.

.. image:: images/09_media/image6.png
      
6. The display is as follows.

.. image:: images/09_media/image7.png
      
The advantage of this method is the graphical display, which is intuitive, but the disadvantage is that data values cannot be obtained. The following section introduces reading XADC information via PS.

Reading XADC Information via PS
-------------------------------

1. Open the Vitis software and create a new Vitis project. The program has already been prepared and can be copied to the new project.

.. image:: images/09_media/image8.png
      
2. In the BSP, you can see that the PS has a built-in XADC peripheral.

.. image:: images/09_media/image9.png
      
3. In this experiment, xadcps.h and xadcps_hw.h are mainly used.

.. image:: images/09_media/image10.png
      
4. This experiment reads temperature and voltage data and prints it via the serial port every 1 second. The XAdcPs_GetAdcData function reads the raw values, the XAdcPs_RawToTemperature macro converts the ADC value to a temperature value, and XAdcPs_RawToVoltage converts it to a voltage value.

.. image:: images/09_media/image11.png
      
5. After downloading via Run As, the following print information can be seen in the serial port tool:

.. image:: images/09_media/image12.png
      
This method is simple and convenient for reading data information, but the signals are not visible to the PL side, making it less flexible. Reference materials: UG585, UG480. The following section introduces reading data via the AXI bus.

Reading XADC Information via AXI Bus
-------------------------------------

In the previous PS-side XADC experiment, data was read by polling. In this section, we want to add interrupts on top of polling to monitor whether the temperature exceeds a certain threshold, and if it does, generate an interrupt.

1. Add the XADC module and click Run Connection Automation with the default settings.

.. image:: images/09_media/image13.png
      
2. Reconfigure the Zynq CPU, add the PL-side interrupt, and click OK to finish.

.. image:: images/09_media/image14.png
      
3. Connect the XADC interrupt to the CPU interrupt port, re-run Generate Output Products. This time, a Bitstream needs to be generated.

.. image:: images/09_media/image15.png
      
Click Generate Bitstream to generate the FPGA download file.

.. image:: images/09_media/image16.png
      
4. Re-export the hardware. Here, select Include bitstream.

.. image:: images/09_media/image17.png
      
5. The XADC has many alarm signals, such as temperature, voltage, etc. This experiment sets the XADC temperature Temp Upper and Temp Lower values to configure interrupts. Once the temperature exceeds the Temp Upper value, an interrupt will be triggered.

.. image:: images/09_media/image18.png
      
The conversion formula between temperature values and ADC Code values is as follows. Ready-made formulas are available in the program.

.. image:: images/09_media/image19.png
      
6. Create a new Vitis project.

.. image:: images/09_media/image20.png
      
7. An additional module has been added in the BSP, which is the XADC module just added. It uses the sysmon.h and sysmon_hw.h header files.

.. image:: images/09_media/image21.png
      
8. The following sets the temperature upper and lower values, enables the global interrupt and temperature interrupt. The interrupt registers can be found in the PG091 document.

.. image:: images/09_media/image22.png
      
The temperature interrupt enable is ALM[0]; simply enable this interrupt.

.. image:: images/09_media/image23.png
      
XSysMon_IntrGlobalEnable(); Global interrupt enable function.

XSysMon_IntrEnable(); Interrupt enable function. MASK macro definitions can be used to specify which interrupts to enable.

9. In the interrupt service routine, use the XSysMon_IntrGetStatus(); function to read the interrupt status register, determine whether it is a temperature interrupt, print the information, and finally use the XSysMon_IntrClear(); function to clear the interrupt.

.. image:: images/09_media/image24.png
      
10. Open the Run Configuration window, create a new System Debugger, select Program FPGA, and click Run.

.. image:: images/09_media/image25.png
      
11. The program sets the Upper threshold to 80 degrees Celsius. When the temperature exceeds 80 degrees, an interrupt will be triggered once. After the temperature drops to the Lower temperature, if it rises above the Upper temperature again, another interrupt will be triggered. As shown in the serial output below.

.. image:: images/09_media/image26.png
      
There are many other alarms available. Different monitoring functions can be implemented by configuring the Alarm Threshold registers and interrupt registers.

.. image:: images/09_media/image27.png
      
This method can not only access temperature and voltage sensors, but also allows access from the PL side. This will not be further explained in this chapter.

Chapter Summary
---------------

This chapter introduced three methods for reading XADC, each with its own advantages and disadvantages. Users can choose the appropriate method based on their needs.
