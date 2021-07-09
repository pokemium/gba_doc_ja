# 拡張パレット

When allocating extended palettes, the allocated memory is not mapped to the CPU bus, so the CPU can access extended palette only when temporarily de-allocating it.

Color 0 of all standard/extended palettes is transparent, color 0 of BG standard palette 0 is used as backdrop. extended palette memory must be allocated to VRAM.

BG Extended Palette enabled in DISPCNT Bit 30, when enabled,

```
 standard palette --> 16-color tiles (with 16bit bgmap entries) (text)
                      256-color tiles (with 8bit bgmap entries) (rot/scal)
                      256-color bitmaps
                      backdrop-color (color 0)
 extended palette --> 256-color tiles (with 16bit bgmap entries)(text,rot/scal)
```

Allocated VRAM is split into 4 slots of 8K each (32K used in total), normally BG0..3 are using Slot 0..3, however BG0 and BG1 can be optionally changed to BG0=Slot2, and BG1=Slot3 via BG0CNT and BG1CNT.

OBJ Extended Palette enabled in DISPCNT Bit 31, when enabled,

- 16 colors x 16 palettes: standard palette memory (=256 colors)
- 256 colors x 16 palettes: extended palette memory (=4096 colors)

Extended OBJ palette memory must be allocated to VRAM F, G, or I (which are 16K) of which only the first 8K are used for extended palettes (=1000h 16bit entries).

