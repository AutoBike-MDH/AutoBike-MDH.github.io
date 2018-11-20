# Programming

## Overview

In this project, I/O operations are made from both the FPGA and RT target of the roboRIO. Since there is not any hybrid mode on the roboRIO, it is quite a challenge to setup I/O on both the RT target and the FPGA and make it work simultaneously. The solution used in this project was to copy and modify the customised FPGA personality, and use this bit file for both the project and in the 'Open FPGA VI Reference'. This procedure is explained in detail in the software manual.

In the initialisation phase of the software, the FPGA is started and the correct bit file is loaded into it. When the communication LED is red the code is ready for the IMU bias calculations, and the channel 7 switch should be turned on. After the bias calculations are done, the communication LED is turned green and the main loop and IMU starts to execute. When the brake command from the RC controller is executed, the communication LED is turned off. An overview of the LabVIEW modules can be seen in the figure below.

![AutoBike-MDH](../Images/SoftwareOverview.PNG)

## Steering Motor Encoder

The output from the encoder is a quadrature phase signal, which is made out of two square waves shifted 90 degrees from each other. When the motor is running at maximum speed the two square waves from the encoder have a frequency of approximately 65 KHz each. Because of Nyquist theorem, the decoder has to run at a rate of at least 130 KHz. To make sure that we do not miss any pulses and to utilise the power of the roboRIO, the encoder is implemented in a single cycle timed loop on the FPGA. To send data between the FPGA and RTOS multiple Read/Write Control VIs from the FPGA palette are utilised. The position of the handlebar is sent both to the RT system to be used as an input for stabilising the bicycle, and to another loop on the FPGA which acts as an safety function and do not allow the handlebar to pass ±45 degrees. To send data between the different loops on the FPGA, local variables are utilised.

## Steering Motor Controller

To control the steering motor, a PWM signal is generated on the FPGA target with a frequency and duty cycle determined by the control loop used for stabilising the bicycle. To control the direction of the motor, a digital high or low is sent through a digital I/O from the FPGA. As mentioned in the previous section a safety function is also present and if the position of the handlebar is ±45 degrees the steering motor gets a PWM signal with duty cycle of 0%; if the switch direction button is pressed the motor starts function normally again.

## IMU

The raw data from the IMU is transferred to the roboRIO with SPI communication. On the master device, in this case the roboRIO, the raw data is read on RT target utilising the SPI express VI. To be able to transfer data, a digital Chip Select (CS) signal has to be set low before transmission and set high after the transmission is completed. With the different CS it is possible to communicate with multiple slaves using the same communication bus. 

To initialise the communication with the IMU, a number of registers and offsets are configured on the IMU from the roboRIO. The 16 bit raw data are computed to a gyroscope and accelerometer value using the equation below.


<a href="https://www.codecogs.com/eqnedit.php?latex=Value=\frac{(Rawdata-Bias)}{Sensitivity}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Value=\frac{(Rawdata-Bias)}{Sensitivity}" title="Value=\frac{(Rawdata-Bias)}{Sensitivity}" /></a>


The bias is calculated in the initialisation phase of the code, where 3000 readings from the gyroscope and accelerometer are averaged and a bias value is produced for each axis for both the accelerometer and gyroscope. When the bias is calculated it is important that the bicycle is in an upright position standing as still as possible. The sensitivity value is found in the MPU9250 data sheet, and are set using the  GYRO\_FS\_SEL and ACCEL\_FS\_SEL in register 27 and 28 respectively for the gyroscope and accelerometer. The gyroscope sensitivity is currently set to 16.4 and the accelerometer sensitivity is 2048.

To acquire the roll and pitch values, the values from the accelerometer and gyroscope are merged and the rapid changes of the accelerometer and the slow changes from the gyroscope are neglected using a complementary filter. In the equation below an example of how the roll angle is calculated is shown; the procedure for the pitch angle is similar but with data from y-axis of the gyro and the pitch contribution from the accelerometer.


<a href="https://www.codecogs.com/eqnedit.php?latex=Roll_{new}=a(Gyro_X&plus;Roll_{old})&plus;(1-a)Roll_{acc}&space;\text{,\quad&space;where&space;}&space;a=0.94" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Roll_{new}=a(Gyro_X&plus;Roll_{old})&plus;(1-a)Roll_{acc}&space;\text{,\quad&space;where&space;}&space;a=0.94" title="Roll_{new}=a(Gyro_X+Roll_{old})+(1-a)Roll_{acc} \text{,\quad where } a=0.94" /></a>

