# Programming

## Overview

In this project, I/O operations are made from both the FPGA and RT target of the roboRIO. Since there is not any hybrid mode on the roboRIO, it is quite a challenge to setup I/O on both the RT target and the FPGA and make it work simultaneously. The solution used in this project was to copy and modify the customised FPGA personality, and use this bit file for both the project and in the 'Open FPGA VI Reference'. This procedure is explained in detail in the software manual.

In the initialisation phase of the software, the FPGA is started and the correct bit file is loaded into it. When the communication LED is red the code is ready for the IMU bias calculations, and the channel 7 switch should be turned on. After the bias calculations are done, the communication LED is turned green and the main loop and IMU starts to execute. When the brake command from the RC controller is executed, the communication LED is turned off. An overview of the LabVIEW modules can be seen in the figure below.

![AutoBike-MDH](../Images/SoftwareOverview.PNG)
