---
layout: page
title: Adding GPIO pins to project with an I/O Expander
permalink: /blog/io-expander/
parent: Electronics
grand_parent: Blog
nav_order: 2
---

When I built my first robot for the International Autonomous Robotics Competition 2018 at IIT-K. I had to build a robot with a bunch of sensors, motors and display. At that time, my experience was limited to Arduino Uno. It had only 19 GPIO pins and more importantly only 6 Analog Input pins. My components required more pins than what the Uno had. I decided to upgrade to a Mega without a second thought since it had more Analog Input pins.

Couple of years later in 2020, I was faced with the same limitation but with a Raspberry Pi. But you can't upgrade a Raspberry Pi like the Arduino can you? You cannot or maybe you can, to a different Single Board Computer, but it is certainly not economical. After a bit of Googling I narrowed down a few possible options like Shift Registers, Multiplexer/Demultiplexer, Auxiliary Processor and Oh Yes! an I/O expander.

The term I/O expander is self explanatory. The best part is these IO expanders come in both SPI and I2C interface options. For example: The MCP23x17, where the identifier x could be s for part with SPI or 0 for part with I2C. Since, I was already using an I2C bus for a different purpose I decided to go with MCP23017.

I purchased a [MCP23017 module](https://robokits.co.in/arduino/arduino-accessories/mcu-2317-mcp23017-16-bit-i-o-expander-serial-i2c-interface-module) which is a 16-bit I/O expander, so we get 16 additional GPIO pins. Which is the almost the number in Uno. The connections are fairly simple, I powered it with 3v3 (the operating voltage of R-Pi). Datasheet suggests it works at 5V too. Pulled the reset pin high(Active High Input), connected the I2C SDA and SCL pins to respective R-Pi pins. Finally there is the feature of configuring the I2C address. We can manually set the I2C address between 0x20 - 0x27 depending on the state of the three hardware pins A2, A1 and A0 on the IC. This is impressive because we can connect the three pins to GND to default the address to 0x20 and in case of conflicting address with another device, it can be changed.

For example: To set the address to 0x21, connect A2 and A1 to GND and A0 to VCC.

We can cascade more than one module together too. Hardware wise, both will utilise the same I2C bus and only the address has to be configured manually as explained above. So, I set them to 0x20 and 0x21 respectively. For demo, I connected B0 pin from each module to a Common Cathode RGB LED as shown below.

The following procedure is for Raspberry Pi:

Adafruit circuitpython blinka and MCP230xx should be installed before running the actual code.

`$ pip3 install adafruit-blinka`

`$ sudo pip3 install adafruit-circuitpython-mcp230xx`

The Hardware Hello World code below alternates between Red and Blue.
