PL "Hello World" LED Experiment
================================

**The Vivado project for this experiment is "led".**

For ZYNQ, PL (FPGA) development is crucial, and this is where ZYNQ has an advantage over other ARM processors — it allows customization of many ARM-side peripherals. Before customizing ARM-side peripherals, let us first familiarize ourselves with the PL (FPGA) development process through an LED example, and get familiar with the basic operations of the Vivado software. This development process is exactly the same as that of FPGA chips without an ARM core.

In this example, we will perform an LED control experiment, toggling the LEDs on the development board once per second, achieving an on-off-on-off control pattern. Once you can control LEDs, you will gradually learn to control other peripherals as well.

LED Hardware Introduction
-------------------------

1) The PL section of the development board is connected to 4 red user LEDs. These 4 LEDs are entirely controlled by the PL.

.. image:: images/05_media/image1.png
      
2) We can determine the binding relationship between the LEDs and PL pins based on the schematic connections.

.. image:: images/05_media/image2.png
      
Corresponding pin information

3) **IOs starting with PS_MIO in the schematic are PS-side IOs, which do not need to be assigned and cannot be assigned**

.. image:: images/05_media/image3.png
      
Creating a Vivado Project
--------------------------

1) Launch Vivado. On Windows, you can launch it by double-clicking the Vivado shortcut.

.. image:: images/05_media/image4.png
   
      
2) In the Vivado development environment, click "Create New Project" to create a new project.

.. image:: images/05_media/image5.png
   
      
3) A new project creation wizard will pop up. Click "Next".

.. image:: images/05_media/image6.png
   
      
4) In the pop-up dialog box, enter the project name and the directory where the project will be stored. Here we use "led" as the project name. Note that the "Project location" path cannot contain Chinese characters or spaces, and the path name should not be too long.

.. image:: images/05_media/image7.png
      
5) Select "RTL Project" as the project type.

.. image:: images/05_media/image8.png
      
6) Set the "Target language" to "Verilog". Although Verilog is selected, VHDL can also be used, as mixed-language programming is supported.

.. image:: images/05_media/image9.png
      
7) Click "Next" without adding any files.

.. image:: images/05_media/image10.png
      
8) In the "Part" option, select "Zynq-7000" for the device family "Family", select "clg400" for the AX7020 development board package type "Package", and select "-2" for Speed to narrow down the selection. Select "xc7z020clg400-2" from the dropdown list. "-2" indicates the speed grade; the higher the number, the better the performance. Higher speed grade chips are backward compatible with lower speed grade chips.

.. image:: images/05_media/image11.png
      
For the AX7010 development board, select the model "xc7z010clg400-1".

.. image:: images/05_media/image12.png
      
9) Click "Finish" to complete the creation of the project named "led".

.. image:: images/05_media/image13.png
      
10) Vivado software interface

.. image:: images/05_media/image14.png
      
Creating a Verilog HDL File to Light Up LEDs
--------------------------------------------

1) Click the Add Sources icon under Project Manager (or use the shortcut Alt+A).

.. image:: images/05_media/image15.png
      
2) Select "Add or create design sources" and click "Next".

.. image:: images/05_media/image16.png
   
      
3) Select "Create File".

.. image:: images/05_media/image17.png
      
4) Set the "File name" to "led" and click "OK".

.. image:: images/05_media/image18.png
      
5) Click "Finish" to complete adding the "led.v" file.

.. image:: images/05_media/image19.png
      
6) In the pop-up "Define Module" dialog, you can specify the module name "Module name" for the "led.v" file. Here, we keep the default name "led". You can also specify some ports, but we will skip that for now. Click "OK".

.. image:: images/05_media/image20.png
      
7) Select "Yes" in the pop-up dialog box.

.. image:: images/05_media/image21.png
      
8) Double-click "led.v" to open and edit the file.

.. image:: images/05_media/image22.png
      
9) Write the "led.v" code. Here we define a 32-bit register called timer, which is used to cyclically count from 0 to 49999999 (1 second). When the count reaches 49999999 (1 second), the timer register resets to 0 and toggles the four LEDs. This way, if the LEDs were off, they will turn on, and if they were on, they will turn off. The completed code is as follows:

