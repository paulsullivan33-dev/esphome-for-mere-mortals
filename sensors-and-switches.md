# Sensors and Switches

**Scope:** the greenhouse node taken apart —
the three kinds of citizen in an ESPHome
configuration (sensors, binary sensors,
switches), the specimen's real wiring in
YAML, and the small automation that makes
the device behave instead of merely report.
The physical half of every claim here is
chapter 2; the proof of behaviour is the
simulator run quoted at the end.

## Sensors: numbers with names

A **sensor** reports a number on a
schedule. The greenhouse's centre of
gravity is a BME280 on the I2C bus:

```yaml
i2c:
  sda: GPIO21
  scl: GPIO22
  scan: true

sensor:
  - platform: bme280_i2c
    address: 0x76
    temperature:
      name: "Greenhouse Temperature"
      id: air_temp
      oversampling: 2x
    pressure:
      name: "Greenhouse Pressure"
    humidity:
      name: "Greenhouse Humidity"
    update_interval: 60s
```

One physical chip, three sensors: the
`bme280_i2c` platform knows the part
reports temperature, pressure, and
humidity, and each gets its own entry,
name, and options under the platform.
Details worth reading closely:

- **`address: 0x76`** is where the chip
  answers on the bus — BME280s answer to
  `0x76` or `0x77` depending on one pin.
  The `scan: true` above makes the device
  list what it finds at boot, so a wrong
  address is a fact you learn from the
  log, not a mystery (chapter 12).
- **`oversampling: 2x`** asks the chip to
  measure twice and average — slower,
  steadier numbers. Sensors are full of
  such trade-offs, and ESPHome's style is
  to put them in the file, in words.
- **`update_interval: 60s`** is the
  metronome. Nothing in ESPHome polls
  faster than you say, and saying so is
  part of the design: a greenhouse changes
  slowly, and a sensor read sixty times a
  second would mostly report noise and
  spend power doing it.
- **`id: air_temp`** appears only on the
  temperature, because only the temperature
  is *referred to* — by the automation
  below. Give ids to things other lines
  point at; names are for humans and
  Home Assistant, ids are for the file.

The file's other two sensors are free
citizens of the chip itself: `uptime`
(seconds since boot) and `wifi_signal`
(the radio's received signal strength, in
dBm — a negative number where closer to
zero is stronger). Both are genuinely
useful in the field: the first exposes
reboot loops, the second settles every
"is it the Wi-Fi?" argument with a number.

## Binary sensors: the world in two states

A **binary sensor** reports on or off.
The greenhouse door:

```yaml
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO18
      mode:
        input: true
        pullup: true
      inverted: true
    name: "Greenhouse Door"
    device_class: door
```

The pin block is chapter 2 in uniform:
listening (`input: true`), held at a
definite level by the internal pull-up,
and `inverted: true` because the reed
switch pulls the pin *low* when the door
opens, and the file prefers its truths
the right way up. **`device_class: door`**
is a promise about meaning — it is how the
device says "this on/off is a door, open
or shut," and it is what lets Home
Assistant show a door instead of a bare
toggle. Device classes are vocabulary,
and using the right one is most of what
"integrates well" means.

## Switches: hands

A **switch** is something the firmware can
turn on and off — and, through the API,
something its owner can turn on and off
too. The vent fan:

```yaml
switch:
  - platform: gpio
    pin: GPIO19
    name: "Vent Fan"
    id: vent_fan
    restore_mode: RESTORE_DEFAULT_OFF
```

The platform is the pin itself; in the
physical build this pin drives the relay
of chapter 2. **`restore_mode`** answers
the question every deployed device
eventually asks: *after a power cut, what
should I be?* `RESTORE_DEFAULT_OFF` means
"remember nothing heroic — come back off
unless told otherwise." For a fan that is
a shrug; for a heater or a pump it is a
safety decision, and ESPHome's making you
write it down is the feature.

## The automation: where behaviour lives

Facts in, facts out is a thermometer. The
device earns its keep with one automation:

```yaml
interval:
  - interval: 30s
    then:
      - if:
          condition:
            sensor.in_range:
              id: air_temp
              above: 30.0
          then:
            - switch.turn_on: vent_fan
          else:
            - switch.turn_off: vent_fan
```

Every 30 seconds: if the temperature
sensor (by id) reads above 30, the fan
(by id) on; otherwise off. Automations
in ESPHome are these small declarative
sentences — a **trigger** (`interval`),
**conditions**, **actions** — and the
vocabulary is deliberately plain. Two
honest observations. First, this rule is
crude: a real greenhouse wants hysteresis
(on at 30, off at 27) so the fan doesn't
chatter at the boundary; chapter 10 writes
that version as a recipe. Second, the rule
runs *on the device*. Wi-Fi can die, Home
Assistant can reboot, and the fan still
obeys the air, because the behaviour was
compiled into the chip. That locality is
ESPHome's deepest selling point, and it
costs nothing to use.

## Proof, from the simulator

The host twin (chapter 4) runs this exact
automation against its lambda weather,
which climbs 0.7 degrees every 5 seconds
and wraps from 33 back to 24. Watched
over the API (chapter 8 shows the client;
these are its `[S]` state lines,
transcript t05, trimmed):

```
[02:46:07][S][sensor]: 'Simulated Greenhouse Temperature' >> 28.2 °C
[02:46:07][S][binary_sensor]: 'Simulated Greenhouse Door' >> ON
...
[02:46:22][S][sensor]: 'Simulated Greenhouse Temperature' >> 30.3 °C
[02:46:22][S][switch]: 'Simulated Vent Fan' >> ON
...
[02:46:42][S][sensor]: 'Simulated Greenhouse Temperature' >> 24.0 °C
[02:46:42][S][switch]: 'Simulated Vent Fan' >> OFF
```

The door (in the twin, a lambda that opens
it past 28 degrees) flips first; the fan
follows the moment the air crosses 30,
and releases when the wrap drops the
pretend weather back to 24. That is the
whole specimen working: senses, threshold,
hands — the same three lines of YAML that
the real node validates against in t01.
