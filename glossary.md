# Glossary

**Scope:** the terms, collected in one
place. Chapter references point at the
fullest explanation.

**API (native API)** — ESPHome's own
binary protocol between a device and its
controllers: state out, commands in,
optionally encrypted. Chapter 8.

**Binary sensor** — a component reporting
one of two states: on or off, open or
shut, present or absent. Chapter 5.

**Board** — the named development board a
configuration targets; the name carries
its pin map. Chapter 2.

**Boot loop** — a device crashing and
restarting forever; the failure safe mode
exists to interrupt. Chapter 7.

**Captive portal** — the small sign-in
page a device serves on its fallback
hotspot, for handing it new Wi-Fi
credentials. Chapter 6.

**Component** — one building block of a
configuration: a top-level section like
`wifi:`, or one entry under a domain.
Chapter 3.

**Device class** — the declared *meaning*
of an entity (`door`, `temperature`), as
opposed to its type; what controllers use
to present it correctly. Chapter 5.

**Domain** — a top-level section holding
a list of individual things: `sensor:`,
`switch:`, `binary_sensor:`. Chapter 3.

**Fallback AP** — the device's own
emergency hotspot, started when it cannot
join the configured network. Chapter 6.

**Firmware** — the compiled program the
chip runs; in ESPHome, one configuration
compiles to exactly one firmware.
Chapter 1.

**Framework** — the C++ library the
generated code stands on: `esp-idf` or
`arduino` on ESP32. Chapter 3.

**GPIO** — general-purpose input/output:
the chip's pins, listenable or drivable.
Chapter 2.

**Host platform** — ESPHome's build target
that compiles firmware into an ordinary
program for your computer, for testing
logic without hardware. Chapters 1 and 4.

**Hysteresis** — two thresholds instead of
one, so a control doesn't chatter at the
boundary. Chapter 10.

**I2C** — the two-wire bus (SDA data, SCL
clock) on which addressed sensors share
pins. Chapters 2 and 5.

**id** — the name the configuration file
itself uses to refer to a thing, as
opposed to its display `name`. Chapter 3.

**Lambda** — a snippet of real C++ inside
the YAML; ESPHome's escape hatch and its
power tool. Chapters 1 and 5.

**Logger** — the component that produces
the device's log, with its level.
Chapter 9.

**mDNS** — the `.local` name system that
lets a device be reached as
`greenhouse.local` without a recorded
address. Chapters 3 and 6.

**Noise protocol** — the encryption
framework behind the API's and OTA's
encrypted connections. Chapters 7 and 8.

**OTA (over-the-air update)** — replacing
a device's firmware over the network,
after the first cable flash. Chapter 7.

**Oversampling** — a sensor measuring
several times and averaging, for steadier
numbers. Chapter 5.

**Platform** — two senses, kept straight
by context: the *kind* of a domain entry
(`platform: bme280_i2c`), and the build
target family (ESP32, host). Chapters 3
and 2.

**PlatformIO** — the build system ESPHome
drives behind the scenes to compile the
generated C++. Chapter 1.

**Pull-up** — a resistor (often the chip's
own) holding a listening pin at a definite
level until a switch overrules it.
Chapter 2.

**Reed switch** — a contact closed by a
nearby magnet; the classic door sensor.
Chapters 2 and 5.

**Relay** — a small electrically operated
switch; the pin whispers, the relay
carries the load. Chapter 2.

**Restore mode** — a switch's declared
behaviour after power returns. Chapter 5.

**Safe mode** — the minimal fallback
firmware a device boots after repeated
failed boots: Wi-Fi, logging, and OTA,
waiting for a corrected update.
Chapter 7.

**secrets.yaml** — the file beside a
configuration holding every value its
`!secret` references point at. Chapter 6.

**Sensor** — a component reporting a
number on a schedule. Chapter 5.

**Switch** — a component the firmware,
and through the API its owner, can turn
on and off. Chapter 5.

**Template** — the family of components
(`template` sensor, switch, and kin) whose
values come from lambdas instead of
hardware; the simulator's entire cast.
Chapter 5.

**Validation** — ESPHome's offline check
of a configuration against every
component's schema: `esphome config`.
Chapter 3.

**YAML** — the indentation-based text
format configurations are written in.
Chapter 3.
