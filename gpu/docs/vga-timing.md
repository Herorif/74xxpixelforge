# VGA Timing Reference — 640x480 @ 60Hz

## Key Parameters

| Parameter             | Value         |
|-----------------------|---------------|
| Pixel Clock           | 25.175 MHz    |
| Pixel Period          | 39.72 ns      |
| Horizontal Frequency  | 31.469 kHz    |
| Vertical Frequency    | 59.94 Hz      |
| Total Pixels/Line     | 800           |
| Total Lines/Frame     | 525           |

## Horizontal Timing (pixels)

```
|<--- 640 --->|<- 16 ->|<- 96 ->|<- 48 ->|
   Visible       Front   HSync     Back
   Area          Porch             Porch
                         ____
HSync: ─────────────────│    │──────────────
                         (active low)
```

| Region         | Pixels | Time (us) |
|----------------|--------|-----------|
| Visible Area   | 640    | 25.42     |
| Front Porch    | 16     | 0.64      |
| HSync Pulse    | 96     | 3.81      |
| Back Porch     | 48     | 1.91      |
| **Total Line** | **800**| **31.78** |

## Vertical Timing (lines)

```
|<--- 480 --->|<- 10 ->|<- 2 -->|<- 33 ->|
   Visible       Front   VSync     Back
   Area          Porch             Porch
                         __
VSync: ─────────────────│  │────────────────
                        (active low)
```

| Region         | Lines   | Time (ms) |
|----------------|---------|-----------|
| Visible Area   | 480     | 15.25     |
| Front Porch    | 10      | 0.32      |
| VSync Pulse    | 2       | 0.064     |
| Back Porch     | 33      | 1.05      |
| **Total Frame**| **525** | **16.68** |

## Counter Implementation

### Horizontal Counter

Three cascaded 74161 chips form a 10-bit counter (0-799, then reset).

| Counter Value | Region       | Action               |
|---------------|--------------|-----------------------|
| 0 - 639       | Visible      | Output pixel data    |
| 640 - 655     | Front porch  | Output black         |
| 656 - 751     | HSync pulse  | HSync low            |
| 752 - 799     | Back porch   | Output black         |

### Vertical Counter

Three cascaded 74161 chips, incremented once per line when H counter resets (0-524, then reset).

| Counter Value | Region       | Action               |
|---------------|--------------|-----------------------|
| 0 - 479       | Visible      | Scanout active       |
| 480 - 489     | Front porch  | Output black         |
| 490 - 491     | VSync pulse  | VSync low            |
| 492 - 524     | Back porch   | Output black         |

### Blanking Signal

```
BLANK = (H_counter > 639) OR (V_counter > 479)
```

When BLANK is active, the DAC inputs are forced to 0V using 74245 bus transceivers that disconnect the palette output, with pull-down resistors holding the DAC inputs low.

## VRAM Address Generation

### Y * 640 + X

Since 640 = 512 + 128:

```
Y * 640 = (Y << 9) + (Y << 7)
```

Two shifted copies of the Y counter fed into cascaded 74283 adders. Then add X (10-bit).

### Alternative: Scanline Base ROM

A 28C256 EEPROM programmed with Y * 640 for each Y (0-479). Address input is the 9-bit Y counter. Data output is the 19-bit base address (two EEPROMs, one for high byte, one for low byte). Then add X with 74283 adders.

This approach uses fewer chips and is easier to debug.

### Address Width

Max address = 479 * 640 + 639 = 307,199 = 0x4AFFF (19 bits). Maps directly to the 512KB SRAM's 19 address lines.

## R-2R DAC

Each color channel uses a 6-bit R-2R resistor ladder:

```
Bit 5 (MSB) ──┤ R ├──┬── Analog Out ──► VGA pin
                      │
Bit 4 ────────┤ R ├──┤
              ┤2R├   │
               │     │
Bit 3 ────────┤ R ├──┤
              ┤2R├   │
               │     │
  ... (bits 2, 1, 0)
               │
              GND
```

- R = 1kΩ, 2R = 2kΩ
- 75Ω termination resistor at VGA connector
- 64 levels per channel, ~11mV per step into 75Ω load
- Three identical DACs for R, G, B

## VGA Connector Pinout

| Pin | Signal       |
|-----|--------------|
| 1   | Red (0-0.7V) |
| 2   | Green (0-0.7V)|
| 3   | Blue (0-0.7V)|
| 5   | Ground       |
| 6   | Red Ground   |
| 7   | Green Ground |
| 8   | Blue Ground  |
| 10  | Sync Ground  |
| 13  | HSync (TTL, active low) |
| 14  | VSync (TTL, active low) |

## Verification Checklist

Before connecting to a monitor, verify with oscilloscope:

1. HSync frequency: 31.469 kHz (±0.5%)
2. VSync frequency: 59.94 Hz (±0.5%)
3. HSync pulse width: ~3.8 us
4. VSync pulse width: ~64 us
5. RGB signals: 0-0.7V during visible, 0V during blanking
6. No ringing or overshoot on sync signals

Most modern LCD monitors tolerate timing variations up to a few percent.