.. code:: verilog

 `timescale 1ns / 1ps
 module led(
     input sys_clk,
     output reg [3:0] led
     );
 reg[31:0] timer_cnt;
 always@(posedge sys_clk)
 begin
     if(timer_cnt >= 32'd49_999_999)
     begin
         led <= ~led;
         timer_cnt <= 32'd0;
     end
     else
     begin
         led <= led;
         timer_cnt <= timer_cnt + 32'd1;
     end
     
 end
 endmodule

1)  Save the code after writing it.

Adding Pin Constraints
----------------------

Vivado uses xdc files as constraint files. The xdc file is mainly used for pin constraints, clock constraints,
and group constraints. Here, we need to assign the input and output ports in the led.v program to the actual FPGA pins.

1) Click "Open Elaborated Design"

.. image:: images/05_media/image23.png
      
2) Click the "OK" button in the pop-up window.

.. image:: images/05_media/image24.png
      
3) Select "Window -> I/O Ports" from the menu.

.. image:: images/05_media/image25.png
      
4) In the pop-up I/O Ports window, you can see the pin assignment status.

.. image:: images/05_media/image26.png
      
5) Bind the reset signal rst_n to the PL-side button, assign pins and voltage standards to the LEDs and clock, and click the save icon when finished.

.. image:: images/05_media/image27.png
      
6) A window pops up asking to save the constraint file. Enter "led" as the file name, keep the default file type "XDC", and click "OK".

.. image:: images/05_media/image28.png
      
7) Open the generated "led.xdc" file. We can see it is a TCL script. If we understand the syntax, we can write the led.xdc file manually to constrain the pins.

.. image:: images/05_media/image29.png
      
Below is an introduction to the most basic XDC syntax. For ordinary IO ports, only the pin number and voltage need to be constrained. The pin constraint is as follows:

**set_property PACKAGE_PIN "pin number" [get_ports "port name"]**

The voltage level constraint is as follows:

**set_property IOSTANDARD "voltage standard" [get_ports "port name"]**

Note that the text is case-sensitive. If the port name is an array, enclose it with { }. The port name must match the name in the source code exactly, and the port name cannot be the same as a keyword.

In the voltage standard, the number after "LVCMOS33" refers to the FPGA BANK voltage. The BANK where the LEDs are located has a voltage of 3.3V, so the voltage standard is "LVCMOS33".\ **Vivado requires all IOs to be assigned the correct voltage standard and pin number by default**\ .

Adding Timing Constraints
--------------------------

In addition to pin assignment, an FPGA design has another important constraint — timing constraints. Here, we demonstrate how to add timing constraints using the wizard.

1) Click "Run Synthesis" to start synthesis.

.. image:: images/05_media/image30.png
      
2) Click "OK" in the pop-up dialog box.

.. image:: images/05_media/image31.png
      
3) After synthesis is complete, click "Cancel".

.. image:: images/05_media/image32.png
      
4) Click "Constraints Wizard".

.. image:: images/05_media/image33.png
      
5) Click "Next" in the pop-up window.

.. image:: images/05_media/image34.png
      
6) The timing constraint wizard analyzes the clocks in the design. Here, set the "sys_clk" frequency to 50MHz, then click "Skip to Finish" to end the timing constraint wizard.

.. image:: images/05_media/image35.png
      
7) Click "OK" in the pop-up window.

.. image:: images/05_media/image36.png
      
8) Click "Finish".

.. image:: images/05_media/image37.png
      
9) At this point, the led.xdc file has been updated. Click "Reload" to reload the file, and save the file.

.. image:: images/05_media/image38.png
      
Generating the BIT File
-----------------------

1) The compilation process can be broken down into synthesis, place and route, and BIT file generation. Here, we directly click "Generate Bitstream" to generate the BIT file.

.. image:: images/05_media/image39.png
      
2) In the pop-up dialog box, you can select the number of tasks, which is related to the number of CPU cores. Generally, the larger the number, the faster the compilation. Click "OK".

.. image:: images/05_media/image40.png
      
3) Compilation starts, and you can see a status message in the upper right corner. During compilation, antivirus software or system managers may block execution, causing compilation to fail or take an excessively long time.

.. image:: images/05_media/image41.png
      
4) If there are no errors during compilation, a dialog box will pop up upon completion asking you to choose the next action. You can select "Open Hardware Manager", or "Cancel". Here we select "Cancel" and skip the download for now.

.. image:: images/05_media/image42.png
      
Vivado Simulation
-----------------

Next, let us try our hand at using Vivado's built-in simulation tool to output waveforms and verify whether the LED program design results match our expectations (Note: simulation can also be performed before generating the BIT file). The specific steps are as follows:

1. Set up Vivado's simulation configuration by right-clicking Simulation Settings under SIMULATION.

.. image:: images/05_media/image43.png
      
2. In the Simulation Settings window, configure as shown below. Here, set the simulation time to 50ms (set as needed). Keep other settings as default and click OK to finish.

.. image:: images/05_media/image44.png
      
3. Add a stimulus test file. Click the Add Sources icon under Project Manager, configure as shown below, and click Next.

.. image:: images/05_media/image45.png
      
4. Click Create File to generate a simulation stimulus file.

.. image:: images/05_media/image46.png
      
In the pop-up dialog box, enter the name of the stimulus file. Here we enter vtf_led_test.

.. image:: images/05_media/image47.png
      
5. Click the Finish button to return.

.. image:: images/05_media/image48.png
      
Here we do not add IO Ports for now. Click OK.

.. image:: images/05_media/image49.png
      
Under the Simulation Sources directory, a newly added vtf_led_test file appears. Double-click to open this file, and you can see that only the module name is defined, with nothing else.

.. image:: images/05_media/image50.png
      
6. Next, we need to write the content of the vtf_led_test.v file. First, define the input and output signals, then instantiate the led_test module so that the led_test program becomes part of this test program. Then add reset and clock stimuli. The completed vtf_led_test.v file is as follows:

.. code:: verilog

 `timescale 1ns / 1ps
 //////////////////////////////////////////////////////////////////////////////////
 // Module Name: vtf_led_test
 //////////////////////////////////////////////////////////////////////////////////
 
 module vtf_led_test;
 // Inputs
 reg sys_clk;
 reg rst_n ;
 // Outputs
 wire [3:0] led;
 
 // Instantiate the Unit Under Test (UUT)
 led uut (
     .sys_clk(sys_clk),   
     .rst_n(rst_n),
     .led(led)
  );
 
 initial 
 begin
 // Initialize Inputs
     sys_clk = 0;
     rst_n = 0 ;
     #1000 ;
     rst_n = 1; 
 end
 //Create clock
 always #10 sys_clk = ~ sys_clk;  
 
 endmodule

