Vivado Development Environment
===============================

Introduction to Vivado Software
-------------------------------

When mentioning Xilinx's development environment, people always think of ISE first and are not very familiar with Vivado. In fact, Vivado is a new generation integrated design environment launched by Xilinx in 2012. Although its popularity is not yet high, it can be said that Vivado represents the future trend of Xilinx FPGA development environments. Therefore, as a Xilinx
FPGA developer, learning and mastering Vivado is both a trend and a necessity. As a developer, the first question that comes to mind is: since ISE already exists, why did Xilinx put so much effort into creating Vivado? In the Vivado Design Suite User Guide : Getting
Started(UG910), it is mentioned that Vivado was launched to improve designer efficiency, as it can significantly increase the design, synthesis, and
implementation efficiency for Xilinx's 28nm process programmable logic devices. It can be inferred that as FPGAs entered the 28nm era, the ISE tool seemed somewhat "outdated" — if the hardware improves but the software does not, design efficiency is inevitably affected. It was precisely for this
reason that Xilinx began planning to launch a new generation software development environment in 2008, and after 10 years of effort, created the masterpiece that is the Vivado tool.

Vivado Software Version
-----------------------

The Vivado software version is continuously being upgraded, and the latest version to date is already 2023.2. Since all the examples and tutorials for the ZYNQ development board were completed in the Vivado
2023.1 development environment, to avoid unexplainable issues caused by software version differences, we recommend that you stay in sync with us during the learning process. Users need to install Vivado
2023.1 software before use. Since the Vivado software is quite large, we do not provide a disc installation file, only a download link. Users can also download it from the Xilinx official website, which requires registering an account.

Xilinx official download address for Vivado software:\ http://china.xilinx.com/support/download.html

.. image:: images/02_media/image1.png
   
      
Vivado provides Linux and Windows versions, as well as a combined version. Here we use the combined version, which can meet both Windows and Linux development needs. Vivado requires a 64-bit operating system.

Vivado Software Installation on Windows
----------------------------------------

1) Download and extract the Vivado software package, then directly click xsetup.exe to start the installation. For a smoother installation, please close antivirus software and all PC management tools. The computer username should not contain Chinese characters or spaces.

.. image:: images/02_media/image2.png
   
      
.. image:: images/02_media/image3.png
   
      
2) If prompted for a version update, ignore the update and click "Continue".

3) Click "next" to proceed with the installation. You can see the system requirements for Vivado.

.. image:: images/02_media/image4.png
   
      
4) Select the product to install. If you do not need to use Vitis (which replaced the former SDK), select only Vivado. Generally, Vitis is needed for ZYNQ or chips with hard processor cores. If it is pure FPGA hardware and you do not plan to study software-related topics, select Vivado.

.. image:: images/02_media/image5.png
   
      
5) Here you select the device libraries to install. Since we do not need UltraScale, UltraScale+, and Versal chips, you can uncheck them to save installation space. Keep the rest as default and click "next".

.. image:: images/02_media/image6.png
   
      
6) Check "I agree" and click "next".

.. image:: images/02_media/image7.png
   
      
7) The installation path is not modified here. The installation path must not contain Chinese characters, spaces, or other special characters, and the computer username should not be in Chinese or contain spaces. You can see that Vivado requires at least 190GB of hard disk space.

.. image:: images/02_media/image8.png
   
      
8) Click "Install" to start the installation.

.. image:: images/02_media/image9.png
      
9) Wait for the installation, which takes a long time. If antivirus software and PC management tools are not closed, the installation process may be intercepted, causing the installed software to be unusable.

.. image:: images/02_media/image10.png
      
10) A prompt indicates the installation was successful.

.. image:: images/02_media/image11.png
      
11) Install the License file. Click "Copy License" and select the "xilinx_ise_vivado.lic" file.

.. image:: images/02_media/image12.png
      
12) You can see the installation was successful.

.. image:: images/02_media/image13.png
      
Reinstalling the Driver
-----------------------

Generally, the downloader driver is installed along with Vivado. If you need to install the downloader driver again, navigate to the Vivado installation path "X:\\XXX\\Vivado\\2023.1\\data\\xicom\\cable_drivers\\nt64\\digilent" and double-click the "install_digilent.exe" file to install it. Before installation, close the Vivado software first. If Vivado cannot detect the downloader, try disabling the firewall and antivirus software. Also, do not open multiple versions of Vivado or ISE at the same time.

.. image:: images/02_media/image14.png
   
      
After installation is complete,\ **connect the downloader,**\ open Device Manager, and find USB Serial Converter under Universal Serial Bus Controllers, which indicates a successful installation.

.. image:: images/02_media/image15.png
      