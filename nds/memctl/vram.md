# VRAM

## 4000240h - NDS7 - VRAMSTAT - 8bit - VRAM Bank Status (R)

 bit  |  内容
---- | ---- 
0   | VRAM C enabled and allocated to NDS7  (0=No, 1=Yes)
1   | VRAM D enabled and allocated to NDS7  (0=No, 1=Yes)
2-7 | 不使用 (常に0)

The register indicates if VRAM C/D are allocated to NDS7 (as Work RAM), ie. if VRAMCNT_C/D are enabled (Bit7=1), with MST=2 (Bit0-2). 

However, it does not reflect the OFS value.

## 4000240h - NDS9 - VRAMCNT_A - 8bit - VRAM-A (128K) Bank Control (W)
## 4000241h - NDS9 - VRAMCNT_B - 8bit - VRAM-B (128K) Bank Control (W)
## 4000242h - NDS9 - VRAMCNT_C - 8bit - VRAM-C (128K) Bank Control (W)
## 4000243h - NDS9 - VRAMCNT_D - 8bit - VRAM-D (128K) Bank Control (W)
## 4000244h - NDS9 - VRAMCNT_E - 8bit - VRAM-E (64K) Bank Control (W)
## 4000245h - NDS9 - VRAMCNT_F - 8bit - VRAM-F (16K) Bank Control (W)
## 4000246h - NDS9 - VRAMCNT_G - 8bit - VRAM-G (16K) Bank Control (W)
## 4000248h - NDS9 - VRAMCNT_H - 8bit - VRAM-H (32K) Bank Control (W)
## 4000249h - NDS9 - VRAMCNT_I - 8bit - VRAM-I (16K) Bank Control (W)

 bit  |  内容
---- | ---- 
0-2 | VRAM MST (VRAM-A,B,H,IではBit2は無視)
3-4 | VRAM Offset (0-3, VRAM-E,H,IではOffsetは使用しない)
5-6 | 不使用
7 | VRAM有効化フラグ (0=無効, 1=有効)

There is a total of 656KB of VRAM in Blocks A-I.

Table below shows the possible configurations.

 VRAM | SIZE | MST | OFS | ARM9, Plain ARM9-CPU Access (so-called LCDC mode)
---- | ---- | ---- | ---- | ---- 
A | 128K | 0 | - | 6800000h-681FFFFh
B | 128K | 0 | - | 6820000h-683FFFFh
C | 128K | 0 | - | 6840000h-685FFFFh
D | 128K | 0 | - | 6860000h-687FFFFh
E | 64K  | 0 | - | 6880000h-688FFFFh
F | 16K  | 0 | - | 6890000h-6893FFFh
G | 16K  | 0 | - | 6894000h-6897FFFh
H | 32K  | 0 | - | 6898000h-689FFFFh
I | 16K  | 0 | - | 68A0000h-68A3FFFh

 VRAM | SIZE | MST | OFS | ARM9, 2D Graphics Engine A, BG-VRAM (max 512K)
---- | ---- | ---- | ---- | ---- 
A,B,C,D | 128K | 1 | 0..3 | 6000000h+(20000h*OFS)
E | 64K | 1 | - | 6000000h 
F,G | 16K | 1 | 0..3 | 6000000h+(4000h\*OFS.0)+(10000h\*OFS.1)

 VRAM | SIZE | MST | OFS | ARM9, 2D Graphics Engine A, OBJ-VRAM (max 256K)
---- | ---- | ---- | ---- | ---- 
A,B | 128K | 2 | 0..1 | 6400000h+(20000h*OFS.0)  ;(OFS.1 must be zero)
E   | 64K  | 2 | -    | 6400000h
F,G | 16K  | 2 | 0..3 | 6400000h+(4000h\*OFS.0)+(10000h\*OFS.1)

 VRAM | SIZE | MST | OFS | 2D Graphics Engine A, BG Extended Palette
