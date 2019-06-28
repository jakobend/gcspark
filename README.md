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

## Overview

I decided to choose the [Digispark](http://digistump.com/products/1) Attiny85
development board as the base of this project. Clones of it are affordable and
very easy to source. The integrated USB plug/port simplifies wiring, and using
it as an USB device via [V-USB](https://www.obdev.at/products/vusb/index.html)
is well-documented. Since it is a development board, no ISP is required to
program it.

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
