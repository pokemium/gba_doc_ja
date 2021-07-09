# 概要

## DS Display Dimensions / Timings

Dot clock = 5.585664 MHz (=33.513982 MHz / 6)

H-Timing: 256 dots visible, 99 dots blanking, 355 dots total (15.7343KHz)
V-Timing: 192 lines visible, 71 lines blanking, 263 lines total (59.8261 Hz)

The V-Blank cycle for the 3D Engine consists of the 23 lines, 191..213.

Screen size 62.5mm x 47.0mm (each) (256x192 pixels)

Vertical space between screens 22mm (equivalent to 90 pixels)

## 400006Ch - NDS9 - MASTER_BRIGHT - 16bit - Master Brightness Up/Down

画面の明るさを決めるレジスタ

ビット | 内容
-- | -- 
0-4 | Factor(後述, 0-16で16より大きい場合は16として扱う)
5-13 | 不使用
14-15 | モード(0=無効, 1=Up, 2=Down, 3=Reserved)
16-31 | 不使用

明るさは次のように決定します。

モード(bit14-15)が

- 1(Up)のとき: `New = Old + ((63-Old) * Factor/16)`
- 2(Down)のとき: `New = Old - (Old * Factor/16)`

## DISPSTAT/VCOUNT

The LY and LYC values are in range 0..262, so LY/LYC values have been expanded to 9bit values: LY = VCOUNT Bit 0..8, and LYC=DISPSTAT Bit8..15,7.

VCOUNT register is write-able, allowing to synchronize linked DS consoles.

For proper synchronization:

- write new LY values only in range of 202..212
- write only while old LY values are in range of 202..212

DISPSTAT/VCOUNT supported by NDS9 (Engine A Ports, without separate Engine B Ports), and by NDS7 (allowing to synchronize NDS7 with display timings).

Similar as on GBA, the VBlank flag isn't set in the last line (ie. only in lines 192..261, but not in line 262).

Although the drawing time is only 1536 cycles (256*6), the NDS9 H-Blank flag is "0" for a total of 1606 cycles (and, for whatever reason, a bit longer, 1613 cycles in total, on NDS7).

## VRAM Waitstates

The display controller performs VRAM-reads once every 6 clock cycles, a 1 cycle waitstate is generated if the CPU simultaneously accesses VRAM. 

With capture enabled, additionally VRAM-writes take place once every 6 cycles, so the total VRAM-read/write access rate is then once every 3 cycles.

## DS Window Glitches

The DS counts scanlines in range 0..262 (0..106h), of which only the lower 8bit are compared with the WIN0V/WIN1V register settings. 

Respectively, Y1 coordinates 00h..06h will be triggered in scanlines 100h-106h by mistake. That means, the window gets activated within VBlank period, and will be active in scanline 0 and up (that is no problem with Y1=0, but Y1=1..6 will appear as if if Y1 would be 0). 

Workaround would be to disable the Window during VBlank, or to change Y1 during VBlank (to a value that does not occur during VBlank period, ie. 7..191).

Also, there's a problem to fit the 256 pixel horizontal screen resolution into 8bit values: X1=00h is treated as 0 (left-most), X2=00h is treated as 100h (right-most). 

However, the window is not displayed if X1=X2=00h; the window width can be max 255 pixels.

## 2D Engines

Includes two 2D Engines, called A and B. Both engines are accessed by the ARM9 processor, each using different memory and register addresses:

```
  Region______Engine A______________Engine B___________
  I/O Ports   4000000h              4001000h
  Palette     5000000h (1K)         5000400h (1K)
  BG VRAM     6000000h (max 512K)   6200000h (max 128K)
  OBJ VRAM    6400000h (max 256K)   6600000h (max 128K)
  OAM         7000000h (1K)         7000400h (1K)
```

Engine A additionally supports 3D and large-screen 256-color Bitmaps, plus main-memory-display and vram-display modes, plus capture unit.

## Viewing Angles

The LCD screens are best viewed at viewing angles of 90 degrees. Colors may appear distorted, and may even become invisible at other viewing angles.
When the console is handheld, both screens can be turned into preferred direction. 

When the console is settled on a table, only the upper screen can be turned, but the lower screen is stuck into horizontal position - which results in rather bad visibility (unless the user moves his/her head directly above of it).

## 4000070h - NDS9 - TVOUTCNT - Unknown (W)

```
  Bit0-3  "COMMAND"  (?)
  Bit4-7  "COMMAND2" (?)
  Bit8-11 "COMMAND3" (?)
```

This register has been mentioned in an early I/O map from Nintendo, as far as I know, the register isn't used by any games/firmware/bios, not sure if it does really exist on release-version, or if it's been prototype stuff...?

## DS-Lite Screens

The screens in the DS-Lite seem to allow a wider range of vertical angles.

The bad news is that the colors of the DS-Lite are (no surprise) not backwards compatible with older NDS and GBA displays. The good news is that Nintendo has finally reached near-CRT-quality (without blurred colors), so one could hope that they won't show up with more displays with other colors in future.

Don't know if there's an official/recommended way to detect DS-Lite displays (?) possible methods would be whatever values in Firmware header, or by functionality of Power Managment device, or (not too LCD-related) by Wifi Chip ID.
