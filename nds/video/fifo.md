# Main Memory Display Mode

## 4000068h - NDS9 - DISP_MMEM_FIFO - 32bit - Main Memory Display FIFO (R?/W)

Intended to send 256x192 pixel 32K color bitmaps by DMA directly

```
 - to Screen A             (set DISPCNT to Main Memory Display mode), or
 - to Display Capture unit (set DISPCAPCNT to Main Memory Source).
```

The FIFO can receive 4 words (8 pixels) at a time, each pixel is a 15bit RGB value (the upper bit, bit15, is unused).

Set DMA to Main Memory mode, 32bit transfer width, word count set to 4, destination address to DISP\_MMEM\_FIFO, source address must be in Main Memory.

Transfer starts at next frame.

Main Memory Display/Capture is supported for Display Engine A only.
