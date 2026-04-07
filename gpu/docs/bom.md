# GPU Module — Bill of Materials

Status: Draft. Exact quantities finalized during schematic capture.

## Core ICs

| Qty | Part              | Package  | Description                | Est. Price |
|-----|-------------------|----------|----------------------------|------------|
| 1   | IS61C5128AL-10TLI | TSOP-44  | 512KB 10ns SRAM (VRAM)    | $3-5       |
| 1   | 62256 or similar  | DIP-28   | 32KB SRAM (palette RAM)   | $1-2       |
| 2   | 28C256            | DIP-28   | 32KB EEPROM (microcode)   | $4-6 each  |
| 1   | ATmega328P        | DIP-28   | SPI interface MCU          | $2-3       |

## 74-series Logic (AC or HCT)

### Counters and Registers

| Qty | Part    | Description                   | Used For                     |
|-----|---------|-------------------------------|------------------------------|
| 6   | 74AC161 | 4-bit synchronous counter     | H counter (3), V counter (3) |
| 8   | 74AC574 | 8-bit D register              | Command processor registers   |
| 2   | 74AC573 | 8-bit D latch                 | Address latches               |

### Arithmetic

| Qty | Part    | Description          | Used For            |
|-----|---------|----------------------|---------------------|
| 5   | 74AC283 | 4-bit binary adder   | Address calculation |

### Multiplexers and Buffers

| Qty | Part    | Description              | Used For                |
|-----|---------|--------------------------|-------------------------|
| 5   | 74AC157 | Quad 2-to-1 mux          | VRAM address muxing     |
| 4   | 74AC245 | Octal bus transceiver     | Bus isolation           |
| 2   | 74AC244 | Octal buffer (tri-state)  | Data bus drivers        |

### Decoders and Glue Logic

| Qty | Part    | Description          | Used For              |
|-----|---------|----------------------|-----------------------|
| 2   | 74AC688 | 8-bit comparator     | Sync timing           |
| 1   | 74AC138 | 3-to-8 decoder       | Command decoding      |
| 2   | 74AC00  | Quad NAND            | Glue logic            |
| 2   | 74AC08  | Quad AND             | Glue logic            |
| 2   | 74AC32  | Quad OR              | Glue logic            |
| 1   | 74AC04  | Hex inverter         | Glue logic            |
| 1   | 74AC74  | Dual D flip-flop     | Clock divider / sync  |

## Clock

| Qty | Part                | Description              | Est. Price |
|-----|---------------------|--------------------------|------------|
| 1   | 25.175 MHz osc      | Crystal oscillator (DIP-8)| $1-2      |
| 1   | 50 MHz osc          | Crystal oscillator (or PLL)| $1-2     |

## Analog / VGA

| Qty  | Part          | Description              | Est. Price  |
|------|---------------|--------------------------|-------------|
| 18   | 1kΩ resistor  | R-2R DAC (R values)      | < $1 total  |
| 18   | 2kΩ resistor  | R-2R DAC (2R values)     | < $1 total  |
| 3    | 75Ω resistor  | VGA output termination   | < $1 total  |
| 1    | DE-15 female  | VGA connector            | $1-2        |

## Power and Decoupling

| Qty  | Part            | Description            | Est. Price  |
|------|-----------------|------------------------|-------------|
| 30   | 100nF ceramic   | Decoupling (per IC)    | < $3 total  |
| 5    | 10uF electro.  | Bulk decoupling        | < $2 total  |
| 1    | AMS1117-3.3     | 3.3V LDO (if needed)  | $0.50       |
| 1    | 7805 or LM2596  | 5V regulator           | $1-3        |

## Connectors

| Qty | Part            | Description                        | Est. Price |
|-----|-----------------|------------------------------------|------------|
| 1   | 2x20 pin header | System bus connector               | $1         |
| 1   | 6-pin header    | SPI connection                     | $0.50      |
| 1   | 2-pin header    | Power input                        | $0.50      |
| 1   | 16 MHz crystal  | For ATmega328P                     | $0.50      |
| 1   | 28-pin DIP socket| For ATmega328P                    | $0.50      |

## PCB

| Qty | Part       | Description                    | Est. Price     |
|-----|------------|--------------------------------|----------------|
| 1   | Custom PCB | 4-layer, ~100x150mm estimated  | $5-15 (JLCPCB) |

## Estimated Total: $48-84

## Sourcing Notes

- 74AC-series preferred for VRAM path (3-5ns propagation). 74HCT fine for slower control logic.
- IS61C5128AL available from Mouser/Digikey. Alternatives: IS61WV5128BLL, CY7C1049G.
- 28C256 programmable with TL866II+ or similar.
- JLCPCB/PCBWay for 4-layer PCBs at low cost. Most 74-series parts cheaper on LCSC.
