# Recipes

**Scope:** four small jobs assembled from
pieces the earlier chapters verified.
Each recipe says which of its steps were
actually run in the lab and which are
assembled from run pieces — the series
rule is that a recipe never quietly
upgrades an ingredient into a result.

## 1. The five-minute logic check

*Run in full (transcripts t02, t03, t05).*

The question every configuration deserves
before its first flash: does the logic do
what the file says?

1. Copy the device configuration to a
   twin file. Replace the board section
   with `host:`.
2. Replace wired components with template
   ones, keeping names and `id`s identical.
   Give the template sensor a lambda that
   walks its value across the thresholds
   your automations care about.
3. Validate the twin, compile it, run it.
4. Connect with `esphome logs` and watch
   a full cycle of the automation fire
   and release.

The greenhouse twin's lambda and cycle
are chapter 5's proof. The recipe's value
scales with the automation's complexity:
a five-minute twin has saved more
greenhouse ladders than any other page
in this guide will.

## 2. The fan that doesn't chatter

*Assembled from the chapter 5 automation,
with the threshold logic run in the twin
only in its simple form.*

Chapter 5's rule — on above 30, off
otherwise — chatters at the boundary: at
29.9, 30.1, 29.8 degrees the relay clicks
like a Geiger counter. The grown-up rule
uses two thresholds and a condition with
a memory:

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
      - if:
          condition:
            sensor.in_range:
              id: air_temp
              below: 27.0
          then:
            - switch.turn_off: vent_fan
```

Between 27 and 30, neither rule fires and
the fan keeps whatever state it had —
that silence between thresholds is
**hysteresis**, and it is the difference
between a control and a nuisance. The
same shape serves heaters (reversed),
pumps, and dehumidifiers.

## 3. A door that reports itself

*The sensor and its state stream were run
(t05); the Home Assistant side is chapter
8's, documented.*

The greenhouse door already reports to
the API, and therefore to any paired
controller, with its `device_class: door`
intact (chapter 5). The recipe is the
checklist that makes the report *useful*
rather than merely present:

1. Name it for the reader, not the
   circuit: "Greenhouse Door", not
   "GPIO18 Binary Sensor".
2. Set the device class (`door`, `window`,
   `motion`, `occupancy` — the vocabulary
   is the project's, and the class is what
   dashboards and assistants understand).
3. Verify the truth direction on the
   bench — door open reads open — before
   the device is mounted. `inverted: true`
   is one word, and so is the embarrassment
   of discovering it backwards from the
   sofa.
4. In the controller, alert on the
   *unusual*, not the event: a door open
   for an hour is a fact worth a
   notification; a door opening is not.

## 4. A configuration you can share safely

*Run throughout the lab — the repository
itself is the proof.*

You have built something worth showing a
forum, a friend, or a guide. To share the
design without sharing the keys:

1. Every credential in the configuration
   is a `!secret` reference — Wi-Fi, API
   key, OTA password, fallback password.
   Grep the file for anything that looks
   like a value; if it isn't behind
   `!secret`, move it.
2. `secrets.yaml` is listed in
   `.gitignore`, and was never committed —
   check the history, not just the folder.
3. Run `esphome config` once more in the
   shared folder: its output echoes
   secrets as `!secret` references
   (chapter 3), so the validation
   transcript is shareable too.
4. Share the configuration, the secrets
   *template* (names, no values —
   `examples/secrets.yaml.example`), and
   one honest paragraph about what the
   device does. That paragraph is the part
   people actually needed.
