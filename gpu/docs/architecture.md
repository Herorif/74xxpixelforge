# GPU Architecture

## Overview

The GPU is a dedicated graphics coprocessor that maintains its own video RAM, accepts drawing commands over SPI and continuously outputs a VGA signal. It operates independently from the CPU — once a draw command is issued, the GPU executes it without further CPU involvement.

## Block Diagram

```
                          ┌─────────────────────────────────────────────────┐
                          │                   GPU MODULE                     │
                          │                                                  │
   SPI from Host ────────►│  ┌───────────┐    ┌──────────────┐              │
   (Pi or CPU)            │  │    SPI     │    │   Command    │              │
                          │  │ Interface  │───►│   FIFO       │              │
                          │  │ (ATmega)   │    │ (74-series)  │              │
                          │  └───────────┘    └──────┬───────┘              │
                          │                          │                       │
                          │                          ▼                       │
                          │                   ┌──────────────┐              │
                          │                   │   Command     │              │
                          │                   │  Processor    │              │
                          │                   │  (74-series   │              │
                          │                   │   state mach) │              │
                          │                   └──────┬───────┘              │
                          │                          │                       │
                          │                   Write  │  Read                 │
                          │                   Port   │  Port                 │
                          │                          ▼                       │
                          │  ┌────────┐    ┌──────────────┐    ┌────────┐  │
                          │  │Palette │    │    VRAM       │    │ VGA    │  │
                          │  │  RAM   │◄───│  512KB SRAM   │───►│Scanout │──────► VGA
                          │  │(256x18)│    │ (interleaved  │    │Engine  │  │     Output
                          │  └────────┘    │  access)      │    └────────┘  │
                          │                └──────────────┘                 │
                          │                                                  │
                          │  ┌──────────────┐                               │
                          │  │ Timing Gen   │──── HSync, VSync, Pixel Clk   │
                          │  │ (25.175 MHz) │                               │
                          │  └──────────────┘                               │
                          └─────────────────────────────────────────────────┘
```

## Functional Blocks

### 1. SPI Interface

**Purpose:** Receives commands and data from the host (Raspberry Pi or CPU module).

**Implementation:** ATmega328P microcontroller. Receives SPI bytes and writes them into the command FIFO. This is the one place a microcontroller is used. SPI slave mode with variable-length messages and flow control would require 15-20 discrete logic chips just for the interface. Using a small MCU here keeps the focus on the GPU's actual graphics pipeline being built from discrete logic.

### 2. Command FIFO

**Purpose:** Buffers incoming commands so the SPI interface and command processor can operate at different rates.

**Implementation:** A small FIFO built from dual-port SRAM (e.g. IDT7130) or 74-series shift registers. 256 bytes is sufficient.

**Signals:**
- Data In (8 bits) — from SPI interface
- Data Out (8 bits) — to command processor
- Write Enable — SPI interface asserts when writing
- Read Enable — command processor asserts when reading
- Empty flag — tells command processor no data available
- Full flag — tells SPI MCU to hold off

### 3. Command Processor

**Purpose:** Reads commands from the FIFO, decodes them and drives the VRAM with pixel writes to execute drawing operations.

**Implementation:** A state machine built from 74-series logic. Uses EEPROMs as microcode lookup tables. The state machine steps through the algorithm for each command (e.g. Bresenham's line algorithm for line drawing) and generates VRAM addresses and data for each pixel.

**Key components:**
- State counter (74161 cascaded)
- Microcode ROMs (28C256 EEPROMs holding state transition and control signal tables)
- Address calculation registers (74574 registers + 74283 adders)
- Comparators (74688) for loop termination

### 4. VRAM

**Purpose:** Stores the framebuffer. Each byte represents one pixel (8-bit palette index).

**Chip:** 512KB SRAM (IS61C5128AL-10TLI or similar, 10ns access time). Only 307,200 bytes are used for 640x480. Remaining space available for sprites or a second buffer.

**Dual-access strategy:** The VRAM is shared between the scanout engine (reading) and the command processor (writing). Access is interleaved using clock doubling:

```
Pixel Clock:   ____╱‾‾‾‾╲____╱‾‾‾‾╲____╱‾‾‾‾╲____
VRAM Clock:    _╱‾╲_╱‾╲_╱‾╲_╱‾╲_╱‾╲_╱‾╲_╱‾╲_╱‾╲_
Access:        [SCAN][DRAW][SCAN][DRAW][SCAN][DRAW]
```

On one phase the scanout engine reads. On the other phase the command processor reads or writes. VRAM clock runs at 2x pixel clock (~50.35 MHz). A 74157 multiplexer switches between the scanout address and command processor address on alternate phases.

### 5. VGA Scanout Engine

**Purpose:** Continuously reads pixel data from VRAM and converts it to VGA analog signals.

**Implementation:** Horizontal and vertical counters (74161 cascaded) generate current pixel coordinates. These are converted to a VRAM address. The pixel byte read from VRAM indexes into the palette RAM.

**Key components:**
- Horizontal counter (74161 x3, counts 0-799)
- Vertical counter (74161 x3, counts 0-524)
- Sync generation (combinational logic from counter values)
- Address calculation (Y * 640 + X using shift-and-add or lookup ROM)

### 6. Palette RAM

**Purpose:** Converts 8-bit pixel indices to analog RGB values.

**Implementation:** A small fast SRAM used as a lookup table. The 8-bit pixel value becomes the address. Output is 18-bit RGB (6 bits per channel). The 6-bit values feed three R-2R resistor ladder DACs producing the 0-0.7V analog VGA signals.

**Palette writes:** The host can update palette entries via SPI command, enabling color cycling and per-application color schemes.

### 7. Timing Generator

**Purpose:** Master pixel clock and all derived timing signals.

**Implementation:** 25.175 MHz crystal oscillator module. A frequency doubler or separate 50 MHz oscillator generates the VRAM clock. Active/blanking signals derived from horizontal and vertical counters.

## Data Flow

1. Host sends draw command over SPI
2. SPI MCU writes command bytes into FIFO
3. Command processor reads FIFO, decodes command
4. Command processor calculates pixel addresses, writes pixel values to VRAM
5. Scanout engine continuously reads VRAM at pixel clock rate
6. Each pixel byte indexes into palette RAM
7. Palette output feeds R-2R DACs
8. DAC analog outputs drive VGA cable

## Design Risks

| Risk | Mitigation |
|------|------------|
| 50 MHz VRAM clock too fast for discrete logic | Use 74AC-series (3-5ns propagation), careful PCB layout with ground plane |
| SRAM timing violations | Select 10ns SRAM, verify margins with oscilloscope |
| PCB signal integrity at 50 MHz | 4-layer PCB, proper decoupling, controlled impedance traces |
| Y*640+X address calculation too slow | Use scanline base lookup ROM (Y*640 pre-calculated in EEPROM) |

## Dual-Mode Operation

The GPU supports two connection modes selected by a jumper:

- **SPI mode:** Commands arrive from a Raspberry Pi via SPI. The ATmega bridges SPI to the FIFO.
- **Bus mode:** Commands arrive from the homebrew CPU via the system bus. The system bus writes directly to command registers, bypassing the ATmega.

This allows GPU development and testing with a Pi before the CPU exists.
