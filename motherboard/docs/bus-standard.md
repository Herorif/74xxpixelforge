# System Bus Standard

## Physical Connector

2x20 pin header (40 pins), 2.54mm pitch. Same form factor as Raspberry Pi GPIO header.

## Pin Assignments

| Pin | Name    | Direction     | Description                              |
|-----|---------|---------------|------------------------------------------|
| 1   | D0      | Bidirectional | Data bus bit 0 (LSB)                    |
| 2   | D1      | Bidirectional | Data bus bit 1                           |
| 3   | D2      | Bidirectional | Data bus bit 2                           |
| 4   | D3      | Bidirectional | Data bus bit 3                           |
| 5   | D4      | Bidirectional | Data bus bit 4                           |
| 6   | D5      | Bidirectional | Data bus bit 5                           |
| 7   | D6      | Bidirectional | Data bus bit 6                           |
| 8   | D7      | Bidirectional | Data bus bit 7 (MSB)                    |
| 9   | A0      | From CPU      | Address bus bit 0 (LSB)                 |
| 10  | A1      | From CPU      | Address bus bit 1                        |
| 11  | A2      | From CPU      | Address bus bit 2                        |
| 12  | A3      | From CPU      | Address bus bit 3                        |
| 13  | A4      | From CPU      | Address bus bit 4                        |
| 14  | A5      | From CPU      | Address bus bit 5                        |
| 15  | A6      | From CPU      | Address bus bit 6                        |
| 16  | A7      | From CPU      | Address bus bit 7                        |
| 17  | A8      | From CPU      | Address bus bit 8                        |
| 18  | A9      | From CPU      | Address bus bit 9                        |
| 19  | A10     | From CPU      | Address bus bit 10                       |
| 20  | A11     | From CPU      | Address bus bit 11                       |
| 21  | A12     | From CPU      | Address bus bit 12                       |
| 22  | A13     | From CPU      | Address bus bit 13                       |
| 23  | A14     | From CPU      | Address bus bit 14                       |
| 24  | A15     | From CPU      | Address bus bit 15 (MSB)                |
| 25  | /RD     | From CPU      | Read strobe (active low)                |
| 26  | /WR     | From CPU      | Write strobe (active low)               |
| 27  | /MREQ   | From CPU      | Memory request (active low)             |
| 28  | /IOREQ  | From CPU      | I/O request (active low)                |
| 29  | CLK     | From Mobo     | System clock                             |
| 30  | /RESET  | From Mobo     | System reset (active low)               |
| 31  | /IRQ    | To CPU        | Interrupt request (active low)          |
| 32  | /NMI    | To CPU        | Non-maskable interrupt (active low)     |
| 33  | /CS     | From Mobo     | Chip select for this slot (active low)  |
| 34  | /WAIT   | To CPU        | Wait/ready (active low = wait)          |
| 35  | /BUSREQ | To CPU        | Bus request for DMA (active low)        |
| 36  | /BUSACK | From CPU      | Bus acknowledge (active low)            |
| 37  | +5V     | Power         | 5V supply                                |
| 38  | +5V     | Power         | 5V supply (second pin)                  |
| 39  | GND     | Power         | Ground                                   |
| 40  | GND     | Power         | Ground (second pin)                     |

## Bus Timing

### Read Cycle

```
CLK:     ──╲___╱──╲___╱──╲___╱──
ADDR:    ──< Valid Address >──────
/MREQ:   ─╲_____________________╱
/RD:      ──╲___________________╱─
DATA:    ──────────< Valid Data >─
                    ▲
                    CPU samples here
```

### Write Cycle

```
CLK:     ──╲___╱──╲___╱──╲___╱──
ADDR:    ──< Valid Address >──────
/MREQ:   ─╲_____________________╱
/WR:      ──╲___________________╱─
DATA:    ──< Valid Data >─────────
                         ▲
                         Device latches here
```

## Address Space

| Address Range    | Size | Device        | Decode Logic                   |
|------------------|------|---------------|--------------------------------|
| 0x0000 - 0x7FFF | 32KB | RAM           | A15 = 0                        |
| 0x8000 - 0xBFFF | 16KB | ROM (EEPROM)  | A15=1, A14=0                   |
| 0xC000 - 0xC0FF | 256B | GPU registers | A15=1, A14=1, A13=0, A12=0     |
| 0xC100 - 0xC1FF | 256B | UART          | A15=1, A14=1, A13=0, A12=0, A8=1 |
| 0xC200 - 0xCFFF | 3.5KB| I/O expansion | Varies                          |
| 0xD000 - 0xFFFF | 12KB | Reserved      | Future expansion                |

## GPU Connection Modes

- **Bus mode (jumper set):** GPU responds to reads/writes at 0xC000-0xC0FF via /CS from motherboard.
- **SPI mode (jumper set):** GPU ignores system bus, receives commands via SPI header from Raspberry Pi.

## Design Notes

- All active-low signals prefixed with `/`.
- 5V TTL logic levels. 3.3V devices need level shifters.
- Max 8 device slots before bus drivers (74245) are needed.
- /WAIT allows slow devices to insert wait states.
- /BUSREQ and /BUSACK support DMA (future use).
