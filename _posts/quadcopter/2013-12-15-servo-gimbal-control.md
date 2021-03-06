---
layout: post
title: Servo gimbal control
date: '2013-12-15T15:57:00+08:00'
tags:
- servo
- 2dof
- pid
- imu
- gimbal
---
A good way to test the few libraries we have so far is to build a servo gimbal controller. The aim is to use the IMU to keep the 2 DOF arm horizontal.

Let's first tape the IMU to the  arm.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-12-15-servo-gimbal-control-1.jpg)
{:refdef}

Before anything else we have to extract the roll, pitch, and yaw from the attitude quaternion. There are different conversion formulas depending on the rotation sequence, but in this case the angles are relatively small so we don't need to worry about the order too much. I used [this formula](https://en.wikipedia.org/wiki/Conversion_between_quaternions_and_Euler_angles#Conversion), here is [the code](https://github.com/marcv81/quadcopter/commit/fc5ebfe5639c7de99b923ec3587c71191e52d16f).

**Open loop**

In open loop mode the IMU is fixed to the (imaginary) vehicle frame. Depending on the vehicle attitude we drive the servos to compensate for the roll and pitch. This method is very straightforward to implement but rather imprecise. Here is [the implementation](https://github.com/marcv81/quadcopter/tree/0642e581e9989f09c3f41357142f9333297cdca8/sketches/OpenLoopServoGimbal), I'll post a video.

**Closed loop**

In closed loop mode the IMU is fixed to the gimbal. The controller adjusts the servo position to null the IMU roll and pitch. This method can be very precise because we can directly read the gimbal orientation and correct even the slightest error.

Different types of controllers can be used. I tried a simple PID and got good results. I first implemented just the proportional (P) term: it worked well but the precision wasn't too good. In order to increase the precision I added an Integral (I) term. I left the Differential (D) term alone because the servo speed is the bottleneck, and there isn't very much to do to improve the PID setting time. Here is [the implementation](https://github.com/marcv81/quadcopter/tree/f1a68d44ebbdd56d1f5b6e7d69701fb8f067c73a/sketches/ClosedLoopServoGimbal), I'll post a video.
