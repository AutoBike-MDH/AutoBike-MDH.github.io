## Electronic System

### Block Diagram of Electronic System

![AutoBike-MDH](../Images/Block_Diagram_v_2.png)

### Microcontroller

The NI roboRIO is a development board specialised for robotic applications, featuring an FPGA along with a normal processor. It is expensive but it houses the ability for real time implementation which makes it ideal for use as the the main computer in the AutoBike.

### IMU

One important state of the bicycle is its orientation in the room, that is, its lean angle. To get access to this information an Inertial Measurement Unit (IMU) is facilitated. This sensor is a composition of three separate sensors; these are an accelerometer, a gyroscope, and a magnetometer. The MPU-9250 is used for the AutoBike. This IMU is developed by InvenSence initially for aerial applications, such as quadcopter.

### Current Sensor

In order to efficiently secure the safety of the battery and the electrical system it is crucial to monitor the current flow and, if needed, save the history in a log. Two current sensor modules of type PmodISNS20 have been installed; the first one monitors how much current that is drawn from the battery, and the second one monitors how much current the steering motor is consuming. The current sensor is put in series with the positive power line which allows measuring of current; the information is then sent over serial peripheral interface to the roboRIO.

### Hall Sensor

To be able to measure the speed of the bicycle a speed measurement sensor needs to be implemented. The original electrical bike sensor uses an internal Hall sensor embedded in the rear wheel bicycle motor. However this sensor proved to be very hard to access, therefore an external hall sensor is implemented.

The sensor of choice is the Honeywell NPN Hall effect sensor. Being an NPN sensor, the sensor needs a pull up resistor to the signal cable, in practice the Vdd to the sensor is connected to the signal output with a 68 kOhm resistor. The output signal is a linear analogue signal, however the signal is only used to sense a magnet passing by; this means that the magnet sensed by the sensor will drive output signal Vdd to ground, which is interpreted as a digital signal instead of original analogue signal.
