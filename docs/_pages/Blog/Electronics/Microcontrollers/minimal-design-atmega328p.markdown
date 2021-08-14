---
layout: page
title: Minimal Microcontroller Circuit Design - ATmega328P
permalink: /minimal-design-atmega328p/
parent: Electronics
grand_parent: Blog
nav_order: 1
---

## Minimal Circuit Design for interfacing an ATmega328P microcontroller

Today, it is so easy to get started with Embedded Systems Development. Thanks to the Arduino ecosystem, students or just about anyone for that matter, can bring their ideas to life with ease using an off-the shelf development board. Simplistic hardware design & programming environment, great community & codebase, and the ease of deploying the code make Arduino an awesome platform.

I started my journey into the world of Embedded Systems with the modest Arduino Uno too. If you're interested in learning how to get a microcontroller up and running, once again Arduino is the best place to begin. In my opinion, the presence of a development board can help correlate the function of each pin on the microcontroller intuitively.

In fact, building the Arduino on a breadboard is simple too. Here are a couple of references from the Arduino website: [Building an Arduino on a Breadboard](https://www.arduino.cc/en/main/standalone) and [From Arduino to a Microcontroller on a Breadboard](https://www.arduino.cc/en/Tutorial/ArduinoToBreadboard).

Through this article, I'm sharing the fundamental steps a.k.a minimal circuit required to interface an ATmega328P to any system.

## Selecting the right package

ATmega328P comes in 4 different packages. Although usually, selection of package is a topic very specific to PCB Design. According to the datasheet, this microcontroller comes in four different packages with different pin counts. Hence, also from a circuit design standpoint, it is important to decide the electrical connection made to each pin. I chose to go with the Thin Quad Flat Package(TQFP) because of it's attractive size and ease of manual assembly for a surface mount microcontroller.

## What is the minimal circuit?

The minimal circuit as the name suggests is the bare minimum circuitry required to have a working microcontroller. There are four basic components of the minimal circuit:

1. Power Supply
2. Clock Circuit
3. Reset Circuit
4. Programming Interface

## Minimal Circuit Design

### 1. Power Supply

Although microcontrollers are designed for low power operation, the power supply can also be one of the factors in deciding the right one. From the Operating Voltage column of the datasheet provided by Atmel, we find that the ATmega328P can work 'normally' if a voltage in the range 1.8-5.5V is supplied. The maximum voltage you can derive from any pin will also be equal to the voltage provided as VCC; There could be slight variations because of the internal gain and noise around the circuit. Also from the absolute maximum readings table we find that the maximum voltage should not exceed 6.0V. Voltages above this level might permanently damage the chip so that's the first limiting factor.

If the power supply available is a fixed output source like the regulated power supply, it can be directly routed between the VCC and GND pins. USB adapters and ports operate at 5.0V DC which lies within the operating voltage of the ATmega328P and can hence be used as a source. In case of stand alone or mobile devices, the source is usually battery. LiPo batteries clearly stand out as favorites because of cost and size. However the voltage of LiPo batteries are in increments of 3.7V. A single cell can provide 3.7V which can be directly provided to ATmega328P but the operating voltage will become 3.7V and if the circuit has any components working on 5V signals, the controller cannot control it. Two cells in series makes the voltage 7.4V which cannot used directly without suitable conversion. Hence we'll require a voltage regulator for that purpose to step the voltage down. There are tons of choice to choose from for a regulator. I also use multiple sources like this:

![Power Supply](assets/minimal-design-atmega328p/powersupply.jpg)

As mentioned already, there are more efficient power supply designs available online. I'm soon planning to share my designs for LDO and Switching type power supplies which worked.

### 2. Clock Circuit

In microprocessors, the speed of operation is determined by the clock. Any command is divided into a set of instructions. When these instructions are executed in sequence, the command works. The cycles per instruction is a parameter that decides how many clock cycles are necessary to execute one instruction. For simplicity, let's say each command has 10 instructions and each instruction requires one clock cycle to execute. If a clock of frequency 10kHz is used, then each second can accommodate 10,000 cycles. Therefore 10,000 instructions can be execute in 1 second. Hence 1,000 commands can be executed in 1 second. However, this speed is still considered very slow. Yes, you read that right.  Our computers operate in GHz range which means 1,00,00,00,000 clock cycles in a second. The operating range of the ATmega328P clock however is 32kHz - 20MHz depending on the clock source. Operating in the MHz range means 1 million clock cycles are present and hence it has earned the name Million Instructions Per Cycle (MIPS) device.

For ATmega328P there are four choices as clock source:

- Internal Oscillator
- Resonator
- RC Oscillator
- Crystal Oscillator

The internal oscillator can provide a clock between 32kHz-8MHz depending upon how the internal fuse bits are programmed. The accuracy is low and hence it cannot be used as a reliable source for applications requiring communication like the SPI protocol. The resonator is very precise but is usually expensive for a very minimal performance upgrade compared to the crystal oscillator. One can also construct an RC network as clock, but it's very unstable due to sensitivity to temperature and other effects.

The best fit would be a crystal oscillator which is accurate, stable and cheap. Most commercial boards also use the crystal as source. If we wish to set the maximum frequency then the choice will be the widely used 16MHz crystal.  The crystal has no polarity and has to connected between pins XTAL 1 and XTAL 2 (pins 7 & 8 of TQFP package).  Two decoupling capacitors are essential to improve the stability and filter the noise around the crystal. Thus, after adding the power supply and clock to our microcontroller, the circuit will be:

![Interfacing Crystal](assets/minimal-design-atmega328p/atmega%20clock%20and%20power.jpg)

### 3. Reset Circuit

There must a source of reset for microcontrollers always. Upon reset, the microcontroller starts executing from the first line of the uploaded program in memory.  I prefer defining it like that because for a long time I thought after reset the program has to be uploaded again. The reset in necessary for a variety of reasons like unexpected performance, repeating a task etc., and can only be limited by your imagination. There are two types of reset, they are: Hardware Reset and Software Reset. In Hardware Reset, a physical stimulus is needed to initiate the reset while in Software it's done by a program or processor. A few common methods of doing the reset are: Power Off-On, Watchdog Timer, External Processor or using a switch between VCC and GND.

The most noobiest way of resetting would be turn the device off and back on. This causes to program to execute from the first line therefore accounting for a reset process. The watchdog timer can be used to time a particular process and trigger the reset internally within a microcontroller. This however has to programmed with caution. Alternatively, an external processor can also be used to produce a reset signal across the RESET pin of the ATmega328P.  Hence the latter two can be classified as software reset.

The best choice for manual reset would be to use a switch. For this purpose, we can use a momentary switch a.k.a push button switch. Since the ATmega328P requires only an active low signal across the RESET (pin no. 29 of TQFP package) longer than the minimum pulse length to perform the reset. The momentary switch can prevent the signal from being accidentally set to LOW always. Since the pulse length is only a small fraction of a second, even a quick press of the momentary switch will achieve the reset. A pull up resistor can be connected to the RESET pin to pull the voltage upto VCC when the switch is not pressed. The resistor also prevents short circuit between VCC and GND when the switch is pressed. Resistor of value between 10-100k can be used.

![Reset Circuit](assets/minimal-design-atmega328p/atmega%20reset%20and%20prog.jpg)

### 4. Programming Interface

There must be a way to program the microcontroller, otherwise it's useless. There are plenty of options for this as well. One can use the UART/USART or USB or the ISP methods.  The method using the Universal Synchronous Asynchronous Receiver Transmitter involves the use of serial communication to transfer the program to the device. Using this method we cannot program new chips because a program called the bootloader is essential to enable the communication. Hence this method can only be used to program existing devices. The USB method is very common and can also be assumed to be similar to USART as the communication is serial. However, the ATmega328P cannot be interfaced with an USB directly. We can use a USB to Serial Converter IC CH340G and its associated circuitry like in the Arduino Nano or use ATmega8U2, a microcontroller with USB interface as programmer like in the Uno.

The last method is the In System Programmer (ISP) which simple and my preferred method of programming as it allows us to burn the bootloader and also program using the same setup.  The ISP is a requires 6 pins to connect with the programmer. One can use the Arduino Uno also as an ISP programmer. The six pins are SS, MOSI, MISO, RESET, VCC, and GND. The standard ISP header comes in the configuration shown below:

![SPI Connector](assets/minimal-design-atmega328p/avr_isp_pin.jpg)

Before programming using the Arduino IDE, we have to burn the Arduino Uno bootloader. [This tutorial](https://www.instructables.com/id/Burning-the-Bootloader-on-ATMega328-using-Arduino-/) explains how to burn a bootloader to an ATmega328P. Post this you can program your microcontroller with the Arduino IDE and commands.

## User Interface

This part of the circuit is not vital to get the microcontroller running. The designer can add anything they want to make it easier to interact with the microcontroller. Generally, it's advisable to have LED's for indication. Test points to measure the voltages are also useful for debugging. Connectors can also be classified under this type. So connect the appropriate pins to the respective connectors according to the application. In our case, we are building a general purpose microcontroller so I'm connecting the standard 2.54mm headers to all the pins. Pin no.13 of Uno also contains the inbuilt LED which I've connected to the ATmega328P in pin PB5 as it maps to pin 13 once the uno bootloader is burnt. I also added a power LED to indicate a working power supply.

![LED's](assets/minimal-design-atmega328p/indicator.jpg)

## Analog Reference and Filters

One pin that was not discussed in the minimal circuit design is AREF pin. This pin is used to provide a voltage reference for the internal Analog to Digital Converter. You can set it to any particular voltage you need the ADC to operate on depending on the range of sensors or any other source of analog signals you plan to interface. Otherwise the AVCC can be used to set the voltage reference and can be left open.

The goal of any electrical circuit will be to perform its task efficiently. However, the real world might contain lot of noise from various sources depending on the environment. Thus one can add decoupling capacitors at noisy junctions (e.g motors driver), protection  from reverse currents using diodes and isolate the noise sources as much as possible. However it's not essential for the minimal working circuit of ATmega328P but for certain ARM based microcontrollers require decoupling capacitors. After all this your circuit should look like this:

![Complete Minimal Circuit](assets/minimal-design-atmega328p/minimal%20ckt%20atmega328p.jpg)

The schematic is available here. Hence the Arduino Uno can be replaced with this circuit and used in the same way. This is the general method to get any microcontroller up and working with the help of the datasheet. Yes, you no longer have to depend on bulky development boards for your projects. Isn't that great?

## Acknowledgement and References

I learnt this during university and was extremely fruitful learning which helped me design compact circuit boards for various robots.

- [Microcontroller Tutorial](https://www.build-electronic-circuits.com/microcontroller-tutorial-part1/)