---- | ---- | ---- | ---- | ---- 
E   | 64K | 4 | -    | Slot 0-3 (下位32KBのみ使用)
F,G | 16K | 4 | 0..1 | Slot 0-1 (OFS=0), Slot 2-3 (OFS=1)

 VRAM | SIZE | MST | OFS | 2D Graphics Engine A, OBJ Extended Palette
---- | ---- | ---- | ---- | ---- 
F,G | 16K | 5 | - | Slot 0 (16KBごとで、下位8KBのみ使用)

 VRAM | SIZE | MST | OFS | Texture/Rear-plane Image
---- | ---- | ---- | ---- | ---- 
A,B,C,D | 128K | 3 | 0..3 | Slot OFS(0-3) ;(Slot2-3: Texture, or Rear-plane)

 VRAM | SIZE | MST | OFS | Texture Palette
---- | ---- | ---- | ---- | ---- 
E   | 64K | 3 | -    | Slots 0-3                 ;OFS=don't care
F,G | 16K | 3 | 0..3 | Slot (OFS.0\*1)+(OFS.1\*4)  ;ie. Slot 0, 1, 4, or 5

 VRAM | SIZE | MST | OFS | ARM9, 2D Graphics Engine B, BG-VRAM (max 128K)
---- | ---- | ---- | ---- | ---- 
C | 128K | 4 | - | 6200000h
H | 32K  | 1 | - | 6200000h
I | 16K  | 1 | - | 6208000h

 VRAM | SIZE | MST | OFS | ARM9, 2D Graphics Engine B, OBJ-VRAM (max 128K)
---- | ---- | ---- | ---- | ---- 
D | 128K | 4 | - | 6600000h
I | 16K  | 2 | - | 6600000h

 VRAM | SIZE | MST | OFS | 2D Graphics Engine B, BG Extended Palette
---- | ---- | ---- | ---- | ---- 
H | 32K | 2 | - | Slot 0-3

 VRAM | SIZE | MST | OFS | 2D Graphics Engine B, OBJ Extended Palette
---- | ---- | ---- | ---- | ---- 
I | 16K | 3 | - | Slot 0 (下位8KBのみ使用)

 VRAM | SIZE | MST | OFS | \<ARM7\>, Plain \<ARM7\>-CPU Access
---- | ---- | ---- | ---- | ---- 
C,D | 128K | 2 | 0..1 | 6000000h+(20000h*OFS.0)  ;OFS.1 must be zero

## 注意

In Plain-CPU modes, VRAM can be accessed only by the CPU (and by the Capture Unit, and by VRAM Display mode). 

In "Plain \<ARM7\>-CPU Access" mode, the VRAM blocks are allocated as Work RAM to the NDS7 CPU.

In BG/OBJ VRAM modes, VRAM can be accessed by the CPU at specified addresses, and by the display controller.

In Extended Palette and Texture Image/Palette modes, VRAM is not mapped to CPU address space, and can be accessed only by the display controller (so, to initialize or change the memory, it should be temporarily switched to Plain-CPU mode).

All VRAM (and Palette, and OAM) can be written to only in 16bit and 32bit units (STRH, STR opcodes), 8bit writes are ignored (by STRB opcode). 

The only exception is "Plain \<ARM7\>-CPU Access" mode: The ARM7 CPU can use STRB to write to VRAM (the reason for this special feature is that, in GBA mode, two 128K VRAM blocks are used to emulate the GBA's 256K Work RAM).

## Other Video RAM

マップ可能なVRAMブロックとは別に、固定アドレスのビデオ関連メモリ領域があります。

```
  5000000h Engine A Standard BG Palette (512 bytes)
  5000200h Engine A Standard OBJ Palette (512 bytes)
  5000400h Engine B Standard BG Palette (512 bytes)
  5000600h Engine B Standard OBJ Palette (512 bytes)
  7000000h Engine A OAM (1024 bytes)
  7000400h Engine B OAM (1024 bytes)
```