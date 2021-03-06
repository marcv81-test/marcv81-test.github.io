---
layout: post
title: 'DIY IMU - Part 1: build'
date: '2013-10-29T00:50:00+08:00'
tags:
- imu
- accelerometer
- magnetometer
- python
- arduino
- stripboard
- gyroscope
---
Now that we can read from the accelerometer/gyroscope and the magnetometer we have all we need to build an Inertial Measurement Unit (IMU).

The first step is to secure the MPU-6050 and the HMC5883L together. For this I built a tiny 8x4 stripboard. With the breakout boards GY-521 and GY-273 it's easy to align VCC, GND, SCL, and SDA. The pin headers need to be soldered in a way which allows this. I soldered mines so that the pins are on the side with no components on both breakout boards. I also punched 3mm holes in a blank plastic card and attached the sensors with M3 nylon screws. This way there is no relative movement between the MPU-6050 and the HMC5883L, and their axes are reasonably well aligned.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-29-diy-imu-part-1-build-1.jpg)
{:refdef}

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-29-diy-imu-part-1-build-2.jpg)
{:refdef}

We shall now agree on an [axes convention](https://en.wikipedia.org/wiki/Axes_conventions). Most aircrafts use the following, so we'll use that as well.

- The X axis points ahead
- The Y axis points to the right
- The Z axis points downwards

I decided that the front of the IMU would be opposite to where the cables go, and that the photo above shows the IMU the right side up. According to the drawings on the breakout boards, the X and Y accelerometer axes already follow our convention. For the magnetometer we need to reverse the direction of the Y axis then swap the X and Y axes. I implemented this with [preprocessor macros](https://github.com/marcv81/quadcopter/blob/322f3bd33da00e50fe8f5c1d2fc6bed426e20a90/libraries/IMU/IMU.h). Neither of the breakout boards have the Z axis drawn so we'll have to determine its direction experimentally.

I wrote a [new sketch](https://github.com/marcv81/quadcopter/blob/322f3bd33da00e50fe8f5c1d2fc6bed426e20a90/sketches/RawIMUTest/RawIMUTest.ino) to output the accelerometer and magnetometer data. It works well in the Arduino serial monitor, but my brain struggles to process six floating point numbers every few milliseconds. I hence decided to plot a somehow more graphical representation of the data. I saw a number of good demos in [Python](https://www.python.org/) so I was eager to give it a try. I installed the main package and two extra components: [VPython](https://www.vpython.org/) and [pySerial](https://pyserial.sourceforge.net/). I'm new to Python but it's all very simple: I wrote a script to display the accelerometer vector in blue and the magnetometer vector in green and it took less than [20 lines](https://github.com/marcv81/quadcopter/blob/322f3bd33da00e50fe8f5c1d2fc6bed426e20a90/python/RawIMUTest/RawIMUTest.py). The only caveat is that pySerial or VPython do not seem to manage a constant stream of data at 115200 baud too well, so we included a 100ms delay in the sketch.

{:refdef: style="text-align: center;"}
![image]({{ site.baseimg }}/images/quadcopter/2013-10-29-diy-imu-part-1-build-3.jpg)
{:refdef}

The IMU is not subject to any other significant acceleration than the gravity so the [accelerometer](https://en.wikipedia.org/wiki/Accelerometer) mostly measures the gravity vector. For some reason the gravity is upward. The IMU is not in proximity to important sources of magnetic interferences, so the [magnetometer](https://en.wikipedia.org/wiki/Magnetometer) mostly measures the Earth's magnetic field vector. I'm in London where the [magnetic dip](https://www.geomag.bgs.ac.uk/data_service/models_compass/wmm_calc.html) is about 66 degrees.

The first try did not give the expected results: the angle between the vectors did not remain constant as expected. This was a problem with the axes. I reviewed the [HMC5883L registers map](https://www51.honeywell.com/aero/common/documents/myaerospacecatalog-documents/Defense_Brochures-documents/HMC5883L_3-Axis_Digital_Compass_IC.pdf) and realised that I had swapped the Y and Z axes. Once this was [fixed](https://github.com/marcv81/quadcopter/commit/ad3171cef16887ff457030b971b38fec3f5d2374) everything else worked fine.

We now have a reliable way to measure the accelerometer and magnetometer vectors. If the gyroscope works too we should be able to combine the data to determine the IMU attitude.
