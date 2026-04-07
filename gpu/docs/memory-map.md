# GPU Memory Map

## VRAM Layout

512KB (524,288 bytes) SRAM, 19 address lines.

```
0x00000 ┌──────────────────────────────┐
        │   Primary Framebuffer        │
        │   640 x 480 x 1 byte         │
        │   307,200 bytes              │
0x4B000 ├──────────────────────────────┤
        │   Sprite / Tile Data         │
        │   151,488 bytes              │
0x65400 ├──────────────────────────────┤
        │   Free / Second Buffer       │
        │   109,312 bytes              │
0x7FFFF └──────────────────────────────┘
```

### Pixel Layout

Row-major, left to right, top to bottom:

```
0x00000: Pixel (0, 0)     — top-left
0x00001: Pixel (1, 0)
...
0x0027F: Pixel (639, 0)   — end of row 0
0x00280: Pixel (0, 1)     — start of row 1
...
0x4AFFF: Pixel (639, 479) — bottom-right
```

**Address formula:** `address = (y * 640) + x`

## Palette RAM

Separate small SRAM, not part of main VRAM. 256 entries, 3 bytes each.

| Address | Content              |
|---------|----------------------|
| 0x000   | Entry 0: Red[5:0]   |
| 0x001   | Entry 0: Green[5:0] |
| 0x002   | Entry 0: Blue[5:0]  |
| 0x003   | Entry 1: Red[5:0]   |
| ...     | ...                  |
| 0x2FF   | Entry 255: Blue[5:0]|

Total: 768 bytes. Only lower 6 bits of each byte used (0-63).

## GPU Internal Registers

Physical 74-series register chips, written via SPI commands or system bus.

| Register          | Width  | Purpose                               |
|-------------------|--------|---------------------------------------|
| CMD_REG           | 8-bit  | Current command being executed         |
| X0_REG            | 16-bit | X coordinate parameter 0              |
| Y0_REG            | 16-bit | Y coordinate parameter 0              |
| X1_REG            | 16-bit | X coordinate parameter 1              |
| Y1_REG            | 16-bit | Y coordinate parameter 1              |
| COLOR_REG         | 8-bit  | Drawing color (palette index)          |
| ADDR_REG          | 19-bit | Current VRAM write address             |
| DX_REG            | 16-bit | Delta X (Bresenham)                   |
| DY_REG            | 16-bit | Delta Y (Bresenham)                   |
| ERR_REG           | 16-bit | Error accumulator (Bresenham)          |
| STATUS_REG        | 8-bit  | GPU status flags                       |
| SCANLINE_BASE_REG | 19-bit | Base address of current scanline       |

## Address Calculation

### Shift-and-Add Method

640 = 512 + 128, so Y * 640 = (Y << 9) + (Y << 7):

```
Y bits:    Y8 Y7 Y6 Y5 Y4 Y3 Y2 Y1 Y0

Y << 9:    Y8 Y7 Y6 Y5 Y4 Y3 Y2 Y1 Y0  0  0  0  0  0  0  0  0  0
Y << 7:     0  0 Y8 Y7 Y6 Y5 Y4 Y3 Y2 Y1 Y0  0  0  0  0  0  0  0

Add both → 19-bit scanline base address
Then add X (10-bit) → final pixel address
```

Five cascaded 74283 4-bit adders needed.

### Lookup ROM Method (recommended)

Two 28C256 EEPROMs pre-programmed with Y * 640 for Y = 0..479. Y counter (9 bits) is the address input. Data output is the 19-bit base address split across two chips (high byte + low byte). Then add X with 74283 adders.

Fewer chips, easier to debug, and the EEPROM data is generated once by a script.
