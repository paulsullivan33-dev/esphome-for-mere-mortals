# Your First Device

**Scope:** the whole first-device path —
install ESPHome, write the greenhouse
configuration, prove it valid, compile
its twin for the host platform, run it —
and an honest account of the one step this
lab could not take. If you have a real
board, your path forks from this one at
exactly one signposted paragraph.

## Install the tool

ESPHome is a Python package. A virtual
environment keeps it (and the compiler
toolchain it drives) out of your system's
way:

```
$ python3 -m venv ~/esphome-venv
$ ~/esphome-venv/bin/pip install esphome
$ ~/esphome-venv/bin/esphome version
Version: 2026.10.0
```

Check: the version prints, as in chapter 1.
Windows and macOS readers: the same pip
install works; the differences arrive at
flashing time (serial port names, USB
drivers), not here. There is also a
**dashboard** — a local web page for
managing devices. In this version it has
moved out of ESPHome itself into a
separate tool, ESPHome Device Builder
(the command says so, in terms, if you
ask for the old one) — but everything the
dashboard does, the command line does
too, and the command line is what this
guide can show you honestly. Nothing
below needs it.

## Write the configuration

One folder per device is the habit that
survives contact with a second device:

```
greenhouse/
  greenhouse.yaml     the configuration
  secrets.yaml        its secrets (chapter 6)
```

The greenhouse configuration is printed
in full in `examples/greenhouse.yaml` and
anatomised in chapter 3; the specimen
chapter (5) walks its working parts. Two
pieces of it belong to *you*, not the
example: anything behind a `!secret`
reference, which you supply in your own
`secrets.yaml` (`examples/secrets.yaml.example`
is the shape), and the API encryption key,
which must be 32 bytes, base64-encoded.
Generate one and put it in the secrets
file, not the configuration:

```
$ openssl rand -base64 32
```

## Prove it before you build it

```
$ esphome config greenhouse.yaml
INFO Configuration is valid!
```

(Chapter 3 quotes the full form of this
and of a failure.) No board is involved in
this step, which is precisely its value:
every mistake of name, pin, or reference
is caught here, in seconds, by name.

## Build the twin, run the logic

Before any firmware meets a chip, this
guide asks a question most guides skip:
*does the logic do what the file says?*
ESPHome's host platform answers it. The
twin configuration, `examples/greenhouse-sim.yaml`,
is the same device re-aimed — `host:`
instead of the board section, template
(platform-faked) components instead of
wired ones, and a lambda that raises the
temperature 0.7 degrees every five seconds
so the automation has something to meet.

It validates the same way (`config`, t02),
and then it *compiles* — for real, the
full code-generation and C++ build, about
a minute on the build machine:

```
$ esphome compile greenhouse-sim.yaml
...
[133/133] LINK program
INFO Successfully compiled program to path '.../greenhouse-sim/program'
```

(Transcript t03, trimmed to its tail in
chapter 9's discussion.) The result is an
ordinary executable. Run it and it behaves
like the device at boot — the lines below
are its own output (transcript t04):

```
[I][app:073]: Running through setup()
[I][app:130]: setup() finished successfully!
[I][app:164]: ESPHome version 2026.10.0 compiled on 2026-10-22 02:39:52 +0000
```

followed by its configuration dump, which
records — among much else — that its API
server listens on port 6053 with Noise
encryption on, and its update service on
8082 with a password configured. Chapter 8
connects a client to that server and
watches the fan automation fire; chapter 7
sends it an update. The twin is not the
device. But a configuration whose logic
has survived the twin arrives at its first
flash with the embarrassing mistakes
already spent.

## The build for real silicon, and the step beyond it

The ESP32 configuration was also compiled
for real — not merely validated. The first
such build is the large one: PlatformIO
downloads the ESP-IDF framework (5.5.6
here) and the chip toolchain before a
line of your code is compiled, and on the
build machine that download, not the
compile, was the wait. What came out the
far end (transcript t08):

```
INFO Created: .../greenhouse/build/firmware.factory.bin
INFO Created: .../greenhouse/build/firmware.ota.bin
INFO Successfully compiled program.
```

Real artifacts with real sizes:
`firmware.ota.bin`, the over-the-air image,
is 860,944 bytes; `firmware.factory.bin`,
the full first-flash image, 926,480. So
the guide can certify the greenhouse file
end to end as far as the compiler: every
section, pin, and reference validated
(t01), and the whole thing built into
firmware a chip would accept.

With a board attached, one command walks
the remaining step:

```
$ esphome run greenhouse.yaml
```

which validates, compiles as above, and
writes the firmware over USB. From the
second change onward the same command can
offer the over-the-air route instead
(chapter 7), and the cable becomes the
rescue tool rather than the routine.

**That flash is the lab's one
documented-only step.** No ESP32 was
attached to the build bench; nothing here
pretends a serial write it didn't watch.
It is described from the project's
documentation, and chapter 12 covers the
ways a first flash commonly goes wrong
and how each is recognised — which, for
most beginners, makes the split a fair
one: the afternoons are lost in the file,
and the file is the part this guide could
prove.