1) After writing and saving, vtf_led_test.v automatically becomes the top level of this simulation hierarchy, with the design file led_test.v below it.

.. image:: images/05_media/image51.png
      
8) Click the Run Simulation button, then select Run Behavioral Simulation. A behavioral simulation is sufficient here.

.. image:: images/05_media/image52.png
      
If there are no errors, Vivado's simulation software will start working.

10. After the simulation interface appears as shown below, the waveform displayed is the result of the simulation automatically running to the 50ms setting.

.. image:: images/05_media/image53.png
      
Since the state change time of LED[3:0] in the program design is long and simulation is time-consuming, we observe the timer[31:0] counter changes instead. Add it to the Wave window for observation (click uut under the Scope panel, then right-click timer in the Objects panel, and select Add Wave Window from the dropdown menu).

.. image:: images/05_media/image54.png
      
After adding, the timer is displayed on the Wave waveform interface, as shown below.

.. image:: images/05_media/image55.png
      
11. Click the Restart button as indicated below to reset, then click the Run All button. (Be patient!!!) You can see that the simulation waveform matches the design. (Note: The longer the simulation time, the more disk space the waveform file occupies. The waveform file is located in the xx.sim folder of the project directory.)

.. image:: images/05_media/image56.png
      
.. image:: images/05_media/image57.png
      
We can see that the led signal changes to F, indicating that LED1~LED4 all light up simultaneously.

Download
--------

1) Connect the JTAG interface of the development board and power on the board.

2) In the "HARDWARE MANAGER" interface, click "Auto Connect" to automatically connect to the device.

.. image:: images/05_media/image58.png
      
3) You can see that JTAG has detected the ARM and FPGA cores.

.. image:: images/05_media/image59.png
      
