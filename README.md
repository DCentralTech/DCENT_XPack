# DCENT Expansion Pack

> The miner-side Expansion Pack for [DCENT_OS](https://github.com/DCentralTech/DCENT_OS).
> Plug it in, scan a QR code, and your Antminer is on Wi-Fi — no travel router,
> no IP hunting, no closed-source bridge firmware.

**DCENT_ExpansionPack** (a.k.a. **DCENT_XPack**) is a small, low-cost, open-source
extension board for industrial Bitcoin miners running DCENT_OS. Clip it onto your miner and DCENT_OS levels up — gaining
home Wi-Fi, an OLED status display, optional Touch–style accessories,
and external temperature feedback for [Mining Heating mode](https://github.com/DCentralTech/DCENT_OS).

It is built and produced by D-Central Technologies and made open-source so anyone, anywhere can buy one, build one,
or fork the design.

---

## Why this exists

Antminers, Avalons, and WhatsMiners ship with one network port and zero Wi-Fi.
For institutional miners that's fine — they live in racks with structured
cabling. For **home miners**, it's the single biggest onboarding pain point.

The status quo for "I want my Antminer on Wi-Fi" is:

- **Vonets / generic travel routers.** Closed firmware, finicky to configure,
  external bricks dangling off the miner with their own power adapter,
  expensive in some regions, no integration with the miner itself.
- **Run Cat-5e across the house.** Works, but most home miners don't have a
  switch in the closet they're heating.
- **USB Wi-Fi dongles.** Kernel-module gymnastics on every firmware update.

None of these solve the *real* onboarding problem either: even after the miner
is on the network, the user still has to find its IP address, log into the
router admin page, and bookmark the right URL. New home miners get stuck here
all the time.

DCENT_XPack replaces all of that with **one small board** that does four jobs:

1. **Wi-Fi onboarding** — scan a QR code, type your home SSID + password, done.
2. **Stable local name** — the miner is always reachable at
   `http://dcent-pack-XXXX.local/`. No IP lookups, ever.
3. **Hardware accessories** — onboard temperature sensor, optional OLED, and a
   header for Touch–style display panels, inspired by the Bitaxe Touch.
4. **DCENT_OS integration** — the miner doesn't need to know your Wi-Fi
   credentials, it just sees an Ethernet gateway and pairs with the bridge.

It is **open hardware** (KiCad sources in this repo) and **open firmware**
(ESP-IDF sources in this repo), with a signed OTA path so the bridge stays
updatable in the field without us touching it.

---

## How it works

```
+--------------+        +-----------------------+        +----------------+
| Your home    |  Wi-Fi |  DCENT Expansion Pack | RJ45   |  Antminer /    |
| Wi-Fi router | <----> |  ESP32-C6 + W5500     | <----> |  Avalon /      |
| (2.4 GHz)    |        |  10.77.0.1/24         |        |  WhatsMiner    |
+--------------+        +-----------------------+        |  running       |
                                                         |  DCENT_OS      |
                                                         +----------------+
```

The bridge runs as a small managed router, not a fragile Layer-2 Wi-Fi bridge:

- **Wi-Fi side** joins your home network as a normal STA.
- **Miner side** is a private Ethernet subnet (`10.77.0.0/24`) with a built-in
  DHCP server and NAPT so the miner gets internet access transparently.
- **mDNS** advertises `dcent-pack-XXXX.local` so you and DCENT_OS find the
  miner without ever knowing its IP address.
- **Reverse proxy** forwards your browser straight through to the miner's
  DCENT_OS dashboard. Type one URL, see the miner.
- **Pairing** happens automatically: DCENT_OS POSTs its identity to the
  default gateway after DHCP and the bridge remembers it across reboots.

Your Wi-Fi password lives **only** on the bridge — the miner never sees it. A
factory-reset button on the bridge wipes credentials cleanly without bricking
anything.

---

## Key features

- **One-QR setup** under five minutes from box-open to mining over Wi-Fi.
- **No miner IP discovery.** `dcent-pack-XXXX.local` redirects to the miner.
- **External temperature feedback** so DCENT_OS Mining Heating mode can
  control hashrate based on the *room* temperature, not the chip junction.
- **Optional OLED** on a 4-pin I²C accessory plug (SSD1306-class, Bitaxe-style).
- **Bitaxe Touch compatibility** via a BAP UART accessory header. Drop in any
  BAP-speaking touchscreen and the bridge mirrors DCENT_OS state to it.
- **Signed OTA** — firmware updates are HMAC-verified before flash.
- **Captive-portal Wi-Fi provisioning** with live credential pre-validation
  (wrong password fails fast with a useful error, not a timeout).
- **Privacy-respecting.** No cloud account, no telemetry phone-home, no
  remote-management backdoor. All traffic stays on your LAN.
- **Auditable.** Hardware (KiCad) and firmware (ESP-IDF) are open source and
  reproducible from this repository.

---

## Hardware

Bill of materials is JLCPCB-native — assembled basic and extended parts only,
no hand-soldering required for a working board.

| Block          | Part                                                        |
| -------------- | ----------------------------------------------------------- |
| Wi-Fi MCU      | ESP32-C6-WROOM module (Wi-Fi 6 capable, 2.4 GHz)            |
| Ethernet PHY   | W5500 SPI Ethernet controller                               |
| Connector      | RJ45 MagJack (HR911105A-class)                              |
| Power input    | Protected 12 V → 3.3 V buck, 1 A min, 2 A preferred         |
| Temperature    | TMP102-class digital I²C sensor                             |
| Local display  | Optional I²C OLED accessory plug                            |
| Touch display  | Optional BAP UART accessory header                          |
| Setup UX       | Per-unit QR sticker, setup/reset button, status RGB LED     |
| Factory / dev  | USB-C or UART header for flashing and serial logs           |

Power input options are SKU-configurable (2-pin locking harness, 2.1 mm barrel,
or Mini-Fit-style) so the same board fits different miner families and cable
kits.

See [`hardware/`](hardware/) for KiCad sources, BOM tables, footprint review
notes, and the pre-JLCPCB release gate. See
[`docs/PRE_JLCPCB_REVIEW.md`](docs/PRE_JLCPCB_REVIEW.md) for the manufacturing
checklist.

---

## Miner compatibility

DCENT_XPack pairs with any miner running [DCENT_OS](https://github.com/DCentralTech/DCENT_OS).
Stock Bitmain firmware, BraiinsOS, VNish, and LuxOS are not pairing targets —
the bridge is built around DCENT_OS's pairing contract.

| Miner family                         | Status                                       |
| ------------------------------------ | -------------------------------------------- |
| Antminer S9 (Zynq)                   | Supported (DCENT_OS sustained mining proven) |
| Antminer S17 / S19 / S19j Pro / S21  | Supported (DCENT_OS Zynq + Amlogic)          |
| Antminer S19k Pro (Amlogic)          | DCENT_OS port in progress                    |
| AvalonMiner (Canaan, K230 RISC-V)    | Roadmap — DCENT_OS-Avalon in development     |
| WhatsMiner M-series (H616 ARM)       | Roadmap — DCENT_OS-WhatsMiner in development |

The bridge itself is miner-agnostic at the hardware level (12 V in, RJ45 out)
and at the protocol level (the pairing contract is HTTP, not vendor-specific).
As DCENT_OS expands to new platforms, DCENT_XPack picks them up for free.

---

## DCENT_OS integration contract

DCENT_OS does not need to know anything about your Wi-Fi. It keeps Ethernet
DHCP enabled and adds a tiny pairing client that:

1. Gets a DHCP lease from the bridge (typically `10.77.0.2`).
2. Detects the default gateway (`10.77.0.1`).
3. POSTs miner identity + dashboard metadata to `http://<gateway>/pair`.
4. Polls bridge telemetry for external temperature when Mining Heating mode
   wants it.
5. Sends periodic heartbeats so the bridge knows the miner is alive.

That's it. No custom kernel modules on the miner, no Wi-Fi drivers, no
credentials stored on the miner side. See
[`docs/DCENT_OS_BRIDGE_CLIENT.md`](docs/DCENT_OS_BRIDGE_CLIENT.md) for the
reference client implementation and
[`docs/PAIRING_FLOW.md`](docs/PAIRING_FLOW.md) for the full state machine.

---

## Status

**Phase 0 — Pre-JLCPCB design freeze.**

- Firmware v0.1 is feature-complete on ESP32-C6 (Wi-Fi STA + setup AP + captive
  portal + W5500 Ethernet + DHCP server + NAPT + mDNS + reverse proxy + signed
  OTA scaffold + factory reset).
- KiCad v0.1 schematic + PCB are scaffolded and going through ERC/DRC review.
- Pre-JLCPCB review gate scripts are in place under
  [`scripts/pre_jlcpcb_gate.ps1`](scripts/pre_jlcpcb_gate.ps1).

**Next milestones:**

- Ship PCB v0.1 to JLCPCB for first-article assembly.
- Bench bring-up against the DCENT_OS S9 / S19j Pro / S21 fleet.
- Field pilot of 10–25 units across common Antminer control-board families.
- Public sale via [d-central.tech](https://d-central.tech).

See [`docs/ROADMAP.md`](docs/ROADMAP.md) for the full plan.

---

## Repository layout

```
dcent-expansion-pack/
├── hardware/      KiCad schematic + PCB, BOM, JLCPCB review notes
├── firmware/      ESP-IDF C firmware for the ESP32-C6 bridge
├── docs/          Pairing flow, customer onboarding, API contracts, roadmap
├── scripts/       Pre-JLCPCB release gate + manufacturing helpers
└── tools/         Bench tools and bring-up utilities
```

Highlights:

- [`docs/CUSTOMER_ONBOARDING.md`](docs/CUSTOMER_ONBOARDING.md) — five-minute
  setup walkthrough (also in
  [French](docs/CUSTOMER_ONBOARDING_FR.md)).
- [`docs/BRIDGE_API.md`](docs/BRIDGE_API.md) — REST API surface.
- [`docs/HOMEASSISTANT_INTEGRATION.md`](docs/HOMEASSISTANT_INTEGRATION.md) and
  [`docs/ESPHOME_INTEGRATION.md`](docs/ESPHOME_INTEGRATION.md) — home-automation
  hooks.
- [`docs/SUPPORT_RUNBOOK.md`](docs/SUPPORT_RUNBOOK.md) — internal support
  playbook.

---

## About D-Central

[D-Central Technologies](https://d-central.tech/) is Canada's leading Bitcoin
mining technology company. Founded in 2016 in Québec, Canada. The Bitcoin *Mining Hackers*. 2,500+ miners repaired, 400+ products, and a stubborn belief
that **every industrial ASIC deserves a second life as a home space heater**.

DCENT_XPack is part of the broader DCENT ecosystem:

- [**DCENT_OS**](https://github.com/DCentralTech/DCENT_OS) — open-source
  firmware for Antminer, AvalonMiner, and WhatsMiner.
- **DCENT_axe** — DCENT_OS for the BitAxe.
- **DCENT Pool** — D-Central's open-source mining pool.
- **DCENT_XPack** — this project.

---

## License

- **Hardware:** [CERN-OHL-S-2.0](LICENSE-HARDWARE) (open hardware, share-alike)
- **Firmware:** [GPL-3.0](LICENSE-FIRMWARE) (open firmware, share-alike)
---
