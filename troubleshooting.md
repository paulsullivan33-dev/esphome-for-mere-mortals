# Troubleshooting

**Scope:** symptoms, in the order a builder
meets them — file, then logic, then
network, then hardware. The entries drawn
from the lab cite their transcripts; the
rest are the documented failure shapes of
the path, worded as such.

## "Platform not found"

```
Failed config

Platform not found: 'sensor.bme280'
```

*(Lab, t06.)* Validation refuses a platform
name the installed ESPHome doesn't have.
Causes, in order of likelihood: the example
you copied is older than your ESPHome —
components get split and renamed between
releases (`bme280` became `bme280_i2c` and
`bme280_spi`); a typo; or the platform
belongs to a different domain than the one
you put it under. The fix is the component
list for *your* version. Validation exits
with code 2 and stops at the first fault —
fix it and run `config` again to meet the
next one, if there is one.

## "Configuration is valid" but the device does nothing

Begin with chapter 9's order. If the file
validates and compiles, ask the device
what it built: the boot dump (serial, or
`esphome logs`) restates every component
as loaded. A component missing from the
dump was missing from the build. If the
dump is right and the behaviour is wrong,
reproduce the behaviour in the host twin
(chapter 10, recipe 1) where you can watch
it at DEBUG with no ladder involved.

## The simulator sits quiet

*(Lab, t04 versus t05.)* The host build
printed its boot and configuration dump,
then produced no periodic output while
left alone; connected over the API, the
same program streamed its state every five
seconds. Treat a quiet standalone host
build as unproven rather than dead:
connect a client (`esphome logs`), and
judge by the stream. On hardware this
entry does not apply — a chip's log is
its serial line, which has no audience
requirement.

## "Waiting for client connection"

```
[W][component:290]: api set Warning flag: waiting for client connection
```

*(Lab, t04, t07.)* The API component's
boot-time note that nobody has connected
yet. On a device nobody has paired, this
is the expected state of the world, and
it clears when a client arrives. It earns
attention only when a client *should* be
connected and isn't — then it is evidence
about the network path, not about the API.

## A client disconnects noisily

```
[W][api.connection:2822]: ESPHome Logs 2026.10.0 (127.0.0.1): Reading failed CONNECTION_CLOSED errno=11
```

*(Lab, t07.)* The device mourning a client
that vanished without ceremony — here, a
log viewer stopped mid-session. Harmless
in itself; the device serves the next
client normally. If the line repeats on a
rhythm, something on the network is
connecting and dropping, and the rhythm
is the clue to take to the network, not
to the device.

## The uptime is impossible

*(Lab, t05.)* The twin reported an uptime
of thousands of seconds minutes after it
started. On the host platform the uptime
sensor reads the *computer's* uptime —
there is no other clock of that kind to
read. On hardware, the same sensor reads
the device's own. Rule: check which
platform a strange number came from
before doubting the sensor, the config,
or yourself.

## OTA refuses, or "Warning flag: unspecified"

During the lab's update the device logged
`esphome.ota set Warning flag: unspecified`
at the start of the transfer — and then
completed it (t10, t09: `OTA successful`).
A warning flag at the start of an update
is the component clearing its throat, not
a verdict; judge an update by its ending.
Actual refusals name their reasons: a
wrong OTA password fails the handshake
outright, and a device in the middle of
its safe-mode window (chapter 7) is
waiting for exactly this fix. If uploads
fail repeatedly from one machine and work
from no other, suspect the network path —
OTA is a fresh inbound connection to the
device, and guest networks and client
isolation eat those for breakfast.

## The sensor isn't found on the bus

Documented shape, not a lab capture: the
device boots, the I2C scan (chapter 3's
`scan: true`) reports nothing at the
expected address, and every reading is
absent. The causes are physical, in this
order in the real world: power and ground;
SDA and SCL swapped; the address pin
selecting `0x77` while the file says
`0x76`. The scan's output is the whole
point of turning it on — it converts
"the sensor doesn't work" into "nothing
answers at 0x76", which is a sentence a
person can act on.

## The first flash fails

Also documented rather than captured —
the lab flashed nothing. The project's
common first-flash failures: the USB cable
carries power but no data (charge-only
cables are the classic); the board needs
its BOOT button held while the write
begins; the serial port belongs to another
program; on some systems, a driver for the
board's USB-to-serial chip is missing.
ESPHome's own installers and the web
flasher name most of these as they happen.
What the lab *can* promise is the half
before the cable: a configuration that
validated, and logic that survived its
twin, arrives at this step with its
software excuses already used up — which
leaves the cable, where this chapter says
to look first anyway.
