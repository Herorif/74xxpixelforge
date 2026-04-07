# GPU Command Set

## SPI Protocol

All commands follow a byte-oriented protocol: `[Command Byte] [Parameter Bytes...]`. The GPU knows how many parameter bytes to expect for each command.

| Parameter      | Value                  |
|----------------|------------------------|
| SPI Mode       | Mode 0 (CPOL=0, CPHA=0)|
| Max Clock      | 20 MHz                 |
| Byte Order     | MSB first              |
| CS Active      | Low                    |

## Commands

### 0x01 — NOP

No operation. Used for synchronization or padding.

| Byte | Value |
|------|-------|
| 0    | 0x01  |

### 0x02 — Clear Screen

Fills the entire framebuffer with a single palette index.

| Byte | Value       |
|------|-------------|
| 0    | 0x02        |
| 1    | Color index |

### 0x03 — Set Pixel

| Byte | Value         |
|------|---------------|
| 0    | 0x03          |
| 1    | X high byte   |
| 2    | X low byte    |
| 3    | Y high byte   |
| 4    | Y low byte    |
| 5    | Color index   |

X range: 0-639, Y range: 0-479.

### 0x04 — Draw Line

Bresenham's line algorithm from (X0,Y0) to (X1,Y1).

| Byte | Value       |
|------|-------------|
| 0    | 0x04        |
| 1-2  | X0 (16-bit) |
| 3-4  | Y0 (16-bit) |
| 5-6  | X1 (16-bit) |
| 7-8  | Y1 (16-bit) |
| 9    | Color index |

### 0x05 — Draw Rectangle (outline)

| Byte | Value       |
|------|-------------|
| 0    | 0x05        |
| 1-2  | X0 (16-bit) |
| 3-4  | Y0 (16-bit) |
| 5-6  | X1 (16-bit) |
| 7-8  | Y1 (16-bit) |
| 9    | Color index |

### 0x06 — Fill Rectangle

| Byte | Value       |
|------|-------------|
| 0    | 0x06        |
| 1-2  | X0 (16-bit) |
| 3-4  | Y0 (16-bit) |
| 5-6  | X1 (16-bit) |
| 7-8  | Y1 (16-bit) |
| 9    | Color index |

### 0x07 — Blit (Block Transfer)

Copies a rectangular region within VRAM.

| Byte  | Value           |
|-------|-----------------|
| 0     | 0x07            |
| 1-2   | Src X (16-bit)  |
| 3-4   | Src Y (16-bit)  |
| 5-6   | Dst X (16-bit)  |
| 7-8   | Dst Y (16-bit)  |
| 9-10  | Width (16-bit)  |
| 11-12 | Height (16-bit) |

### 0x08 — Scroll

Scrolls the entire framebuffer by (DX,DY). Exposed region filled with specified color.

| Byte | Value              |
|------|--------------------|
| 0    | 0x08               |
| 1-2  | DX (signed 16-bit) |
| 3-4  | DY (signed 16-bit) |
| 5    | Fill color index   |

### 0x09 — Set Palette Entry

| Byte | Value                   |
|------|-------------------------|
| 0    | 0x09                    |
| 1    | Palette index (0-255)   |
| 2    | Red (lower 6 bits)      |
| 3    | Green (lower 6 bits)    |
| 4    | Blue (lower 6 bits)     |

### 0x0A — Bulk Palette Load

| Byte       | Value                |
|------------|----------------------|
| 0          | 0x0A                 |
| 1          | Start index          |
| 2          | Count (1-256, 0=256) |
| 3..3+N*3   | R, G, B triplets     |

### 0x0B — Write Pixels (bulk)

Raw pixel data stream to VRAM.

| Byte       | Value                     |
|------------|---------------------------|
| 0          | 0x0B                      |
| 1-3        | Start address (19-bit)    |
| 4-5        | Length (16-bit)            |
| 6..6+N     | Pixel data (color indices)|

### 0x0C — Draw Character

8x8 character from built-in font ROM.

| Byte | Value            |
|------|------------------|
| 0    | 0x0C             |
| 1-2  | X (16-bit)       |
| 3-4  | Y (16-bit)       |
| 5    | ASCII code       |
| 6    | Foreground color  |
| 7    | Background color  |

### 0x0D — Draw String

Null-terminated string.

| Byte  | Value            |
|-------|------------------|
| 0     | 0x0D             |
| 1-2   | X (16-bit)       |
| 3-4   | Y (16-bit)       |
| 5     | Foreground color  |
| 6     | Background color  |
| 7..N  | ASCII bytes       |
| N+1   | 0x00 (terminator) |

### 0xFE — Read Status

Host sends 0xFE then clocks in one byte:

| Bit | Meaning            |
|-----|--------------------|
| 7   | FIFO full          |
| 6   | FIFO empty         |
| 5   | GPU busy (drawing) |
| 4   | VSync active       |
| 3-0 | Reserved           |

### 0xFF — Reset

Clears VRAM, resets palette to default, clears FIFO.

## Default Palette

On reset the GPU loads a 256-color palette: 16 classic CGA/VGA system colors followed by a 6-8-5 RGB color cube (6 red x 8 green x 5 blue = 240 colors).

## Flow Control

Check the status register (0xFE) before large command sequences. If FIFO full is set, wait. The SPI MCU also holds MISO low while the FIFO is full as hardware flow control.

## Bounds Handling

Out-of-bounds coordinates are silently clipped. The GPU never writes outside the framebuffer region.
