# The Native API and Home Assistant

**Scope:** the conversation a device has
with its owner — ESPHome's native API, its
encryption, how a client listens, and the
pairing with Home Assistant that most
readers are here for. The API half of this
chapter was exercised for real, against
the lab twin, with ESPHome's own client.
The Home Assistant half is described
from the projects' documentation: no Home
Assistant was running in the lab, and the
chapter does not pretend otherwise.

## One protocol, two jobs

The `api:` section of chapter 3 opens a
server on the device (TCP port 6053) that
speaks ESPHome's **native API** — a binary
protocol designed for exactly this: a
controller connecting, learning what
entities the device has, receiving their
state changes as they happen, and sending
commands back (turn the fan on; the file's
switches and much else are controllable
this way). It is also the channel `esphome
logs` uses to stream a device's log —
chapter 9 lives on it.

The greenhouse configuration's version of
the section is two meaningful lines:

```yaml
api:
  encryption:
    key: !secret api_key
```

That key turns on the protocol's
encryption — the **Noise** protocol
framework (the configuration dump calls
the result "Noise encryption: YES"), with
the base64 key from chapter 4 as the
shared secret both sides must hold. A
client without the key can connect and
be told, in effect, *no*. The honest
framing: this is good, modern encryption
of the same family other local protocols
use, and its strength is the key's
custody — one key per device is the
hygiene the projects recommend, because a
shared key turns one lost file into every
device's problem.

## A client at work

The twin's boot dump advertises the
service (transcript t04, trimmed):

```
[C][api:368]: Server:
  Address: greenhouse-sim.local:6053
  Noise encryption: YES
```

And this is `esphome logs`, ESPHome's own
API client, connecting to it (t05):

```
$ esphome logs greenhouse-sim.yaml --device 127.0.0.1
INFO Starting log output from 127.0.0.1 using esphome API
INFO Successfully connected to greenhouse-sim @ 127.0.0.1 in 0.005s
INFO Successful handshake with greenhouse-sim @ 127.0.0.1 in 0.479s
```

(The lab addressed the twin by IP because
the sandbox's name service was not asked
to play; on a home network the mDNS name
from chapter 3 is the usual address.)
Handshake done — encrypted, key accepted —
the device first replays its configuration
dump, then streams every state change as
it happens, which is where chapter 5's fan
proof came from. The device's own log of
the same arrival is one dry line (t07):

```
[D][api.connection:2816]: ESPHome Logs 2026.10.0 (127.0.0.1): connected
```

Two details in the client transcript
deserve a beginner's attention. The
handshake *time* is printed — about half
a second here — because on real hardware
it is the first number that tells you a
network is struggling. And the client is
identified to the device by name and
version, because the API is a conversation
between named parties, not a broadcast:
the device serves a handful of clients at
a time (the dump says 8, with a listen
backlog of 4) and knows who each is.

## Home Assistant, from here

Home Assistant is the API's most important
client, and the pairing — described, not
performed, in this guide — works like
this. A correctly built device announces
itself on the network by mDNS. Home
Assistant, listening, discovers it and
offers a notification: *new device found*.
Accepting takes you to the ESPHome
integration's setup, which asks for the
device's address and, because the
greenhouse configuration encrypts, its
**API encryption key** — the same one in
your `secrets.yaml`, which is the moment
chapter 6's filing system pays rent.
Key accepted, every entity in the
configuration appears in Home Assistant
with its name and device class already
attached: Greenhouse Temperature as a
temperature, the door as a door, the fan
as a switch you can flip from a dashboard
or an automation.

Three honest properties of that marriage:

- **It is local.** No cloud account sits in
  the path; the conversation above is the
  whole architecture. Either side being
  down does not stop the device's own
  automations (chapter 5), which run on
  the chip regardless.
- **The configuration is the contract.**
  Rename a sensor in the YAML and you have
  renamed it for Home Assistant at the
  next update; the entities are the file,
  projected. There is no second place
  where the device is defined, which is
  the blessing and, when a rename breaks
  someone's dashboard, the curse.
- **MQTT is the alternative, not a
  requirement.** ESPHome can publish to an
  MQTT broker instead of (or as well as)
  the API, for owners whose controller is
  not Home Assistant. The specimen doesn't:
  one well-lit path, honestly walked, is
  this guide's way.
