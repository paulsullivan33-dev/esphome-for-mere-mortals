# Writing Criteria

Every guide in this repo is written to the same
standard. If you're adding or editing a guide —
human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language,
   never dumbed-down concepts. Explain jargon on
   first use; collect terms in the glossary.
2. **No hype.** No marketing adjectives, no
   "revolutionary," no promises about what
   something will do for the reader.
3. **Honest about limits.** Say what the thing
   *can't* do. State uncertainty instead of
   smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults,
   and specs are either measured or qualified with
   "depends on your setup." If it wasn't tested,
   say so.
5. **Checkable.** Commands are meant to be run,
   with "Check:" blocks showing what success looks
   like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out
   Windows/Mac/Linux differences. Don't assume a
   GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get
   stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and
   points elsewhere for the rest. No guide
   pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after
   examples, symptom → fix. Scannable beats essay.

## Verification notes for this guide

Standard 3 and 4 applied to this guide,
specifically. There was **no ESP board of
any kind** on the build bench. What ran was
software: the official **ESPHome 2026.10.0**
(pip, in a venv), which validated two
configurations, compiled both of them —
one for a real ESP32 target under ESP-IDF
5.5.6, one for the host platform — and ran
the host build as a program, served its
API to a real client, and pushed a real
OTA update into it. Every quoted output in
the chapters is a capture from those runs;
working transcripts are kept with the build
(`esphome-lab/transcripts/`, t00–t11).

**The lab.** One fictional device, two
configurations, kept verbatim in
`examples/`:

- *The Greenhouse Node* (`greenhouse.yaml`):
  an ESP32 (`esp32dev`, ESP-IDF framework)
  in an invented backyard greenhouse —
  BME280 on I2C at `0x76` (temperature with
  `id: air_temp`, pressure, humidity), a
  reed-switch door on GPIO18, a relay
  switch on GPIO19, and a 30-second
  automation: fan on above 30 °C, off
  otherwise. Validated (t01) and compiled
  for real silicon (t08).
- *The twin* (`greenhouse-sim.yaml`): the
  same names, ids, and automation aimed at
  the `host:` platform, with template
  parts — a lambda raising pretend
  temperature 0.7 °C every 5 s (wrapping
  33 → 24), a door lambda (open past
  28 °C), a template switch for the fan.
  Logger at DEBUG by design. Validated
  (t02), compiled (t03), run (t04, t07,
  t10), and observed through its own API
  by `esphome logs` (t05).

**Verified by running, per chapter:**

- *Ch. 1:* `esphome version` → 2026.10.0
  (t00). The framing facts are conceptual.
- *Ch. 3:* `esphome config greenhouse.yaml`
  in full (t01, 238 lines): the tool echoes
  the configuration as understood, secrets
  still shown as `!secret` references, and
  ends `INFO Configuration is valid!`.
  The planted-fault copy (t06) exits 2 with
  `Platform not found: 'sensor.bme280'` —
  the first planted fault (an outdated
  platform name). The second planted fault
  (door switch on GPIO21, the I2C SDA pin)
  was placed behind it and, validation
  being sequential, was not separately
  adjudicated; the chapters say exactly
  that.
- *Ch. 4:* the install (pip, ESPHome
  2026.10.0); both validations (t01, t02);
  the host compile (t03: 133 objects,
  `LINK program`, success line); the twin's
  boot lines (t04); the ESP32 compile (t08:
  ESP-IDF 5.5.6 framework downloaded by
  PlatformIO, then `firmware.factory.bin`
  926,480 bytes, `firmware.ota.bin`
  860,944 bytes, `firmware.elf`, and the
  success line). Also probed, not
  transcribed: `esphome dashboard` answers
  that the built-in dashboard has been
  removed in favour of the separate ESPHome
  Device Builder tool; chapter 4 reports
  its wording.
- *Ch. 5:* the whole specimen behaviour,
  over the API (t05): temperature climbing
  25.4 → 30.3 °C in 0.7 steps every 5 s;
  the door ON at 28.2 °C; the **fan ON at
  30.3 °C** (the interval automation,
  unassisted); the wrap to 24.0 °C; door
  OFF; **fan OFF**; the cycle repeating.
  The greenhouse.yaml text quoted is the
  file that validated in t01 and compiled
  in t08, byte for byte.
- *Ch. 6:* the secrets mechanism — the lab
  built and validated throughout against a
  real `secrets.yaml` that is not in this
  repository; t01 proves the echo behaviour
  (`!secret` unresolved in output); t11 is
  the control experiment: the same
  configuration in a folder without
  `secrets.yaml` fails validation, exit 2,
  naming the missing file. The invented
  household's credentials (Wi-Fi name
  "Marigold Cottage WiFi", and passwords)
  appear in chapter 6 as specimens, are
  fictitious, and protect nothing.
- *Ch. 7:* the OTA service in the twin's
  boot dump (t04: address port 8082,
  version 2, "Encryption: offered,
  plaintext accepted", "Password
  configured"); a real update (t09:
  `esphome upload`, 989,608-byte image
  compressed to 314,010, encrypted
  handshake, `OTA successful`, total
  0.65 s); the device's side of it (t10:
  Noise handshake, staging file, progress,
  inflation to full size); safe mode's
  terms from the dump (t04: successful
  after 60 s, invoke after 10 attempts,
  duration 300 s) and its healthy counter
  line observed live (t07: `Boot seems
  successful; resetting boot loop
  counter`). The microcontroller OTA port
  (3232) is the project's documented
  default, not a lab observation.