4) Select xc7z020_1, right-click and choose "Program Device..."

.. image:: images/05_media/image60.png
      
5) Click "Program" in the pop-up window.

.. image:: images/05_media/image61.png
      
6) Wait for the download to complete.

.. image:: images/05_media/image62.png
      
7) After the download is complete, we can see that the 4 LEDs start toggling once per second. At this point, the basic Vivado workflow experience is complete. Subsequent chapters will introduce how to burn the program to Flash, which requires PS system cooperation. PL-only projects cannot directly write to Flash. This is covered in the FAQ section of the "Experience ARM, Bare-Metal Output 'Hello World'" chapter.

Online Debugging
----------------

Previously, we introduced simulation and download, but simulation does not require the program to be loaded onto the board and produces idealized results. Below, we introduce Vivado's online debugging method to observe internal signal changes. Vivado has a built-in logic analyzer called ILA, which can be used to observe internal signal changes online and is very helpful for debugging. In this experiment, we observe the signal changes of timer_cnt and led.

Adding the ILA IP Core
~~~~~~~~~~~~~~~~~~~~~~

1. Click IP Catalog, search for "ila" in the search box, and double-click the ILA IP.

.. image:: images/05_media/image63.png
      
2. Change the name to ila. Since we need to sample two signals, set the number of Probes to 2. Sample Data Depth refers to the sampling depth — the higher the setting, the more signals are captured, but more resources are consumed accordingly.

.. image:: images/05_media/image64.png
      
3. On the Probe_Ports page, set the probe widths. Set PROBE0 width to 32 for sampling timer_cnt, and set PROBE1 width to 4 for sampling led. Click OK.

.. image:: images/05_media/image65.png
      
In the pop-up interface, select OK.

.. image:: images/05_media/image66.png
      
Then configure as follows and click Generate.

.. image:: images/05_media/image67.png
      
4. Instantiate the ILA in led.v and save.

.. image:: images/05_media/image68.png
      
5. Regenerate the Bitstream.

.. image:: images/05_media/image69.png
      
6. Download the program.

.. image:: images/05_media/image60.png
      
At this point, you can see the bit and ltx files. Click Program.

.. image:: images/05_media/image70.png
      
7. The online debugging window pops up, showing the signals we added.

.. image:: images/05_media/image71.png
      
Click the Run button to display the signal data.

.. image:: images/05_media/image72.png
      
You can also use triggered capture. In the Trigger Setup window, click "+", and select the timer_cnt signal for depth.

.. image:: images/05_media/image73.png
      
Change the Radix to U (unsigned decimal), and set the Value to 49999999, which is the maximum count value of timer_cnt.

.. image:: images/05_media/image74.png
      
Click Run again, and you can see the trigger is successful. At this point, timer_cnt is displayed in hexadecimal, and led toggles at the same time.

.. image:: images/05_media/image75.png
      
MARK DEBUG
~~~~~~~~~~

Above, we introduced online debugging by adding an ILA IP. Below, we introduce adding synthesis attributes in the code to achieve online debugging.

1. First, open led.v and comment out the ILA instantiation section.

.. image:: images/05_media/image76.png
      
2. Add (\* MARK_DEBUG="true" \*) before the definitions of led and timer_cnt, and save the file.

.. image:: images/05_media/image77.png
      
3. Click Synthesis.

.. image:: images/05_media/image78.png
      
4. After synthesis is complete, click Set Up Debug.

.. image:: images/05_media/image79.png
      
5) Click Next in the pop-up window.

.. image:: images/05_media/image80.png
      
Click Next with the default settings.

.. image:: images/05_media/image81.png
      
In the sampling depth window, select Next.

.. image:: images/05_media/image82.png
      
Click Finish.

.. image:: images/05_media/image83.png
      
Click Save.

.. image:: images/05_media/image84.png
      
The added ILA core constraints can be seen in the xdc file.

.. image:: images/05_media/image85.png
      
5. Regenerate the bitstream.

.. image:: images/05_media/image86.png
      
6) The debugging method is the same as before and will not be repeated here.

Experiment Summary
------------------

This chapter introduced how to develop programs on the PL side, including project creation, constraints, simulation, and online debugging methods. These methods can be referenced in subsequent code development.
