# Logs and Debugging

**Scope:** hearing what a device thinks —
the logger, its levels, the two ways to
listen (its own output and the API), and
the order in which a careful builder
debugs: file first, logic second, hardware
last. Everything quoted is from the lab.

## The device talks constantly

ESPHome firmware is chatty by design.
Every component introduces itself at boot,
every state change can be a line, and the
chatter is graded by **level**:

| Level | What it carries | Use it when |
| --- | --- | --- |
| `ERROR` | Things that failed | Something is already wrong |
| `WARN` | Things that smell wrong | Watching a suspicion |
| `INFO` | Boots, connections, the shape of a normal day | Default for deployed devices |
| `DEBUG` | State sends, internal decisions, configuration dumps | Building and debugging |
| `VERY_VERBOSE` | Everything, including the sausage being made | The documentation tells you to |

The greenhouse runs `INFO`. The simulator
runs `DEBUG`, set in its file for exactly
one reason: this guide needed to watch it
think. That pairing is the recommended
posture — quiet in production, loud on
the bench, and the difference is one line
you change before the "real" build.

A boot at INFO already tells a story
(t04, from the twin's own standard output):

```
[I][app:073]: Running through setup()
[W][component:290]: api set Warning flag: waiting for client connection
[I][app:130]: setup() finished successfully!
[I][app:164]: ESPHome version 2026.10.0 compiled on 2026-10-22 02:39:52 +0000
```

Read it the way the firmware means it:
setup ran; a component flagged a warning —
the API noting, at boot, that no client
has connected yet, which on a healthy
device is a weather report, not a fault;
setup finished; the version line closes
the boot like a signature. At DEBUG the
same boot continues into the full
configuration dump — every component
restating its settings as it understood
them — which is the single most useful
debugging document the device produces:
when behaviour surprises you, the dump is
what the device *actually* built, as
opposed to what you remember writing.

## Two ways to listen

**Its own output.** A device on USB prints
its log to the serial line, and a host
build prints it to your terminal — the
twin's transcripts t04, t07, and t10 are
exactly that stream, captured. This is
the ground truth, and it needs no network
and no key.

**Over the API.** `esphome logs <config>`
connects as a client (chapter 8) and
streams the same lines, prefixed by the
client with their arrival times. Its
highest use is state-watching: the `[S]`
lines of chapter 5 are the client
rendering the device's state traffic.
One behavioural note from the lab, stated
as observed: the twin, left alone, printed
its boot and dump and then sat quiet —
its periodic updates were seen by the
*connected client*, streaming steadily
(t05), rather than accumulating in the
standalone output. Whether that is the
host platform's scheduling or this
sandbox's nature, the practical lesson
survives either way: judge a device's
health by a client's view of it, because
that is the view its owner will have.

The device's log of being watched is
worth a glance too (t07): it records the
client's arrival by name, and — when the
client was stopped mid-session — its
departure:

```
[W][api.connection:2822]: ESPHome Logs 2026.10.0 (127.0.0.1): Reading failed CONNECTION_CLOSED errno=11
```

A warning, because the goodbye was
abrupt; a non-event, because the client
had simply been put away. Chapter 12
keeps it so you don't chase it at midnight.

## The debugging order

Everything in this guide points the same
way: **most ESPHome trouble is in the
file, and the file can be interrogated
without the device.** The order that
falls out of the lab:

1. **`esphome config` first.** Chapter 3's
   discipline. A platform name from an old
   example (`sensor.bme280`, t06) never
   reaches hardware at all — and neither
   do missing secrets, bad pins, or
   dangling `id`s. If it validates, the
   file is at least *grammatical*.
2. **The twin second.** If the question is
   *behaviour* — will the fan fire, does
   the door read the right way up — the
   host build answers it in a minute, on
   your desk, with DEBUG on and no ladder
   required (chapters 4 and 5).
3. **The dump third.** Device misbehaving
   in the field? Read its boot dump
   against your file. The component that
   is missing from the dump is the line
   that didn't make it into the build.
4. **The network last, and by numbers.**
   Log lines carry their evidence: connect
   times in the handshake, signal strength
   in `wifi_signal`, the API's own warning
   flags. Chapter 12 indexes the symptoms;
   the numbers decide which entry is yours.

What this order refuses to do is start at
the hardware — reseating sensors and
re-flashing at random — while the file
has not yet been asked a single question.
In the lab's experience the file answered
every question it was asked. That won't
always be true, and chapter 12 says what
the hardware-side failures look like. But
the order is the order because the odds
are the odds.
