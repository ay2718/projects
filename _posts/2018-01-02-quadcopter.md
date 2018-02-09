---
layout: post
title:  "Quadcopter"
date:   2018-01-02 15:14:53 -0500
permalink: /:categories/:title
categories:
---


In tenth grade, I decided that I wanted a quadcopter, but I didn't have the budget to pay for one outright.
Therefore, I decided to build my own.
I bought an Arduino, a cheap accelerometer and gyroscope chip, and the cheapest Hobbyking brushless motors and speed controllers.
These were all haphazardly strapped onto a plywood frame held together with nylon bolts.
When I first tested it, a combination of factors prevented the quadcopter from leaving the ground.
A lack of knowledge of control theory meant that the quadcopter stabilization algorithm that I had programmed into the Arduino was already malfunctioning, and the dirt cheap motors spun at inconsistent speeds.
To make matters worse, the frame was unacceptably floppy.

Many months and two hand injuries later, after buying better motors and redesigning the frame and the control algorithms, the quadcopter took its first shaky flight in the kitchen.
The PID tuning coefficients were somewhat off, but they were close enough that I could tune the quadcopter to get stable flight.

Tragedy struck when the quadcopter was first flown in an unconfined open area.
Fifteen seconds after takeoff, one of the propellers spun itself off, and the quadcopter ungracefully crashed into an empty parking lot.
Fortunately, no important components were damaged beyond repair, and the frame was redesigned yet again.
Each time the quadcopter crashed and broke the frame beyond repair (this happened several times), the frame was redesigned and reinforced.
In its current form, the frame is lightweight, rigid, and nearly indestructible.

The brain of the quadcopter is the Arduino.
I programmed the Arduino to receive radio signals, communicate with the gyroscope chip, send signals to the motor controllers, and also perform the stabilization algorithms necessary for flight.
The quadcopter uses nested PID loops in order to fly.
In an ideal system tuned with a PID controller, the process variable will change roughly linearly when the control variable is held constant.
Since the control variable is motor thrust, the process variable is naturally angular velocity.
However, flying a quadcopter is difficult without direct control over its angular position.
The solution is to use a PID loop to control angular velocity, and then have a PID loop with angular position as the process variable and angular velocity as the control variable.
In this case, the I and D coefficients can be set to zero, so I really only have a P controller for angular position. The code for the Arduino can be found [here][quad-github].

In order to tune the PID coefficients without risking crashing the quadcopter, I wrote a Python simulation of the quadcopter along one axis.
Code for the Python simulation can be found [here][simu-github].
The simulation included motor electrical characteristics, inherent delays in the Arduino programming, as well as the option to add a systematic bias or random noise.

## Pictures

<img src="/assets/quadcopter/Quadcopter4.jpg" alt="Quadcopter top view" style="float: left; width: 49%; margin-right: 2%; margin-bottom: 20px"/>
<img src="/assets/quadcopter/Quadcopter5.jpg" alt="Quadcopter bottom view" style="float: left; width: 49%; margin-bottom: 20px"/>

As can be seen in the above pictures, the quadcopter frame consists of three main parts.
The structural center of the quadcopter is an x-shaped piece of 1/4" plywood reinforced with vertical pieces of plywood.
This element is extremely stiff and strong, and it holds the other two components.
The electronics board is weakly attached to the center piece, so that it can break free in the event of a crash.
Having the entire board break free is better than having individual components fly off.
The arms are made of 1 cm^2 basswood, and they are bolted to the frame with nylon bolts so that they can be easily removed if they break.
Motor controllers, motors, and landing gear are attached to the arms.
Overall, the frame is designed to be lightweight, durable, and easily repairable.

## Update!

I recently upgraded the quadcopter software and fixed two major bugs.
First, I was using a library that computed a position approximation from accelerometer and gyroscope measurements on the chip itself.
Unfortunately, this library was not designed for quadcopters.
In order to get a position estimate from a gyroscope and accelerometer signal, the gyroscope is integrated and the accelerometer data is used to correct drift.
It is typical for a quadcopter to undergo significant acceleration for a couple of seconds, during which the accelerometer data should not align with the gyroscope data.
I wrote a new library that accounted for and minimized the effects of large extended acceleration, resulting in a much more stable quadcopter.

The second and more insidious bug was a problem with the built in Arduino I2C library.
I eventually realized that this library had no timeout, so a failed transition would irreparably lock up the entire Arduino, resulting in a crash.
Worse yet, the servo library uses timer interrupts independent of the main process, so without any updates from the main loop, the motors would continue spinning instead of stopping immediately.
I found a new I2C library with a timeout, and I haven't seen any crashes yet.

Unfortunately, I used extremely low quality plywood when building the quadcopter, and I broke the main part of the frame shortly after fixing the software.
I ordered carbon fiber rod and sheet, and I am building a new frame entirely out of carbon fiber.
Pictures will be posted when it is done.

[quad-github]: https://github.com/ay2718/arduino-quadcopter
[simu-github]: https://github.com/ay2718/quadcopter-pid-tuner
