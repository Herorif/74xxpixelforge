# project-74xxpixel-forge

A homebrew PC built from discrete 74-series logic chips. GPU, CPU and motherboard from scratch.

## Overview

This project builds three core hardware modules from the ground up using 74-series discrete logic ICs on custom PCBs. No FPGAs. The only microcontroller used is a small ATmega328P for SPI bridging on the GPU module — everything else is logic gates, counters, registers and adders.

```
┌─────────────┐     SPI Bus      ┌─────────────┐
│ Raspberry Pi │◄────────────────►│  GPU Module  │──── VGA Out ──► Monitor
│ (Dev Host)   │                  │              │
└─────────────┘                  └─────────────┘

           --- Phase 1: GPU development against Pi ---

┌─────────────┐                  ┌─────────────┐
│  CPU Module  │◄── System Bus ──►│  GPU Module  │──── VGA Out ──► Monitor
│  (74-series) │        │         │              │
└─────────────┘        │         └─────────────┘
                       │
               ┌───────┴───────┐
               │  Motherboard   │
               │  RAM / ROM     │
               │  UART          │
               │  Addr Decode   │
               │  Clock / Power │
               └───────────────┘

           --- Phase 2+: Full homebrew computer ---
```

## Modules

### [GPU](gpu/) — Phase 1 (Active)

A graphics coprocessor with 640x480 VGA output, 8-bit paletted color (256 colors from an 18-bit programmable palette) and its own 512KB VRAM. Accepts drawing commands over SPI from a Raspberry Pi or over the system bus from the homebrew CPU. Built from 74AC/HCT-series logic with SRAM and EEPROMs.

| Parameter          | Value                              |
|--------------------|-------------------------------------|
| Resolution         | 640x480 @ 60Hz                     |
| Color              | 8-bit indexed, 256-entry 18-bit palette |
| VRAM               | 512KB SRAM                         |
| Pixel Clock        | 25.175 MHz                         |
| Host Interface     | SPI (to Pi) or system bus (to CPU) |
| Video Output       | VGA via R-2R DAC                   |

### [CPU](cpu/) — Phase 2 (Planned)

An 8-bit CPU built from discrete 74-series logic. Custom instruction set with ALU (74181-based), register file, program counter and EEPROM-based microcode control unit. Plugs into the motherboard's expansion bus.

### [Motherboard](motherboard/) — Phase 3 (Planned)

The system backplane. Provides the 40-pin system bus, address decoding, clock distribution, power regulation, 32KB RAM, 16KB ROM and a UART for serial terminal access.

## Repository Structure

```
project-74xxpixel-forge/
├── README.md
├── LICENSE
├── .gitignore
├── gpu/
│   ├── docs/
│   │   ├── architecture.md      # Block diagram and theory of operation
│   │   ├── command-set.md       # SPI command protocol
│   │   ├── vga-timing.md        # VGA signal timing reference
│   │   ├── memory-map.md        # VRAM and register layout
│   │   └── bom.md               # Bill of materials
│   ├── schematics/              # KiCad schematic files
│   ├── pcb/                     # KiCad PCB layout files
│   ├── firmware/                # EEPROM microcode and lookup tables
│   └── software/                # Raspberry Pi SPI driver and test code
├── cpu/
│   ├── docs/                    # (Phase 2)
│   ├── schematics/
│   ├── pcb/
│   └── firmware/
├── motherboard/
│   ├── docs/
│   │   ├── bus-standard.md      # 40-pin system bus specification
│   │   └── design-decisions.md  # Project-wide design rationale
│   ├── schematics/
│   ├── pcb/
│   └── firmware/
```

## Design Philosophy

- **Discrete logic over microcontrollers.** The CPU and GPU command processor are built from 74-series chips. No hidden MCU doing the real work, except the SPI bridge which is explicitly noted.
- **Modular.** Each module is a self-contained PCB with a defined bus interface. Modules can be developed and tested independently.
- **GPU-first.** The GPU connects to a Raspberry Pi over SPI for development and testing before the CPU or motherboard exist.
- **Documented for learning.** Every design decision is explained with alternatives considered and reasoning given.

## Tools

- **KiCad** — Schematic capture and PCB layout
- **Logic analyzer / Oscilloscope** — Signal debugging and VGA timing verification
- **Raspberry Pi** — GPU development host over SPI
- **TL866II+** (or similar) — EEPROM programmer for microcode and lookup tables

## Inspiration

- [Ben Eater's 8-bit breadboard computer](https://eater.net/8bit)
- [bitluni's RISC-V Supercluster](https://github.com/bitluni/Supercluster2)
- [Gigatron TTL computer](https://gigatron.io/)
- Classic VGA chipsets (Yamaha V9938, TMS9918, IBM VGA)

## License

MIT — see [LICENSE](LICENSE)
