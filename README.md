<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=700&size=40&duration=3000&pause=1000&color=4ADE80&center=true&vCenter=true&multiline=true&repeat=true&width=700&height=100&lines=project-74xxpixel-forge">
    <img alt="project-74xxpixel-forge" src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=700&size=40&duration=3000&pause=1000&color=22C55E&center=true&vCenter=true&multiline=true&repeat=true&width=700&height=100&lines=project-74xxpixel-forge">
  </picture>
</p>

<p align="center">
  <em>A homebrew PC built from discrete 74-series logic chips. GPU, CPU and motherboard from scratch.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/status-Phase%201%20GPU-22c55e?style=for-the-badge" alt="Status">
  <img src="https://img.shields.io/badge/logic-74xx%20Series-f59e0b?style=for-the-badge" alt="Logic">
  <img src="https://img.shields.io/badge/output-640x480%20VGA-3b82f6?style=for-the-badge" alt="Output">
  <img src="https://img.shields.io/badge/license-MIT-a855f7?style=for-the-badge" alt="License">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/EDA-KiCad-314CB0?style=flat-square&logo=kicad&logoColor=white" alt="KiCad">
  <img src="https://img.shields.io/badge/Dev_Host-Raspberry%20Pi-A22846?style=flat-square&logo=raspberrypi&logoColor=white" alt="Raspberry Pi">
  <img src="https://img.shields.io/badge/MCU-ATmega328P-00979D?style=flat-square&logo=arduino&logoColor=white" alt="ATmega">
  <img src="https://img.shields.io/github/last-commit/Herorif/74xxpixelforge?style=flat-square&color=22c55e" alt="Last Commit">
  <img src="https://img.shields.io/github/repo-size/Herorif/74xxpixelforge?style=flat-square&color=6366f1" alt="Repo Size">
</p>

---

<br>

## What is this?

A personal computer built entirely from individual logic chips -- the same 74-series ICs that powered computing in the 1970s and 80s. No FPGAs. No system-on-chips. Every register, every counter, every multiplexer is a physical chip on a custom PCB.

The only microcontroller in the entire system is a small ATmega328P that bridges SPI communication on the GPU module. Everything else -- the graphics pipeline, the CPU, the bus logic -- is discrete.

<br>

## Architecture

```
┌─────────────┐     SPI Bus      ┌─────────────┐
│ Raspberry Pi │◄────────────────►│  GPU Module  │──── VGA Out ──► Monitor
│ (Dev Host)   │                  │              │
└─────────────┘                  └─────────────┘

           ╌╌╌ Phase 1: GPU development against Pi ╌╌╌

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

           ╌╌╌ Phase 2+: Full homebrew computer ╌╌╌
```

<br>

## Modules

### `gpu/` -- Graphics Coprocessor &nbsp; ⬤ Active

> 640x480 VGA output from discrete logic. 256 colors. Custom drawing commands. Connects to a Raspberry Pi over SPI for development, or to the homebrew CPU via system bus.

| Spec               | Value                                    |
|---------------------|------------------------------------------|
| Resolution          | 640 x 480 @ 60Hz                        |
| Color               | 8-bit indexed, 256-entry 18-bit palette  |
| VRAM                | 512KB SRAM (10ns access time)            |
| Pixel Clock         | 25.175 MHz                               |
| VRAM Clock          | ~50.35 MHz (interleaved dual-access)     |
| Host Interface      | SPI (to Pi) or 8-bit parallel system bus |
| Video Output        | Analog VGA via 6-bit R-2R DAC            |
| Drawing Commands    | 14 commands (pixel, line, rect, fill, blit, scroll, text, palette) |
| Logic Family        | 74AC (high-speed path), 74HCT (control)  |

<details>
<summary><strong>GPU Documentation</strong></summary>

<br>

- [`gpu/docs/architecture.md`](gpu/docs/architecture.md) -- Block diagram and theory of operation
- [`gpu/docs/command-set.md`](gpu/docs/command-set.md) -- SPI command protocol (14 commands)
- [`gpu/docs/vga-timing.md`](gpu/docs/vga-timing.md) -- VGA signal timing and R-2R DAC design
- [`gpu/docs/memory-map.md`](gpu/docs/memory-map.md) -- VRAM layout, palette RAM, internal registers
- [`gpu/docs/bom.md`](gpu/docs/bom.md) -- Bill of materials (~$48-84 estimated)