- *Ch. 8:* the API server in the dump
  (t04: port 6053, "Noise encryption:
  YES", 8 max connections); a real client
  session (t05: resolved, connected in
  0.005 s, handshake 0.479 s, configuration
  replayed, then state streamed); the
  device's record of the client (t07).
  Home Assistant pairing is *not* in this
  list — see Documented-only.
- *Ch. 9:* the boot sequence and warning
  flag (t04); DEBUG's configuration dump
  (t02, t04); the client's state stream
  (t05); the client's abrupt-departure
  warning, errno 11 (t07); the debugging
  order is method, drawn from the above.
- *Ch. 10:* recipe 1 is t02+t03+t05 as a
  procedure. Recipe 2's hysteresis shape
  assembles chapter 5's verified
  `sensor.in_range` conditions into two
  one-way rules; that exact two-rule file
  was not itself compiled, and the recipe
  says it is assembled. Recipe 3's sensor
  and stream are t05; its controller steps
  are chapter 8's documented material.
  Recipe 4 was followed by this
  repository's own construction.
- *Ch. 11:* the host-platform observations
  are t04/t05/t07 (quiet standalone, client
  stream, the borrowed uptime). The
  release-cadence and breaking-change
  claims are the project's public
  behaviour, with t06 as a local fossil.
- *Ch. 12:* the lab entries cite t04–t07
  and t09–t11 in place; the bus and
  first-flash entries are labelled in
  their own text as documented shapes, not
  captures.

**Two honest negatives from the bench:**

- *The quiet twin.* Run standalone, the
  host program printed its boot and
  configuration dump, then produced no
  periodic state output across 30 s of
  watching; its CPU time did not advance.
  With an API client connected, the same
  program streamed state every 5 s. The
  guide reports the observation and the
  practical lesson (judge by a client's
  view), and does not claim a cause: the
  sandbox may be part of the story.
- *The borrowed uptime.* The twin's uptime
  sensor reported 3,328 s at its first
  reading and 3,436 s later — the build
  machine's uptime, the host platform
  having no device clock of its own to
  report. Quoted in chapters 11 and 12 as
  the kind of difference a simulator
  legitimately has.

**Capture hygiene (declared trims):**
chapter quotations trim long outputs with
`...` (the compile's middle, the config
echo's bulk, the state stream's
repetitions) and shorten two absolute
paths to `.../program`; transcripts
themselves are the full captures, with
terminal colour codes stripped by `sed`
and nothing else edited, reordered, or
cleaned up. No secret needed redacting in
any transcript: `esphome config` echoes
`!secret` references unresolved (t01), and
no run printed a credential — the lab's
API key, generated for the lab with
`openssl rand -base64 32`, exists only in
the lab's own `secrets.yaml`, outside this
repository. The invented passwords printed
in chapter 6 are specimens, declared there.

**Build-machine weather, stated once:** the
lab ran in a sandbox whose clock reads UTC
(the transcripts' 02:4x timestamps are
21:4x CDT, 2026-10-21). Loopback TCP — all
the lab's networking — worked; mDNS name
resolution was not exercised (clients
addressed the twin as 127.0.0.1), and no
claim in the guide rests on it.

**Documented-only (not run), and why:**

- *Flashing a real device:* no hardware.
  The USB first flash, `esphome run`, port
  names, BOOT-button and driver wrinkles
  are described from the project's
  documentation. The ESP32 *compile* is
  verified (t08); only the write to
  silicon is not.
- *Home Assistant pairing and use:* no
  Home Assistant instance was run in the
  lab. The device side of the conversation
  — discovery data, encrypted API,
  entity names and device classes — is
  verified through ESPHome's own client
  (t05); the controller side (discovery
  notification, integration setup, key
  entry, dashboards) is described from
  the two projects' documentation.
- *Real sensors and radios:* the BME280,
  reed switch, and relay were never
  wired; Wi-Fi association, roaming,
  range, and the fallback hotspot on a
  phone are physical-world behaviour,
  described from documentation. Every
  sensor value in the guide is the twin's
  lambda arithmetic, labelled simulated
  wherever it appears.
- *The I2C scan on real hardware, address
  selection, and pin conflicts in silicon:*
  the `scan:` setting and `0x76` address
  validated (t01); their on-hardware
  effects are chapter 12's documented
  shapes.
- *The dashboard / ESPHome Device
  Builder:* not installed. Its existence
  and the old command's refusal message
  were observed (chapter 4); its use is
  described, not walked.
- *MQTT, the web server component,
  Ethernet, deep sleep, Bluetooth:* named
  where they bound the subject, described
  from documentation, never enabled in
  the lab's configurations.
- *Chapter 2's electrical facts* (pin
  behaviour, pull-ups, chip variants) are
  the manufacturers' and project's
  published material. The one safety line
  there is a boundary statement, not
  instruction.

Specimen discipline, for the record: the
greenhouse, its cottage, and its network
are invented; no real household,
organisation, or deployment is described
anywhere in the guide. The lab's one real
secret (the generated API key) never
entered this repository, and no command in
this guide was aimed at any system outside
the lab directory and its own processes.
