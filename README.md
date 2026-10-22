# ESPHome for Mere Mortals

Somewhere in your house there is a job a
fifty-cent computer chip could be doing:
watching the greenhouse temperature,
noticing whether the door is shut, turning
a fan on when the air gets too warm.
**ESPHome** is the open-source tool that
turns that idea into working electronics
without asking you to become a firmware
programmer first. You describe the device
you want in one plain YAML file — which
board, which sensor, which pin, what should
happen when — and ESPHome writes the C++
program, compiles it, and puts it on the
chip. This guide explains that exchange
honestly: what the YAML really is, what the
chip really does with it, where the secrets
live, how updates reach a device sealed in
a box, and where the whole approach runs
out of road.

The lab is unusual, and the guide is
upfront about it: there was no ESP32 on the
build bench. Instead the guide's specimen —
an invented **Greenhouse Node**, a BME280
watching the air, a reed switch on the
door, a relay for the vent fan — was
written as a real ESPHome configuration,
validated and compiled for its real ESP32
target with the real tool, and then built
a second time for ESPHome's **host
platform**, where the same firmware logic
compiles into an ordinary program on a
computer. That program was run. It booted,
served its encrypted API to a real client,
streamed its pretend temperatures, and —
when the air crossed 30 degrees — turned
its fan on, exactly as the configuration
says. It also accepted a real over-the-air
update. What never happened is electricity
meeting silicon: no board was flashed, no
real sensor was read, and the chapters say
so where it matters. `criteria.md` keeps
the full ledger of what was run and what
was only read.

## What this guide covers

- [**What ESPHome is**](what-esphome-is.md) —
  the mental model: YAML in, firmware out;
  what ESPHome is not; the lab
- [**The hardware**](the-hardware.md) —
  the ESP32 and ESP8266 families, the other
  platforms, pins and power, and what a
  sensor on a bus actually is
- [**Anatomy of a configuration**](anatomy-of-a-configuration.md) —
  the sections of the file, top to bottom;
  validation, and what a validation error
  is really telling you
- [**Your first device**](your-first-device.md) —
  installing ESPHome, writing the greenhouse
  configuration, proving it valid, and the
  one step the lab could not take: the flash
- [**Sensors and switches**](sensors-and-switches.md) —
  the specimen in detail: a BME280 on I2C,
  a reed switch on a pin, a relay, and the
  small automation that ties them together
- [**Wi-Fi and secrets**](wifi-and-secrets.md) —
  how a device joins your network, the
  fallback hotspot, and the `secrets.yaml`
  bargain, stated honestly
- [**Updates over the air**](updates-over-the-air.md) —
  OTA: changing a device you no longer have
  to touch, safe mode, and the update this
  guide actually performed
- [**The native API and Home Assistant**](the-native-api-and-home-assistant.md) —
  how the device talks to its owner:
  an encrypted protocol, discovery, and
  the pairing Home Assistant readers came for
- [**Logs and debugging**](logs-and-debugging.md) —
  log levels, reading a device's mind over
  the API, and debugging the configuration
  before it ever reaches a chip
- [**Recipes**](recipes.md) — the five-minute
  logic check, the too-hot fan, a door that
  reports itself, and a configuration you
  can safely share
- [**Limits and honest edges**](limits-and-honest-edges.md) —
  what ESPHome can't do, what it shouldn't
  be asked to do, and the costs of a device
  whose whole personality is one file
- [**Troubleshooting**](troubleshooting.md) —
  the platform that isn't found, the silent
  simulator, the warning flags, and the
  uptime that belongs to the wrong machine
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Writing your own ESPHome components in C++,
ESPHome's display and touchscreen machinery,
voice assistants, Bluetooth proxies in
depth, and the long tail of supported
sensors beyond the patterns chapter 5
teaches. Soldering and circuit design
belong to the electronics guides; this one
assumes you can wire a sensor to the pins
the datasheet names. Home Assistant itself
gets one chapter's worth of pairing, not a
guide of its own. And nothing here is
electrical-safety advice: a relay that can
switch a fan can switch things that bite,
and chapter 2 says where that line is.

## How to use this guide

Read chapter 1 for the model and chapter 3
before writing anything; chapter 4 walks
the whole first-device path end to end.
Chapters 5 to 8 are the reference to return
to while a real project is on the bench.
Chapter 9 is where to live when something
misbehaves, with chapter 12 as its index of
symptoms. The configurations the chapters
quote live in `examples/`; the transcripts
behind every claim live in
`esphome-lab/transcripts/` (t00–t11), and
the lab's own notes are
`examples/lab-notes.txt`.
