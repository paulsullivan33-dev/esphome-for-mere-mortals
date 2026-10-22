# The Hardware

**Scope:** the chips ESPHome targets, the
boards they arrive on, and the electrical
vocabulary — pins, buses, pull-ups — that
the configuration file speaks. This chapter
is background for chapter 3; nothing here
was on the build bench, and the electrical
facts are the chips' published ones.

## The chips

ESPHome began with Espressif's two Wi-Fi
microcontroller families, and they are
still its centre of gravity:

| Chip | Character | Reach for it when |
| --- | --- | --- |
| **ESP32** (original) | Two cores, Wi-Fi and Bluetooth, plenty of pins | The default choice; the greenhouse specimen targets it |
| **ESP32-S2 / S3 / C3 / C6** | Newer variants: cheaper, lower power, or extra radios (the C6 adds Thread/Zigbee) | A named board uses one; buy the board, not the letter |
| **ESP8266** | The older, smaller sibling: one core, fewer pins, less memory | A device you already own; new projects usually start with ESP32 |

A **microcontroller** is a whole small
computer — processor, memory, and the
circuits that talk to pins — on one chip,
with no operating system and no screen.
It runs one program from the moment power
arrives until power leaves. That is all
the greenhouse node is: a program with
pins.

ESPHome also supports other platforms —
RP2040 and RP2350 (the Raspberry Pi Pico
family), BK72xx and RTL87xx chips found
inside many commercial smart plugs and
bulbs, and a few more. The configuration
shape is the same for all of them; only
the board section at the top changes
(chapter 3). And there is one platform
that is not a chip at all: **host**, which
builds the firmware as a program for your
computer. Chapter 1 explained why this
guide leans on it; chapter 11 is honest
about how far that leaning reaches.

## Boards

Chips reach mortals as **development
boards**: the chip on a small circuit
board with a USB socket, a voltage
regulator, and its pins fanned out to
labelled holes. Names to recognise:
the generic `esp32dev` (the greenhouse
configuration's board — a whole class of
near-identical boards), Adafruit's and
SparkFun's named boards, the ESP32
variants inside finished products like
smart plugs and the official ESP32
development kits. The `board:` line in a
configuration must name a board ESPHome
knows, because the name carries the pin
map: which numbered hole is which
**GPIO** — general-purpose input/output,
the pins you can ask to sense or drive a
signal.

Two practical warnings belong here:

- **Pin labels lie by omission.** Boards
  print friendly names (D1, D2…) over the
  chip's GPIO numbers, and different boards
  reuse the same friendly name for
  different GPIOs. Configurations speak
  GPIO numbers (`GPIO21`), and the board's
  own documentation is the only reliable
  translation.
- **Some pins are spoken for.** Strapping
  pins influence how the chip boots; the
  USB serial pins are how you flash and
  read logs. A configuration that repurposes
  them casually makes a device that is hard
  to talk back into sense. The project's
  pin tables mark the safe defaults.

## The electrical vocabulary

Four ideas carry nearly every ESPHome
project, and chapter 5 assumes them:

- **GPIO as input.** A pin can *listen*.
  The greenhouse door is a **reed switch**
  — a contact that closes when a magnet
  (on the door) comes near. Wired between
  a pin and ground, it gives the pin two
  states: door shut, door not shut.
- **Pull-up.** A listening pin left
  unconnected drifts with electrical noise
  and reports nonsense. A pull-up resistor
  holds it at a definite level until the
  switch overrules it. ESP chips have
  pull-ups built in; the configuration
  asks for one with `pullup: true`, and
  the door switch is then read *inverted* —
  the switch closing pulls the pin low,
  and `inverted: true` makes low mean
  "door open" in the way the wiring was
  drawn. Chapter 5 shows the lines.
- **GPIO as output, and relays.** A pin can
  *speak*, but only in whispers: a few
  milliamps at 3.3 volts. To control
  anything real — a fan, a pump, a lamp —
  the pin drives a **relay**, a switch
  flipped by a small electromagnet, whose
  other side can carry the load. Relay
  boards exist precisely to sit between a
  pin and the world.
- **Buses.** Smarter sensors don't use a
  pin per fact; they hold a conversation.
  **I2C** is the common two-wire bus
  (a data line, SDA, and a clock line,
  SCL) on which several sensors can share
  the same two pins, each answering to its
  own **address** — the greenhouse BME280
  answers to `0x76`. SPI is the faster,
  more-pins cousin. Chapter 5's sensor is
  an I2C citizen; that is the modern shape
  of "wiring a sensor": power, ground, and
  the bus.

**The safety line, stated once and
plainly.** Pins and sensors live at
3.3 volts and are safe to learn on. The
far side of a relay may be mains voltage,
and mains wiring is a different subject,
with its own rules and its own qualified
people. This guide's relay switches a
fictional fan; nothing here instructs
anyone to open a wall box.
