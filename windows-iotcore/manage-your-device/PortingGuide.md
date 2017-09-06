---
title: Porting Guide for Windows 10 IoT Core
author: saraclay
ms.author: saclayt
ms.date: 09/05/17      
ms.topic: article
ms.prod: Windows
ms.technology: IoT
description: Learn how to migrate existing code to Windows.
keywords: windows iot, porting, AVR, quark, migrate, microcontroller, microprocessor
---

# Porting Guide

## Architectural differences between AVR and Quark
___

### Real-time OS vs General OS
The key difference between general-computing operating systems and real-time operating systems is the need for "deterministic" timing behavior in the real-time operating systems. - belhob.wordpress.com

- [Real-time OS](http://en.wikipedia.org/wiki/Real-time_operating_system) - The scheduling algorithms guarantee deterministic timing (e.g. AVR)
- [General OS](http://en.wikipedia.org/wiki/Operating_system) - The scheduling algorithms make no guarantees about timing (e.g. Windows)

___

### Microcontroller verses Microprocessor (CPU)

Microcontrollers are designed for embedded applications, in contrast to the microprocessors used in personal computers or other general purpose applications. - wikipedia.org
- [Microcontroller](http://en.wikipedia.org/wiki/Microcontroller) - A small computer on a single integrated circuit containing a processor core, memory, and programmable input/output peripherals (e.g. Atmel ATmega328).
- [Microprocessor](http://en.wikipedia.org/wiki/Microprocessor) - a multipurpose, programmable device that accepts digital data as input, processes it according to instructions stored in its memory, and provides results as output (e.g. Intel Quark).

## Serial

### [Logic Level Voltages](http://en.wikipedia.org/wiki/Logic_level#Logic_voltage_levels)

The voltage levels used internally are called the "logic level", while the voltage levels used externally are called the "line level". In particular, when connecting a system that uses TTL levels internally to a RS-232 cable, the TTL levels are the "logic level". When connecting a system that uses 3.3 V CMOS levels internally to an IEEE 1284 bus, the TTL levels are the "line level".

- CMOS
   - LOW - 0V to 1/3Vdd
   - HIGH - 2/3Vdd to Vdd

- TTL

   - LOW - 0V to 0.8V</li>
   - HIGH - 2V to 5V</li>

### [RS232](http://en.wikipedia.org/wiki/RS-232)

  The RS-232 standard is commonly used in computer serial ports. The standard defines the electrical characteristics and timing of signals, the meaning of signals, and the physical size and pinout of connectors. The current version of the standard is TIA-232-F Interface Between Data Terminal Equipment and Data Circuit-Terminating Equipment Employing Serial Binary Data Interchange, issued in 1997.

___

## Porting Code

### Arduino/AVR

Direct Port Manipulation

- [Port Registers](http://www.arduino.cc/en/Reference/PortManipulation): The port registers allow you to set a block of Arduino pins with a single instruction, resulting in performance gains.
  - This can be ported by issuing the equivalent instruction for each pin represented in the bitmask.
- DDR[B/C/D] = pinMode();
- PORT[B/C/D] = digitalWrite();
- PIN[B/C/D] = digitalRead();


- [SPI Registers](http://www.arduino.cc/en/Tutorial/SPIEEPROM):
	  This fine grain level of control is not offered by the Windows Developer Program for IoT and in most cases, simply using the SPI library can replace this functionality.

### GCC

Non-portable GCC compiler commands/options

- `__atrribute__(__packed__)`
This can be replaced by pushing a pack attribute on the data alignment stack [i.e. <code>#pragma pack(push, 1)</code>], then popping it off once your structs have been defined [e.g. `#pragma pack(pop)</code>`].

   Check [MSDN](http://msdn.microsoft.com/en-us/library/vstudio/2e70t5y1(v=vs.100).aspx) for more details.

- `asm volatile("nop");`
The same functionality exists on Windows, however the syntax is different <code>__asm nop</code>. The MSVC compiler does not optimize around assembly, so the `volatile` is not valid.

   For a deeper discussion please check [StackOverflow](http://stackoverflow.com/questions/25878898/is-asm-nop-the-windows-equivalent-of-asm-volatilenop-from-gcc-compile).
