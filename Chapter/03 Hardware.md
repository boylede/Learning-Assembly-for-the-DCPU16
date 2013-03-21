---
layout: index
title: Chapter 3: Hardware
subtitle: Some kind of subtitle.
permalink: 01
---
## Hardware
Doing some basic math on the computer is all well and good, but lets explore talking with hardware.

On the DCPU, interfacing with hardware is done in a very consistant manner. First, you need to know what hardware is connected and what order it is connected in, so that your program can talk to each piece of hardware individually.

In order to do this, you first get the number of hardware devices connected,```hwn i```, and then iterate through them, talking to each one in turn to learn about it. ```hwq i``` will ask the hardware to return details about itself in the registers. ```hwi i``` will send a signal to the hardware called an Interrupt.

### Hardware Interrupts
Each piece of hardware is capable of talking directly with the DCPU's memory and getting the values of registers. The hardware is only allowed/supposed to do this when you ask it to by sending it an interrupt. 

Once you know which device you want to comminucate with you can use ```hwi``` to send it an interrupt.

```
; 	send an interrupt to the 21st piece of hardware connected
hwi 21
; 	send an interrupt to the i'th piece of hardware.
hwi i
```
Usually, the value of the ```a``` register, and possibly other registers, is set in order to communicate a certain intention to the piece of hardware.

Hardware specifications are posted to ```http:///www.dcpu.com```, and it is a good idea to peruse them in order to understand what kind of things you can do with hardware. All of the functionality is general enough to allow the programmer a fair amount of control over what is happening. This also means that not a whole lot is done for you, but that is OK as far as I am concerned.

