# Anatomy of a Configuration

**Scope:** one ESPHome configuration, read
top to bottom — what each section declares,
and what happens when ESPHome checks the
file before believing it. The specimen is
`examples/greenhouse.yaml`, and every claim
about validation below is a capture from
running the real tool against it.

## The shape of the file

A configuration is a YAML document whose
top-level keys name **components** — the
building blocks ESPHome will assemble into
firmware. The greenhouse file, in skeleton:

```yaml
esphome:
  name: greenhouse
  friendly_name: Greenhouse

esp32:
  board: esp32dev
  framework:
    type: esp-idf

logger:
api:
ota:
wifi:
captive_portal:
i2c:
sensor:
binary_sensor:
switch:
interval:
```

Read it as a parts list with opinions.
Some components appear once (`wifi:`);
some are *domains* holding a list of
individual things (`sensor:` holds three
sensors in the full file). Each entry
under a domain names a **platform** — which
kind of thing it is — and its settings:

```yaml
sensor:
  - platform: bme280_i2c
    address: 0x76
    temperature:
      name: "Greenhouse Temperature"
      id: air_temp
      oversampling: 2x
    update_interval: 60s
```

Two fields recur everywhere and are worth
learning once. **`name:`** is what the
outside world will call the thing — it is
how the sensor appears in Home Assistant
and in the logs. **`id:`** is the name the
rest of the *file* uses: automations refer
to `air_temp`, never to the friendly name.
Confusing the two is the beginner's most
common knot; chapter 5 unties it in place.

## The sections, in order

**`esphome:`** names the device itself.
`name` becomes its network identity
(`greenhouse.local` on your LAN, via mDNS)
and must be lowercase letters, numbers,
and underscores. `friendly_name` is the
display label. Everything else in the file
belongs to this name.

**The board section** — `esp32:` here —
says what the firmware is *for*. It names
the board (hence the pin map) and the
**framework**, the underlying C++ library
the generated code stands on: `esp-idf`
(Espressif's own) or `arduino` (the hobby
ecosystem's). Other chips have their own
section names (`esp8266:`, `rp2040:`), and
the host build of chapter 1 replaces this
whole section with a bare `host:`. A
configuration targets exactly one board;
moving a project between chips is an edit
here, and usually only here.

**`logger:`** is the device's voice:
serial and network logging, with a level
(`DEBUG`, `INFO`, up through `ERROR`).
Chapter 9 is devoted to it. The greenhouse
runs `INFO`; the simulator runs `DEBUG`,
deliberately, so the guide could watch it
think.

**`api:`** and **`ota:`** are the two
doors back into the device once it is
deployed — the native API it speaks to
Home Assistant and tools (chapter 8), and
over-the-air updating (chapter 7). Both
carry credentials in the full file; both
are fed by `!secret` references rather
than literal values (chapter 6).

**`wifi:`** joins the network, with a
fallback of its own if it can't
(chapter 6). Ethernet exists for some
boards as an `ethernet:` component
instead; Wi-Fi is the common case and
the only one this guide builds.

**`captive_portal:`**, present as a bare
key, enables the little sign-in page the
fallback hotspot serves. A component with
no settings still has to be *named* to
exist — an empty section is a statement,
not an omission.

**`i2c:`** sets up the bus the sensor
rides: which pins are data and clock, and
`scan: true`, which makes the device report
what it finds on the bus at boot — the
single most useful line when a sensor
"doesn't work" (chapter 12).

**The domains** — `sensor:`,
`binary_sensor:`, `switch:` — are the
device's senses and hands; chapter 5.

**`interval:`** is one of ESPHome's
automation engines: do something every so
often. Others exist (`on_...` triggers on
nearly every component). Automations are
where a parts list becomes behaviour.

## Validation: the file is checked before it is believed

None of this reaches a compiler until
ESPHome has checked the file against every
component's schema — every required field,
every pin, every cross-reference between
`id`s. That check is a command, it needs
no hardware, and it is the most useful
habit this guide can teach:

```
$ esphome config greenhouse.yaml
INFO ESPHome 2026.10.0
INFO Reading configuration greenhouse.yaml...
...
INFO Configuration is valid!
```

(The full output — 238 lines, the whole
configuration echoed back as ESPHome
understood it, secrets still shown as
`!secret` references — is transcript t01.)
Run `config` after every edit. It is the
difference between arguing with a compiler
for an afternoon and being told, in
seconds, exactly which line is wrong.

Being told looks like this. The guide
planted two faults in a copy of the file;
validation stopped at the first, an
out-of-date platform name:

```
$ esphome config broken.yaml
INFO ESPHome 2026.10.0
INFO Reading configuration broken.yaml...
INFO Unable to import component bme280.sensor: No module named 'esphome.components.bme280'
Failed config

Platform not found: 'sensor.bme280'
```

(Transcript t06; the command exits with
code 2.) Three lessons ride on those six
lines. Platform names are **exact** — there
is no `bme280` in ESPHome 2026.10, because
the combined component was split into
`bme280_i2c` and `bme280_spi`, so old
examples copied from the internet fail
exactly here. Validation is **complete but
sequential** — it found the platform fault
and stopped; the second planted fault (a
door switch assigned to a pin the I2C bus
already owns) waits behind it, and fixing
the first is how you get to be told about
the second. And the check is **offline**:
no board, no network, nothing but the file
and the tool. Everything chapter 4 does
before the flash rests on that.
