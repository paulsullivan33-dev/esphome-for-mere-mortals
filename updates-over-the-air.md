# Updates Over the Air

**Scope:** how a device whose whole
personality is a compiled file gets a new
personality without a cable — the OTA
component, the update this guide actually
performed, and the safe-mode machinery
that exists because updates can go wrong.

## Why OTA is the normal case

A deployed device is inconvenient by
definition: screwed to a greenhouse frame,
behind a panel, at the top of a pole. If
every fix meant fetching it and finding a
USB cable, nobody would ever fix anything.
So ESPHome's workflow is cable *once* —
the first flash, chapter 4 — and network
forever after. The `ota:` section of the
configuration is what makes the device
listen for its own replacement:

```yaml
ota:
  - platform: esphome
    password: !secret ota_password
```

Two lines, three decisions already made:
the ESPHome OTA protocol (there are other
platforms, for Arduino-style and web-based
updating), a password required before the
device will accept firmware from anyone,
and that password arriving by `!secret`
like every other credential in the file.

## An update, performed

This one is not description — the lab did
it. The host-platform twin (chapters 1, 4)
runs the same OTA component, and its boot
dump advertises the service (transcript
t04, trimmed):

```
[C][esphome.ota:129]: Over-The-Air updates:
  Address: greenhouse-sim.local:8082
  Version: 2
  Encryption: offered, plaintext accepted
[C][esphome.ota:153]:   Password configured
```

(On real ESP32/ESP8266 hardware the OTA
port is 3232; the host build uses 8082.
Note the honest wording of that middle
line — encryption is *offered*, and
plaintext is *accepted*, which is why the
password matters and why chapter 6 insists
it be a real one.) With the twin running,
the update command is the same one used
for hardware:

```
$ esphome upload greenhouse-sim.yaml --device 127.0.0.1
INFO Connecting to 127.0.0.1 port 8082...
INFO Connected to 127.0.0.1
INFO Uploading .../program (989608 bytes)
INFO Connection established; device supports OTA version 2
INFO Encrypted connection established
INFO Compressed to 314010 bytes (deflate)
INFO Handshake complete
INFO Upload took 0.56 seconds, waiting for result...
INFO OTA successful
INFO Successfully uploaded program.
```

(Transcript t09.) And the device's own
view of the same minute (t10):

```
[D][esphome.ota:732]: Starting handshake from 127.0.0.1
[D][esphome.ota:146]: Noise handshake complete
[D][esphome.ota:732]: Starting update from 127.0.0.1
[D][ota:233]: OTA begin: staging=...program.ota.new, size=989608
[D][esphome.ota:865]: Progress: 0.3%
[D][esphome.ota:088]: Inflated 989608 bytes from 314010
```

Read the pair together and the anatomy is
plain: an encrypted handshake (the same
Noise protocol as the API, chapter 8), a
password check inside it, the image sent
compressed, staged under a temporary name,
and committed only once whole — the device
never runs a half-arrived firmware. On the
twin the numbers are a computer program's
(989,608 bytes); a real ESP32 image is a
different size but the same choreography.

## Safe mode: the plan for the bad day

An update can still strand a device — a
configuration that validates, compiles,
and then crashes the chip at boot, over
and over. ESPHome's answer is baked into
every build, and the twin's dump states
its terms (t04):

```
[C][safe_mode:058]: Safe Mode:
  Successful after: 60s
  Invoke after: 10 attempts
  Duration: 300s
```

The logic: if the device fails to stay up
repeatedly — ten unsuccessful boots — it
stops trying to be itself and boots
**safe mode** instead: a minimal firmware
with Wi-Fi, logging, and OTA, and none of
your configuration's ambitions. It waits
there (five minutes by default) for the
one thing that can fix it: a corrected
update, pushed over the air. And the
counter is real — during the lab's API
session the twin, healthy, recorded the
other side of the bargain (t07):

```
[I][safe_mode:154]: Boot seems successful; resetting boot loop counter
```

Sixty seconds of successful running
cleared the slate. A boot loop is not a
brick; it is a device asking, stubbornly
and by design, to be sent better firmware.
What safe mode cannot survive is a build
broken *below* the configuration level —
bad board definitions, power problems —
and the final rung of the ladder is the
one from chapter 4: the USB cable, which
asks nothing of the firmware at all.

**Discipline worth copying.** Because the
update path is this smooth, the failures
that remain are self-inflicted and
sudden: pushing to the wrong device,
editing the API or Wi-Fi sections into a
shape that locks you out. ESPHome lets
you — it is your firmware. The standing
habit of careful builders is boring and
effective: change one section at a time,
keep the last known-good configuration
file, and never edit the door you are
standing in front of — Wi-Fi, API, OTA —
in the same build as anything else.
