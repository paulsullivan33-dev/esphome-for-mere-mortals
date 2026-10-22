# Limits and Honest Edges

**Scope:** what ESPHome does not do, what
it should not be asked to do, and the
costs built into its bargain. Every tool
in this series gets this chapter; ESPHome's
is not a warning off, but it is a real
list.

## You are maintaining firmware

The YAML is friendly. What it builds is
not a document but a program etched into
a device that may be screwed to a wall in
another building. Consequences that follow
from that, none of them hypothetical:

- **A bad build can stop a device talking.**
  Safe mode (chapter 7) rescues most bad
  configurations; it cannot rescue a build
  whose network or API sections are the
  broken part, nor anything below the
  configuration's level. The final tool is
  a cable and physical access. Sites that
  are hard to reach deserve a bench-tested
  configuration, and the five-minute twin
  (chapter 10) is how you earn one.
- **There is no partial update.** The
  device is its whole file. Two people
  editing two "halves" of one device are
  editing one program, and merge conflicts
  in a device definition are resolved the
  way they are in any codebase: by someone
  reading the whole thing.

## The ceiling of a description

ESPHome's vocabulary is enormous — scores
of component families — and it is still a
vocabulary, not a language. When a project
needs behaviour the components don't
express, the exits are:

- **Lambdas** — real C++, inlined in the
  YAML. Powerful, and the moment a lambda
  grows past a few lines the project has
  quietly changed species: now it is a C++
  program wearing a YAML coat, with a C++
  program's debugging and none of its
  tooling conveniences.
- **External components** — code other
  people wrote, pulled in by reference.
  They extend the vocabulary at the price
  of trusting it.
- **A different tool.** Some jobs — heavy
  signal processing, serious user
  interfaces, anything with a screenful of
  ambition — belong in a hand-written
  firmware or a bigger computer. ESPHome's
  honesty about being a *device description
  system* is a virtue; your project's
  honesty about outgrowing it should match.

## The living costs

- **Monthly motion.** ESPHome releases
  every month and makes breaking changes
  without apology: components split (this
  guide's captured `bme280` error is a
  fossil of one), options rename, defaults
  move. A configuration is pinned to the
  era of the tool that builds it; builders
  who keep devices for years keep the
  ESPHome version that built them, or set
  aside an afternoon per upgrade to read
  the release notes and fix what moved.
  Neither is wrong; pretending neither is
  necessary is.
- **Wi-Fi is the assumed road.** The
  native API, OTA, and discovery all ride
  the network the device joined. Ethernet
  variants exist; Thread and Zigbee are
  neighbours, not ESPHome's transport.
  A device that must live where Wi-Fi
  doesn't is a different chapter of a
  different guide.
- **The device trusts its network.**
  The API's encryption is real, but the
  threat model is a home LAN: a flat
  network where reaching the device is
  already most of the battle. ESPHome is
  not a hardening framework, and a device
  on a network you do not control deserves
  the scepticism of chapter 6's secrets
  discussion, doubled.

## The bench's own limits

This guide's lab deserves its entry in
the list, because a reader could
otherwise leave with more confidence than
the evidence bought:

- **The host platform is not the chip.**
  It compiles the same generated code and
  runs the same logic, and that is all.
  Timing, memory pressure, radio
  behaviour, sensor truth — none of it
  transfers. The twin was observed, in
  this sandbox, to sit quiet after its
  boot dump until a client connected;
  whether quirk or sandbox, it underlines
  the point: the twin proves the *file*,
  not the *device*.
- **Host oddities are real oddities.**
  The twin's uptime sensor reported the
  *computer's* uptime (thousands of
  seconds, minutes after the program
  started), because on the host platform
  there is no other uptime to report.
  Small, harmless, and exactly the kind of
  difference that multiplies quietly
  between a simulator and silicon.
- **Nothing physical was measured.** No
  sensor reading in this guide is a
  measurement of air. No range, current
  draw, or temperature figure for real
  hardware appears anywhere in it, and if
  a number ever seems to, it is the
  simulated lambda's, labelled as such.

The summary a fair reviewer would write:
ESPHome trades the open-ended power of
writing firmware for the speed and safety
of describing it — an excellent trade for
the large class of devices that are a few
sensors, a few switches, and a few rules,
which happens to be most of the devices a
house actually wants.
