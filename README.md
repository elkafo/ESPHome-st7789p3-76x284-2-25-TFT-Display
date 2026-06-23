# ESPHome – ST7789P3 76×284 Display

Working ESPHome configuration for the **ST7789P3 76×284 px TFT display**
(the narrow bar-style display sold on AliExpress, e.g. item 1005008920033631).

This took considerable trial-and-error to get working. Key findings are documented here so nobody else has to repeat the process.

This file was written by Claude. I thought this is an easy way to share my findings which where also assisted bei Claude. 

---

## Hardware

| Property | Value |
|---|---|
| Resolution | 76 × 284 px |
| Colors | 262K (16-bit RGB565) |
| Driver IC | ST7789P3 |
| Interface | SPI |
| Backlight | Active LOW (inverted PWM) |

---

## Key findings

### 1. Use `mipi_spi` with `CUSTOM` model
The older `st7789v` platform does **not** work with this panel.
Use `platform: mipi_spi` with `model: CUSTOM` and a full `init_sequence`.

### 2. SPI Mode 3
This panel requires `spi_mode: mode3`. Mode 0 (default) produces a white screen.

### 3. MADCTL / orientation
The panel must be addressed in **portrait mode** by the controller (width=76, height=284).
LVGL rotation is handled separately with `rotation: 270°` in the `lvgl:` block.
MADCTL register `0x36` must be `0x00` (no hardware rotation).

### 4. CGRAM offsets
The ST7789P3 has an internal 240×320 framebuffer.
The 76×284 visible window sits at:

```
offset_width:  82   # (240 - 76) / 2 = 82
offset_height: 0
pad_height:    18   # 320 - 284 - 18 = 18 invisible rows below
```

`pad_height: 18` is essential – without it, 18 pixels of uninitialised VRAM
appear on the right edge of the display (after LVGL rotation).

### 5. Do NOT include SLPOUT / DISPON / INVON in init_sequence
`mipi_spi` appends these automatically. Including them causes display issues.
Also do not include COLMOD (0x3A) – it is set automatically and triggers a warning.

### 6. Backlight is active LOW
```yaml
output:
  - platform: ledc
    pin: GPIO_BL
    id: backlight_pwm
    inverted: true   # ← required
```

---

## ESPHome configuration

### `display_st7789p3_76x284.yaml`

Drop this file in your ESPHome config directory and include it via `packages:`.
Adjust the pin substitutions to match your wiring.

```yaml
packages:
  display: !include display_st7789p3_76x284.yaml
```

See [`display_st7789p3_76x284.yaml`](display_st7789p3_76x284.yaml) in this repo.

---

## Tested environment

- ESPHome **2026.5.x**
- ESP32 (classic, not S3), 4 MB Flash, PSRAM
- `framework: esp-idf`, version: recommended
- LVGL via ESPHome native integration

---

## Wiring example (adjust to your board)

| Display pin | ESP32 GPIO |
|---|---|
| SCL (CLK) | GPIO5 |
| SDA (MOSI) | GPIO23 |
| CS | GPIO15 |
| DC | GPIO2 |
| RST | GPIO4 |
| BLK (Backlight) | GPIO27 |

---

## License

MIT – do whatever you want with it.
