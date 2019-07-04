# GCSpark

The goal of this project is to create a low-cost, easily buildable USB adapter
for Gamecube input peripherals.

## Why?

I own a Wii dance mat and want to play [StepMania](https://www.stepmania.com)
on it.

Existing commercial solutions tend to be expensive, and cheaper clones are
known to have issues. Previously published DIY projects use overpowered
(and thus expensive) microcontrollers, are hard to build for inexperienced
people and/or do not implement rumble.

Basically, the goal is to make something that can be built by anyone that knows
how to use a soldering iron.

## Overview

I decided to choose the [Digispark](http://digistump.com/products/1) Attiny85
development board as the base of this project. Clones of it are affordable and
very easy to source. The integrated USB plug or port simplifies wiring, and
using it as an USB device with
[V-USB](https://www.obdev.at/products/vusb/index.html) is well-documented.
Since it is a development board, no ISP is required to program it.

Gamecube controllers expect both a 5V and a 3.3V supply and operate on 3.3V
logic level. To cut down on the amount of external components, a breakout board
for the AMS1117 LDO is used to supply 3.3V. The bidirectional level shifting
will be done using a traditional circuit based around the 2N7000 MOSFET, which
is hopefully able to switch at the required 1 MHz.

Instead of emulating a USB PID (physical interface device) for rumble support,
which other developers seem to have no luck with, the Microsoft XInput protocol
will be used. Plain USB HID support may be added in the future, either
switchable (if memory limitations allow it) or as alternative firmware.

## Cost Estimation

The following estimation is based on the lowest AliExpress prices, including
shipping to Europe, at time of writing. Some components are primarily available
in bulk, and the PCS column shows the bulk size.

| Count | Component               | Est. Cost | PCS |
|-------|-------------------------|-----------|-----|
| 1     | Digispark (clone)       | $1.30     | 1   |
| 1     | AMS1117 breakout board  | $1.20     | 5   |
| 1     | GC extension cable      | $2.00     | 1   |
| 1     | Perfboard               | $1.00     | 5   |
| 1     | 2N7000 MOSFET (generic) | $0.80     | 20  |
| 2     | 10K resistors           | $0.70     | 100 |
|       | Sum                     | $7.00     |     |

The extension cable is only required for the female connector. If one is willing
to cut off the cable of their controller and hardwire it into the adapter, it
can be omitted. Shoving solid wire (e.g. Dupont connectors) in the male GC
connector may also work as a short-term solution.

## Wire Protocol

The communication between a Gamecube and its peripherals happens over a
proprietary asynchronous half-duplex serial protocol called "Joybus". The data
line is pulled up internally by the peripheral, pulled down to transfer a bit,
then released again. Stop bits indicate the end of a transmission.

The timings are as following:
- Zero: 3 μs low, 1 μs high
- One: 1 μs low, 3 μs high
- Console stop: 1 μs low, 2 μs high
- Peripheral stop: 2 μs low, 2 μs high

This results in an effective bitrate of 250 kHz at a clock speed of 1 MHz.

The protocol uses a simple request-response model where the console sends a
command and the peripheral answers after reading a stop bit.

The stop bits are commonly misinterpreted as ones, and implementing them as such
may work in practice. However, the console should release the line quickly
after a console stop so the peripheral can answer.

### Connector Pinout

Pin location is given with the flat side pointing up, looking into the
connector.

| *Female*   | Left    | Middle   | Right   |
|------------|---------|----------|---------|
| **Top**    | (1) 5V  | (2) DATA | (3) GND |
| **Bottom** | (4) GND | (5) NC   | (6) 3V3 |

And for quick reference,

| *Male*     | Left    | Middle   | Right   |
|------------|---------|----------|---------|
| **Top**    | (3) GND | (2) DATA | (1) 5V  |
| **Bottom** | (6) 3V3 | (5) NC   | (4) GND |


## Related Projects

- [Shinewave](https://github.com/Serisium/Shinewave), a Gamecube controller USB
  mod based on the ATtiny84, without rumble
- [Gamecube/N64 controller to USB adapter](https://www.raphnet.net/electronique/gcn64_usb_adapter_gen3/index_en.php), based on the ATmega32U2
  - [Earlier revision](https://www.raphnet.net/electronique/gc_n64_usb/index_en.php)
    using the ATmega8/168
- [Adapt-FFB-Joy](https://code.google.com/archive/p/adapt-ffb-joy/), an
  AVR-based USB adapter for the Microsoft SideWinder Force Feedback Pro
  controller
- [DigisparkJoystick](https://github.com/digistump/DigisparkArduinoIntegration/tree/master/libraries/DigisparkJoystick), HID joystick emulation specifically for the Digispark
- [ArduinoXInput_AVR](https://github.com/dmadison/ArduinoXInput_AVR), XInput emulation for AVRs

## Related Information

- [Nintendo Gamecube Controller Protocol (int03)](http://www.int03.co.uk/crema/hardware/gamecube/gc-control.html)
- [Console Protocols: Joybus (Google Sites)](https://sites.google.com/site/consoleprotocols/home/nintendo-joy-bus-documentation)
- [sbarrack/Project-Bongo protocol documentation (Google Cache)](https://webcache.googleusercontent.com/search?q=cache%3Ahttps%3A%2F%2Fgithub.com%2Fsbarrack%2FProject-Bongo%2Fwiki%2FGameCube-N64-Controller)
