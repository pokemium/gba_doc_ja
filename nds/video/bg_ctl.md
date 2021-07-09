# BGMode と BG制御

## 4000000h - NDS9 - DISPCNT

 bit  |  内容
---- | ----
0-2   | A+B | BG Mode
3     | A   | BG0 2D/3D Selection (instead CGB Mode) (0=2D, 1=3D)
4     | A+B | Tile OBJ Mapping        (0=2D; max 32KB, 1=1D; max 32KB..256KB)
5     | A+B | Bitmap OBJ 2D-Dimension (0=128x512 dots, 1=256x256 dots)
6     | A+B | Bitmap OBJ Mapping      (0=2D; max 128KB, 1=1D; max 128KB..256KB)
7-15  | A+B | Same as GBA
16-17 | A+B | Display Mode (Engine A: 0..3, Engine B: 0..1, GBA: Green Swap)
18-19 | A   | VRAM block (0..3=VRAM A..D) (For Capture & above Display Mode=2)
20-21 | A+B | Tile OBJ 1D-Boundary   (see Bit4)
22    | A   | Bitmap OBJ 1D-Boundary (see Bit5-6)
23    | A+B | OBJ Processing during H-Blank (was located in Bit5 on GBA)
24-26 | A   | Character Base (in 64K steps) (merged with 16K step in BGxCNT)
27-29 | A   | Screen Base (in 64K steps) (merged with 2K step in BGxCNT)
30    | A+B | BG Extended Palettes   (0=Disable, 1=Enable)
31    | A+B | OBJ Extended Palettes  (0=Disable, 1=Enable)

## BG Mode

Engine A BG Mode (DISPCNT LSBs) (0-6, 7=Reserved)


Mode | BG0 | BG1 | BG2 | BG3 
-- | -- | -- | -- | -- 
0 | Text/3D | Text | Text     | Text
1 | Text/3D | Text | Text     | Affine
2 | Text/3D | Text | Affine   | Affine
3 | Text/3D | Text | Text     | Extended
4 | Text/3D | Text | Affine   | Extended
5 | Text/3D | Text | Extended | Extended
6 | 3D      | -    | Large    | - 

Of which, the "Extended" modes are sub-selected by BGxCNT bits:

BGxCNT.Bit7 | BGxCNT.Bit2 | Extended Affine Mode Selection
-- | -- | -- 
0  | CharBaseLsb | rot/scal with 16bit bgmap entries (Text+Affine mixup)
1  | 0           | rot/scal 256 color bitmap
1  | 1           | rot/scal direct color bitmap

Engine B: Same as above, except that: Mode 6 is reserved (no Large screen bitmap), and BG0 is always Text (no 3D support).

Affine = formerly Rot/Scal mode (with 8bit BG Map entries)

Large Screen Bitmap = rot/scal 256 color bitmap (using all 512K of 2D VRAM)

## Display Mode (DISPCNT.16-17):

 bit  |  内容
---- | ----
0 | Display off (screen becomes white)
1 | Graphics Display (normal BG and OBJ layers)
2 | Engine A only: VRAM Display (Bitmap from block selected in DISPCNT.18-19)
3 | Engine A only: Main Memory Display (Bitmap DMA transfer from Main RAM)

Mode 2-3 display a raw direct color bitmap (15bit RGB values, the upper bit in each halfword is unused), without any further BG,OBJ,3D layers, these modes are completely bypassing the 2D/3D engines as well as any 2D effects, however the Master Brightness effect can be applied to these modes. 

Mode 2 is particulary useful to display captured 2D/3D images (in that case it can indirectly use the 2D/3D engine).

## BGxCNT

character base extended from bit2-3 to bit2-5 (bit4-5 formerly unused)

```
  engine A screen base: BGxCNT.bits*2K + DISPCNT.bits*64K
  engine B screen base: BGxCNT.bits*2K + 0
  engine A char base: BGxCNT.bits*16K + DISPCNT.bits*64K
  engine B char base: BGxCNT.bits*16K + 0
```

- char base is used only in tile/map modes (not bitmap modes)
- screen base is used in tile/map modes,
- screen base used in bitmap modes as BGxCNT.bits\*16K, without DISPCNT.bits\*64K
- screen base however NOT used at all for Large screen bitmap mode

bgcnt size | text | rotscal | bitmap | large bmp
-- | -- | -- | -- | -- 
0 | 256x256 | 128x128   | 128x128 | 512x1024
1 | 512x256 | 256x256   | 256x256 | 1024x512
2 | 256x512 | 512x512   | 512x256 | -
3 | 512x512 | 1024x1024 | 512x512 | -

bitmaps that require more than 128K VRAM are supported on engine A only.

For BGxCNT.Bit7 and BGxCNT.Bit2 in Extended Affine modes, see above BG Mode description (extended affine doesn't include 16-color modes, so color depth bit can be used for mode selection. Also, bitmap modes do not use charbase, so charbase.0 can be used for mode selection as well).

```
  for BG0CNT, BG1CNT only: bit13 selects extended palette slot
                           (BG0: 0=Slot0, 1=Slot2, BG1: 0=Slot1, 1=Slot3)
```

**Direct Color Bitmap BG, and Direct Color Bitmap OBJ**

BG/OBJ Supports 32K colors (15bit RGB value) - so far same as GBAs BG.

However, the upper bit (Bit15) is used as Alpha flag. That is, Alpha=0=Transparent, Alpha=1=Normal (ie. on the NDS, Direct Color values 0..7FFFh are NOT displayed).

Unlike GBA bitmap modes, NDS bitmap modes are supporting the Area Overflow bit (BG2CNT and BG3CNT, Bit 13).


