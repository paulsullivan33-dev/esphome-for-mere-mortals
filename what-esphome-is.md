# What ESPHome Is

**Scope:** the mental model — what you hand
ESPHome, what it hands back, and where it
sits among the other ways of making a small
chip do a job. No commands yet except the
one that proves the tool exists; chapter 3
opens the file itself up.

## The exchange

ESPHome is a program that runs on your
computer. You give it a **configuration
file** — a text file in YAML format that
describes a device:

- which chip it runs on,
- how it joins your Wi-Fi,
- what is wired to it (a temperature
  sensor, a door switch, a relay),
- and what it should do (report the
  temperature every minute; turn the fan
  on when the air passes 30 degrees).

ESPHome reads that file and does three
things with it. It **validates** it —
checking every section, name, and pin
against what the real components accept.
It **generates** a complete C++ program
(the *firmware*) from your description.
And it hands that program to a compiler
(PlatformIO, working behind the scenes)
that turns it into a binary the chip can
run. The first time, you copy that binary
onto the chip over a USB cable. Every time
after that, the chip can update itself over
the air — chapter 7.

So the honest summary is: **you are a
firmware author who writes in YAML.** The
YAML is not a settings file for somebody
else's program. It *is* the program,
written at a level where "a BME280 sensor
on the I2C bus, read once a minute" is one
short paragraph instead of four hundred
lines of C++ you would rather not debug.

## What it is not

- **Not an operating system.** There is no
  shell on the chip, no logging in, no
  installing packages later. The firmware
  is one fixed program: your configuration,
  compiled. Changing anything means
  compiling and uploading again — which
  ESPHome makes cheap, but never free.
- **Not a cloud.** A device can live its
  whole life talking only to your own
  network. ESPHome itself is free and
  open-source (it lives under the Open Home
  Foundation, alongside Home Assistant),
  and nothing in the chain phones home.
- **Not just for Home Assistant.** The
  pairing is famous and chapter 8 covers
  it, but a device can also publish to
  MQTT, answer over its own small web
  server, or simply do its automation
  alone in the dark. Home Assistant is the
  most common owner, not a required one.
- **Not no-code.** It is *less* code, at a
  friendlier level. The moment a project
  outgrows the built-in components, the
  escape hatch is a **lambda** — a snippet
  of real C++ inside the YAML. This guide's
  simulator uses one, openly, in chapter 5.

## The tool, proven present

Everything in this guide was done with
ESPHome **2026.10.0**, installed in a Python
virtual environment (chapter 4 shows the
install). The version is the whole
personality of the tool — components are
renamed and split between releases, as
chapter 3's captured error demonstrates —
so the guide states it once, here:

```
$ esphome version
Version: 2026.10.0
```

Check: your version will differ, and that
is fine. What matters is that when a
chapter's example disagrees with your
machine, the difference is usually the
version — ESPHome releases monthly and is
not shy about breaking changes, a cost
chapter 11 returns to.

## The lab, introduced

The specimen is the **Greenhouse Node**: a
fictional ESP32 in an invented backyard
greenhouse, with a BME280 sensor (air
temperature, humidity, pressure), a reed
switch on the door, and a relay that runs
a vent fan. Its configuration is a real
file, kept at `examples/greenhouse.yaml`,
written to be flashed to a real board.

There was no board. So the same logic was
also written as `examples/greenhouse-sim.yaml`,
targeting ESPHome's **host platform** —
the firmware compiled as an ordinary
computer program, with a lambda pretending
to be the weather. That program ran on the
build machine: it booted, listened on its
API and update ports, streamed its state
to a connected client, worked its fan
automation through a full cycle, and
accepted an over-the-air update. Those
runs are this guide's transcripts,
`esphome-lab/transcripts/` t00–t10.

The division of honesty for the whole
guide follows from that: everything about
the *file* — its structure, its validation,
its compilation, its logic — is verified.
Everything about the *physical world* —
radio behaviour, a real sensor's readings,
the flash itself — is described from the
project's documentation and labelled as
such. `criteria.md` keeps the ledger.
