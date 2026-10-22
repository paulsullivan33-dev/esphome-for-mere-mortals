# Wi-Fi and Secrets

**Scope:** how a device with no screen and
no keyboard joins a password-protected
network, what it does when it can't, and
where the passwords live in the meantime.
The arrangement is genuinely clever and
genuinely imperfect; this chapter says
both, plainly.

## The joining problem

A laptop asks you for the Wi-Fi password.
A chip sealed in a greenhouse can't. So
the password has to be *in the firmware* —
compiled in, from the configuration, at
build time. ESPHome's Wi-Fi section is
therefore both a network setting and a
custody arrangement for a secret:

```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "Greenhouse Fallback"
    password: !secret ota_password
```

On boot the device joins the named
network and takes a DHCP address, and is
thereafter reachable by its mDNS name —
`greenhouse.local` — without anyone
recording the address. (A `manual_ip:`
block exists for readers who pin addresses;
the guide's specimen lets DHCP and mDNS
do their jobs.)

## The fallback: a device that asks for help

The `ap:` block is the plan for failure.
If the device cannot join — wrong password,
router down, greenhouse moved out of range
of the network it remembers — it does not
sit in the dark. After a timeout it starts
its **own** little network, the fallback
hotspot ("Greenhouse Fallback" above), and
the `captive_portal:` component named back
in chapter 3 serves a page on it: connect
your phone to the hotspot, and the page
lets you hand the device new Wi-Fi
credentials, which it keeps and retries
with. This is the same trick every smart
plug uses on its first day, and it doubles
as the rescue path for the much commoner
accident: *you* changed the home Wi-Fi
password, and every device you own is now
stranded. The fallback password is a
secret too, and the specimen deliberately
reuses the OTA password for it — one fewer
password to invent, and a choice the file
makes visibly rather than by accident.

Honest limits of the rescue: it needs the
device within radio reach of a person with
a phone, and it needs the device *running*
— a fallback hotspot does not fix a device
whose configuration is broken (that is
safe mode's job, chapter 7, and ultimately
the USB cable's).

## Secrets: the bargain

The `!secret` tag is a YAML-level promise:
*this value lives somewhere else.* At
validation and build time ESPHome looks
each name up in **`secrets.yaml`**, a file
beside the configuration:

```yaml
# secrets.yaml — yours will differ in every value
wifi_ssid: "Marigold Cottage WiFi"
wifi_password: "correct-horse-battery-staple-42"
api_key: "..."
ota_password: "hummingbird-ota-1234"
```

(The values above are invented for a
fictional household and protect nothing;
`examples/secrets.yaml.example` is the
template with the key-generating command
in a comment. The lab's real secrets file
was never copied into this repository —
and, as chapter 3 noted, `esphome config`
echoes secrets back as `!secret`
references, so even the validation
transcript carries none of them.)

What the bargain buys:

- **The configuration becomes shareable.**
  You can post your greenhouse.yaml in a
  forum, commit it to a public repository,
  or print it in a guide, and it contains
  no password to steal. The interesting
  part of a configuration — the design —
  travels; the credentials don't.
- **One file to guard, one file to rotate.**
  Change the Wi-Fi password and you edit
  one line in one place; every device
  built from that folder follows at its
  next build.

And what it does not buy, stated plainly:

- **`secrets.yaml` is plaintext.** It is a
  filing system, not a safe. Anyone who
  can read the file has the passwords;
  ESPHome offers no encryption for it.
  Treat the folder like a key drawer.
- **The secret still ends up in the
  firmware.** It has to — the device must
  know the password to use it. Someone
  with the physical chip and determination
  can extract firmware; for home devices
  the realistic defence is that the
  password they would win is your Wi-Fi's,
  which is one more reason the fallback
  and OTA passwords should not be the
  password to anything else you value.
- **The lookup is by folder.** Move a
  configuration without its `secrets.yaml`
  and validation fails on the spot, naming
  the missing file (`Error reading file
  .../secrets.yaml: No such file or
  directory`, transcript t11) — an alarming
  error the first time, and the correct
  behaviour every time.

A closing discipline worth adopting
verbatim: `secrets.yaml` goes in
`.gitignore` before the folder's first
commit. ESPHome's own tooling assumes the
habit; the guides in this series state it
because habits stated are habits kept.
