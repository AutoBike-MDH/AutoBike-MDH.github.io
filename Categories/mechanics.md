## Mechanical System

This page holds an overview of the design of the mechanical system for the bicycle. This includes mounts for electronic components, safety systems, as well as a brake system.

### Steering Motor Mount

It is crucial for the steering angle precision that the motor mount is robust, straight, and doesn't rattle. The solution implemented is a rubber band attached around two cogwheels. The cogwheels have the same diameter; this is because the motor is geared so it is at this stage not desired to change the relation between them.

For the specified mechanism a motor with a gearbox and encoder has been selected. This entails that the placement of the mount have to be in the front of the main frame to ensure stability and simplify the mechanism.

### Component Mounting

The bicycle has to include a way to mount all external electronic components such as the roboRIO and different circuit boards. The mounting must fulfil the following requirements:

- It has to favour the dynamic model
- All components have to be able to be connected in an efficient manner
- It has to protect the electronics in case of a fall
- It still has to look like a bicycle after construction

A triangular shaped box in Plexiglass was created and mounted in the middle of the main frame. The box is located around the centre of gravity and it is thus possible to divide the weight evenly on both sides of the centre of the bicycle. The design includes a centre plate with both sides covered in Velcro; this solution enables the components to be placed in any formation. All of the components are strategically placed on the triangular shaped box in order to minimise the noise and to make the electrical system modular. All wires are measured and placed in a manner to make the electrical components as accessible and replaceable as possible.

### Brake System

The bicycle requires an automated brake system to be able to slow down the speed without crashing. To brake with the back wheel a 12V DC motor and a wired disc brake is used. The motor winds the wire around its rotor to tighten and loosen the disc brake and thereby slows down the speed. If the motor tightens the wire too hard there is a risk that the brake system will break; this is controlled with a motor controller of type L298 which responds to a PWM signal sent from the roboRIO. The mount of the motor is placed where the chainset of the bicycle usually is located. This is because it is easy to create a tight and stable mount for the motor in that location and also to pull the wire from the brake system to the rotor.

### Component Protection

To protect the electronics a steel beam has been mounted on the main frame. The steel beam has two useful areas; the first is to take the hit if the bicycle would fall over and the second is to use it as a handle. Below is a photo of the final construction; tennis balls has been attached to the edges to soften the impact and to get a softer grip.