From the equation, one can see that the values from the gyroscope are integrated over time. If the value of *a* is increased, more rapid changes are excluded from the accelerometer; however, more drifting from the gyroscope is included. The Roll<sub>acc</sub>, which is the contribution from accelerometer to the roll angle is calculated using the first equation below, and to calculate the pitch angle the second equation below is used.

<a href="https://www.codecogs.com/eqnedit.php?latex=Roll_{acc}=atan2(AccX,AccZ)\frac{180}{\pi}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Roll_{acc}=atan2(AccX,AccZ)\frac{180}{\pi}" title="Roll_{acc}=atan2(AccX,AccZ)\frac{180}{\pi}" /></a>


<a href="https://www.codecogs.com/eqnedit.php?latex=Pitch_{acc}=atan2(AccY,AccZ)\frac{180}{\pi}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Pitch_{acc}=atan2(AccY,AccZ)\frac{180}{\pi}" title="Pitch_{acc}=atan2(AccY,AccZ)\frac{180}{\pi}" /></a>

## Current Sensors

The current sensors communicate with the roboRIO over the SPI protocol, but with separate CS signals from the IMU. The raw data consists of 16 bits per frame, however, the data of interest are only 12 bits long and complemented with 4 stuffed bits. The data is converted to a mA value using the equation below. To tune the readings from the current sensors, the number inside the the parentheses, in this case 2042, should be changed.


<a href="https://www.codecogs.com/eqnedit.php?latex=(Rawdata-2042)\frac{1000}{89.95}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?(Rawdata-2042)\frac{1000}{89.95}" title="(Rawdata-2042)\frac{1000}{89.95}" /></a>


## Brake Controller

When the main loop has finished executing, the PWM to the forward motor and the steering motor are set to 0% duty cycle. In addition to turning off the motors, the brake motor is turned on for three seconds to assure that the bicycle comes to a complete stop. At the moment, the brake is only activated when the bicycle is shut down, but it would be possible to integrate in the main loop to have the bicycle brake to tune the velocity of the bicycle. After the brake motor has finished executing the FPGA execution is also aborted.

## Speed Control

There are 12 magnets mounted on the back wheel of the bicycle; each magnet triggers two peaks when passing by the hall sensor which makes it basically an irregular PWM signals that is read by the FPGA on the roboRIO. By simply measuring the time it takes between the magnets the velocity is extracted.

The speed control works in two stages; when the error between the reference speed and the current speed is more than \pm1km/h the increase or decrease in duty cycle to the motor is regulated with only a P controller. However, when the error is less than \pm1km/h, the P controller is switched to a PI controller to ensure a small steady state error, and to reach the reference speed in reasonable time.

## Safety Switch

If the software fails to engage its safety procedures, a mechanical structure needs to take control to ensure that the handlebar doesn't continually keeps turning. The chosen design is a simple circuit header that that gets pulled out from its socket when the handlebar goes 75 degrees in either direction. The mechanics to pull the header is similar to the steering of a soapbox car, where to steer the car you pull the string either left or right to turn. In this case the steering motor does the steering and the string simply pulls the header when the handlebar goes out of range. 

The motor controller needs main power, ground, a PWM signal, and Direction signal. The PWM signal is needed to invoke a duty cycle to the motor. Grounding the PWM pin will make the controller think that a PWM signal is being sent with a duty cycle of 0% and therefore cuts the power to the motor. By routing the PWM signal through the header the system can mechanically switch off the power to the motor, if the header is pulled.

## Remote Control

To be able to remotely control some functionality of the bicycle, a TGY-i6S remote control is used together with a TGY-iA6C receiver. From the receiver a pulse position modulation (PPM) signal is sent to the roboRIO, where it is demodulated on the FPGA. The PPM signal starts with a 4-10 ms sync signal, and followed by the eight channels and their corresponding data. When the signal has been demodulated, it is possible to see that if a switch is moved, the corresponding channels' high pulse will be longer or shorter, depending on the position of the switch. 

### Remote control function assignment

The remote controller purpose is to be able to remotely brake and shut down the bike if needed. However, switching between walk assist mode and cruise control, manually adjusting the steering, and IMU calibration functionaries are just as important. The receiver has a capacity of 8 channels; the first four channels are predefined as the joysticks, while the remaining channels are allocated for auxiliary channels. These are configurable through the controller UI. The auxiliary channels are configured to the switches SWA, SWB, SWC, and SWD. Switches SWA and SWD have two states whereas the SWB and SWC has the three switchable states. SWA is the Emergency stop, while SWB has two functionalities. The first is the Walk assist and the second is the Cruise control. SWC is a safety switch with the only purpose to avoid accidental operations of the steering motor. SWD is the calibration switch; it is mainly used at the beginning to obtain the biased values for the IMU.
