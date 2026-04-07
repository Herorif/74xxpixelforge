# Design Decisions Log

## DD-001: GPU resolution and color depth

**Decision:** 640x480 @ 60Hz, 8-bit paletted color (256 colors from 18-bit palette)

**Considered:** 320x240 16-bit (easy timing but low res), 640x480 16-bit (600KB VRAM, too much bandwidth for discrete logic), 640x480 4-bit (too limited), monochrome (too restrictive).

**Reasoning:** 640x480 is the baseline VGA every monitor supports. 8-bit paletted keeps VRAM at ~300KB and halves scanout bandwidth vs 16-bit. The programmable 18-bit palette still allows 256 simultaneous colors from 262,144 possible.

## DD-002: SPI MCU for host interface

**Decision:** ATmega328P handles SPI communication.

**Considered:** Full discrete SPI slave (~15-20 chips), FPGA.

**Reasoning:** SPI slave mode with variable-length messages and flow control would be complex and fragile in 74-series logic. The MCU handles only SPI-to-FIFO bridging. All graphics processing is discrete logic.

## DD-003: VRAM interleaved access

**Decision:** Time-division multiplexing at 2x pixel clock.

**Considered:** Dual-port SRAM (expensive, hard to source large), double buffering with page flip (doubles VRAM), priority-based access (simpler but slower drawing).

**Reasoning:** Interleaving is how many real VGA-era chips solved this. Requires 50 MHz VRAM clock and 10ns SRAM, both achievable with modern parts.

## DD-004: System bus design

**Decision:** 8-bit data bus, 16-bit address bus, active-low control signals, memory-mapped I/O. Inspired by Z80/ISA.

**Considered:** 16-bit data bus (more chips), dedicated I/O bus (unnecessary complexity), SPI interconnect (too slow for CPU-GPU comms).

**Reasoning:** Well understood, well documented, proven. 8-bit minimizes CPU chip count. 16-bit addressing gives 64KB which is sufficient. Memory-mapped I/O means GPU and UART use the same read/write instructions as RAM.

## DD-005: GPU dual-mode operation

**Decision:** GPU supports both SPI (to Raspberry Pi) and system bus (to homebrew CPU), selected by jumper.

**Reasoning:** Allows GPU development and testing with a Pi before the CPU exists. In SPI mode the ATmega feeds the FIFO. In bus mode the system bus writes directly to command registers.

## DD-006: 74AC-series for high-speed paths

**Decision:** 74AC for VRAM path and timing-critical logic. 74HCT acceptable for slower control logic.

**Reasoning:** 74AC has 3-5ns propagation, fast enough for the 50 MHz VRAM clock (20ns period). 74HCT is fine for non-critical paths and is 5V TTL-compatible. Mixing families is standard practice.

## DD-007: GPU-first build order

**Decision:** Build the GPU before motherboard or CPU.

**Reasoning:** Testable standalone with a Raspberry Pi. Immediate visual feedback. Forces early bus standard definition. Most rewarding module to see working.

## DD-008: 4-layer PCB

**Decision:** 4-layer PCBs for all modules.

**Considered:** 2-layer (signal integrity problems at 50 MHz), 6-layer (overkill).

**Reasoning:** Signal/ground/power/signal stackup provides ground plane under high-speed traces. Cost difference at Chinese fabs is minimal ($2-5 per board). Essential for signal integrity and EMI at 50 MHz.