</details>

<br>

### `cpu/` -- 8-bit Discrete Logic CPU &nbsp; ○ Phase 2

> Custom instruction set. ALU built from 74181 bit-slice chips. Register file, program counter and EEPROM-based microcode control unit -- all on a single PCB that plugs into the motherboard.

<br>

### `motherboard/` -- System Backplane &nbsp; ○ Phase 3

> 40-pin system bus, address decoding, clock distribution, power regulation, 32KB RAM, 16KB ROM and UART serial terminal.

<details>
<summary><strong>System Documentation</strong></summary>

<br>

- [`motherboard/docs/bus-standard.md`](motherboard/docs/bus-standard.md) -- 40-pin bus specification with timing diagrams
- [`motherboard/docs/design-decisions.md`](motherboard/docs/design-decisions.md) -- Design rationale log (8 decisions documented)

</details>

<br>

## Repository Structure

```
project-74xxpixel-forge/
│
├── gpu/                          ← ACTIVE
│   ├── docs/                     5 design documents
│   ├── schematics/               KiCad schematics
│   ├── pcb/                      KiCad PCB layout
│   ├── firmware/                 EEPROM microcode + lookup tables
│   └── software/                 Raspberry Pi SPI driver
│
├── cpu/                          ← PHASE 2
│   ├── docs/
│   ├── schematics/
│   ├── pcb/
│   └── firmware/
│
├── motherboard/                  ← PHASE 3
│   ├── docs/                     Bus standard + design decisions
│   ├── schematics/
│   ├── pcb/
│   └── firmware/
│
├── README.md
├── LICENSE
└── .gitignore
```

<br>

## Design Principles

| | Principle | Detail |
|---|---|---|
| **1** | Discrete logic over microcontrollers | The CPU and GPU pipeline are 74-series chips. No hidden MCU doing the real work. |
| **2** | Modular | Each module is a self-contained PCB with a defined bus interface. Develop and test independently. |
| **3** | GPU-first | Test against a Raspberry Pi over SPI before the CPU or motherboard exist. |
| **4** | Documented for learning | Every decision explained with alternatives and reasoning. |

<br>

## Estimated BOM

| Module      | Estimated Cost | Status |
|-------------|----------------|--------|
| GPU         | $48 - $84      | Documented |
| CPU         | TBD            | Planned |
| Motherboard | TBD            | Planned |

<sub>Costs assume sourcing from LCSC/JLCPCB. PCB fabrication included.</sub>

<br>

## Tools

| Tool | Purpose |
|------|---------|
| **KiCad** | Schematic capture and PCB layout |
| **Logic Analyzer** | Bus signal debugging |
| **Oscilloscope** | VGA timing verification |
| **Raspberry Pi** | GPU development host (SPI) |
| **TL866II+** | EEPROM programmer for microcode |

<br>

## Inspiration

| Project | What I learned |
|---------|----------------|
| [Ben Eater's 8-bit computer](https://eater.net/8bit) | Discrete logic CPU fundamentals |
| [bitluni's RISC-V Supercluster](https://github.com/bitluni/Supercluster2) | Parallel microcontroller clustering |
| [Gigatron TTL computer](https://gigatron.io/) | TTL-only computer with VGA output |
| Classic VGA chipsets (V9938, TMS9918) | Palette-based graphics architecture |

<br>

## Roadmap

- [x] Define GPU architecture and command set
- [x] Define system bus standard (40-pin)
- [x] Document VGA timing and memory map
- [x] Bill of materials
- [ ] GPU KiCad schematics -- VGA scanout engine
- [ ] GPU KiCad schematics -- command processor
- [ ] GPU KiCad schematics -- SPI interface
- [ ] GPU PCB layout
- [ ] GPU fabrication and assembly
- [ ] Raspberry Pi SPI driver and test patterns
- [ ] GPU bring-up and VGA output test
- [ ] CPU architecture and instruction set design
- [ ] Motherboard schematic and PCB
- [ ] Full system integration

<br>

## License

Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details.

Hardware designs are open source. Build one yourself.

<br>

---

<p align="center">
  <sub>Built with 74xx logic, solder and stubbornness.</sub>
</p>
