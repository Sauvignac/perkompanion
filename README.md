# Perkompanion

> A hardware + software companion for the Erica Synths Perkons HD-01.

🇫🇷 [Lire en français](README.fr.md)

---

## What is Perkompanion?

Perkompanion is a modular hardware and software extension designed to expand the capabilities of the [Erica Synths Perkons HD-01](https://www.ericasynths.lv/shop/standalone-instruments/perkons-hd-01/). It adds a drum trigger pad, virtual voices, motion recording, hotcue library, DSP effects, and deep MIDI routing — while keeping the Perkons as the master clock and the core percussion engine.

The goal is to turn the Perkons into a full live-performance instrument without replacing what makes it unique.

---

## Status

**Version 0.9** — design finalized, components on order, assembly phase starting.

This project is in active development. The hardware architecture and software specifications are documented (see [Documentation](#documentation) below), but no firmware or software has been released yet. The repository currently contains the design and technical documentation.

---

## Key features (planned)

- **8×4 Cherry MX trigger pad** with 8 voice selectors and 5 mode buttons integrated into an okoumé wood panel
- **8 voice columns** (V1–V4 physical through the Perkons, V5–V8 virtual) with individual OLED displays and encoders
- **128 hotcues library** stored in PSRAM for instant triggering
- **Motion recording** to capture live parameter automations
- **Multi-slot DSP chain** per voice with 13 effect types
- **Deep MIDI routing** via a 7-inch touchscreen interface
- **Fail-safe mode**: remains playable even if the Pi or Teensy stops responding — the Perkons stays autonomous

---

## Hardware architecture

- **Teensy 4.1 Fully Loaded (32 MB PSRAM)**: real-time core for MIDI, audio DSP, sequencing, hotcues
- **Raspberry Pi 5**: user interface (Chromium kiosk mode on a 7-inch touchscreen), hotcue library management
- **Audio path**: 2× PCM1808 (4 inputs from Perkons) + 6× PCM5102A (12 outputs) + 1× TPA6120 headphone amplifier
- **IO expansion**: 20× MCP23S17 on 3 SPI chains for buttons, LEDs, encoders
- **Panel**: 450×370 mm matte black acrylic base plate + 190×115 mm okoumé plywood module for the 9×5 button grid
- **MIDI**: 1 IN + 1 OUT DIN connected to the Perkons via ESI M8U eX router

Full details in [`panel_design.md`](panel_design.md) and [`teensy_pinout.md`](teensy_pinout.md).

---

## Roadmap

- **Phase 0** (current): documentation, architecture freeze, component sourcing
- **Phase 1**: Teensy + Pi bring-up, basic MIDI routing, screen display
- **Phase 2**: virtual voices, hotcue engine, basic DSP
- **Phase 3**: full DSP chain, motion recording, advanced routing
- **Phase 4**: final enclosure, performance polish, public release

See [`PERKOMPANION_VISION.md`](PERKOMPANION_VISION.md) for the detailed roadmap.

---

## Documentation

The design and technical specifications live in this repository:

- [`PERKOMPANION_VISION.md`](PERKOMPANION_VISION.md) — Product vision, features, roadmap, BOM
- [`PERKOMPANION_PHASE0.md`](PERKOMPANION_PHASE0.md) — Foundational technical decisions
- [`PERKOMPANION_PROTOCOL_TABLES.md`](PERKOMPANION_PROTOCOL_TABLES.md) — Reference tables (MIDI, protocol, IDs)
- [`panel_design.md`](panel_design.md) — Mechanical panel layout and dimensions
- [`teensy_pinout.md`](teensy_pinout.md) — Teensy 4.1 GPIO assignments
- [`midi_hardware.md`](midi_hardware.md) — MIDI DIN interface circuit

---

## Build your own

A full build guide will be published once Phase 1 is validated. In the meantime, the documentation above should give enough information to start sourcing components and planning a build. Feel free to open an issue if you want to follow along or adapt the project for your own setup.

---

## About the development

Perkompanion is designed and built by Sauvignac, an amateur musician passionate about techno and hardware synthesizers for over twenty years, with no formal background in electronics or embedded development.

The architecture, firmware, and documentation are developed with the assistance of [Claude](https://www.anthropic.com/claude) (Anthropic), used as a design partner to navigate technical decisions, explore trade-offs, and structure implementation details. Every design choice and artistic direction remains Sauvignac's own — Claude is a tool, not a co-author.

This project is also meant as a small demonstration that ambitious hardware builds are becoming accessible to non-engineers when AI assistance is used thoughtfully.

---

## License

MIT License. See [`LICENSE`](LICENSE) for details.

---

## Contact

Issues and discussions welcome on this repository.
