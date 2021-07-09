# DS Files - 2D Video

例: DSiランチャー `"rom:\layout\cmn\launcher_d.szs\.."`

## Nitro Color Palette

**Header**

```
  000h 4    Chunk ID "RLCN" (aka NCLR backwards, Nitro Color Resource)
  004h 2    Byte Order    (FEFFh)
  006h 2    Version       (0100h)
  008h 4    Total Filesize
  00Ch 2    Offset to "TTLP" Chunk, aka Size of "RLCN" Chunk (0010h)
  00Eh 2    Total number of following Chunks (1=TTLP) (or 2=TTLP+PMCP ?)
```

**TTLP Chunk**

```
  000h 4    Chunk ID "TTLP" (aka PLTT backwards, Palette data)
  004h 4    Chunk Size (eg. 0218h)
  008h 4    Reportedly Color Depth (ie. "tile usage info") (3=4bpp, 4=8bpp)
  00Ch 4    Zero
  010h 4    Palette Data Size in bytes (eg. 200h) (or 200h-N? no, blah!)
  014h 4    Offset from TTLP+8 to Palette Data? (always 10h)
  018h N*2  Palete Data (16bit colors, 0000h..7FFFh)
```

Most DSi titles use full 200h-byte palettes (Paper Plane has a smaller one in Graphics.NARC\Seq\pause.zcl). There seem to be no DSi titles with PMCP chunks.

PMCP Chunk (存在しないこともあるらしい...？)

```
  000h 4    Chunk ID "PMCP" (aka PCMP backwards, Palette CMP?)
  004h 4    Chunk Size (reportedly always 12h ???)
  008h 2    Number of palettes in file (uh?)
  00Ah 2    Unused (BEEFh=Bullshit)
  00Ch 4    Offset from PMCP+8 to Palette IDs? (always 08h)
  DATA N*2  "Palette ID numbers for each palette (starting from 0)"
```

## Nitro Character Tiles

参考: 

- DSiランチャー `"rom:\debug\DebugFont.NCGR" `        -- with SOPC chunk
- DSiランチャー `"rom:\layout\cmn\launcher_d.szs\.."` -- without SOPC chunk

**Header**

```
  000h 4    Chunk ID "RGCN" (aka NCGR backwards, Nitro Char Graphics Resource)
  004h 2    Byte Order    (FEFFh)
  006h 2    Version       (0101h) (unknown if 0100h does also exist?)
  008h 4    Total Filesize
  00Ch 2    Offset to "RAHC" Chunk, aka Size of "RGCN" Chunk (0010h)
  00Eh 2    Total number of following Chunks (1=RAHC, or 2=RAHC+SOPC)
```

**RAHC Chunk**

```
  000h 4    Chunk ID "RAHC" (aka CHAR backwards)
  004h 4    Chunk Size (eg. 1420h)
  008h 2    Tile Data Size in Kilobytes   ;\or both set to FFFFh
  00Ah 2    Unknown (always 20h)          ;/(when size<>N*1024)
  00Ch 4    Color Depth (3=4bpp, 4=8bpp)
  010h 2    Zero   ;or 10h (when SOPC not exists? kbyte size rounded up?)
  012h 2    Zero   ;or 20h (when SOPC not exists?)
  014h 4    Zero
  018h 4    Tile Data Size in Bytes (eg. 1400h)
  01Ch 4    Offset from RAHC+8 to Tile Data?  ;=always 18h
  020h ...  Tile Data (eg. 20h-byte zerofilled for 4bpp SPC char?)
```

Nonzero \[10h,12h\] spotted in Paper Plane "rom:\Graphics.NARC\Plane\plane.zcg".

**SOPC Chunk (Tile Data sizeが1024バイトの倍数の時のみ存在)**

```
  000h 4    Chunk ID "SOPC" (aka CPOS backwards)
  004h 4    Chunk Size (10h)
  008h 4    Zero
  00Ch 2    Same as [00Ah] in RAHC chunk? (always 20h)
  00Eh 2    Same as [008h] in RAHC chunk? (size in kilobytes)
```

## Unknown Character Tiles

Apart from above NCGR, there is reportedly another tile format:

```
  NCGR (Nitro Character Graphic Resource) - Graphical Tiles --> see above
  NBGR (Nitro Basic Graphic Resource)     - Graphical Tiles --> what ???
```

If it does really exist for real... the header ID be "RGBN" (aka NBGR backwards), and file extension might be NBGR? But even if so, it's unknown if/when/where/why that NBGR format is used. If the "B" is for "Basic" then might have less features than NCGR, or maybe it might be "B" for non-tiled Bitmaps, or whatever?

## Nitro BG Maps Screens

**Header**

```
  000h 4    Chunk ID "RCSN" (aka NSCR backwards, Nitro Screen Resource)
  004h 2    Byte Order    (FEFFh)
  006h 2    Version       (0100h)
  008h 4    Total Filesize
  00Ch 2    Offset to "NRCS" Chunk, aka Size of "RCSN" Chunk (0010h)
  00Eh 2    Total number of following Chunks (1=NRCS)
```

**NRCS Chunk**

```
  000h 4    Chunk ID "NRCS" (aka SCRN backwards, Screen)
  004h 4    Chunk Size
  008h 4    Screen Width in pixels
  00Ah 2    Screen Height in pixels
  00Ch 4    Zero
  010h 4    Screen Data Size (width/8)*(height/8)*2
  014h N*2  Screen Data (16bit BG Map entries, palette+xyflip+tileno)
```

## Nitro OBJ Animations 

**Header**

```
  000h 4    Chunk ID "RNAN" (aka NANR backwards, Nitro Animation Resource)
  004h 2    Byte Order    (FEFFh)
  006h 2    Version       (0100h)
  008h 4    Total Filesize
  00Ch 2    Offset to "KNBA" Chunk, aka Size of "RNAN" Chunk (0010h)
  00Eh 2    Total number of following Chunks (1=KNBA, or 3=KNBA+LBAL+TXEU)
```

One chunk exists in DSi Launcher.

Three chunks exist in DSi Flipnote `"rom:ManualData.Eu\md2res_narc.blz\data\obj"`

**KNBA Chunk**

```
  000h 4    Chunk ID "KNBA" (aka ABNK backwards, Animation Bank)
  004h 4    Chunk Size (always padded to 4-byte boundary if LABL chunk follows)
  008h 2    Number of 16-byte Animation Blocks ;implies NumLabels in LABL chunk
  00Ah 2    Number of 8-byte Frame Blocks
  00Ch 4    Offset from KNBA+8 to Animation Blocks ;=18h
  010h 4    Offset from KNBA+8 to Frame Blocks     ;=[0Ch]+[08h]*10h
  014h 4    Offset from KNBA+8 to Frame Data       ;=[10h]+[0Ah]*8
  018h 8    Zero
  DATA ..   Animation Blocks (16-byte entries)
   00h 4      Number of Frames
   04h 2      Unknown        (0)
   06h 2      Unknown Always (1)   ;reportedly "always unknown"
   08h 4      Unknown        (1..2)
   0Ch 4      Offset from FrameBlock+0 to First Frame
  DATA ..   Frame Blocks (8-byte entries)
   00h 4      Offset from FrameData+0 to whatever?   (always 4-byte aligned?)
   04h 2      Frame Width   ;3Ch or 01..06h   ;Time in 60Hz units? num meta's?
   06h 2      Unused (usually 0000h, or BEEFh=Bullshit)
  DATA ..   Frame Data (2-byte entries)
   00h 2      Unknown 16bit values? (maybe CELL index or whatever??)  (CCCCh=?)
```

**LBAL Chunk**

```
  000h 4    Chunk ID "LBAL" (aka LABL backwards, Labels)
  004h 4    Chunk Size (not padded to 4-byte size, following TXEU is unaligned)
  008h 4*N  Offsets from LabelArea+0 to Labels (for each Animation Block)
  ...  ..   Label Area (ASCII Strings, terminated by 00h)
```

The LabelArea starts at LBAL+8+NumLabels*4 (whereas, NumLabels is found in KNBA chunk).

**TXEU Chunk**

Caution: Not 4-byte aligned (the preceeding LBAL chunk can have odd size).

```
  000h 4    Chunk ID "TXEU" (aka UEXT backwards, Whatever Extension or so?)
  004h 4    Chunk Size (0Ch)
  008h 4    Unknown (usually 0) (reportedly 0 or 1)
```

## Nitro OBJ Metatile Cells

**Header**

```
  000h 4    Chunk ID "RECN" (aka NCER backwards, Nitro Cell Resource)
  004h 2    Byte Order    (FEFFh)
  006h 2    Version       (0100h)
  008h 4    Total Filesize
  00Ch 2    Offset to "KBEC" Chunk, aka Size of "RECN" Chunk (0010h)
  00Eh 2    Total number of following Chunks (1=KBEC, or 3=KBEC+LBAL+TXEU)
```

**KBEC Chunk**

```
  000h 4    Chunk ID "KBEC" (aka CEBK backwards, Cell Bank)
  004h 4    Chunk Size (always padded to 4-byte boundary if LABL chunk follows)
  008h 2    Number of Metatiles
  00Ah 2    Metatiles Entry Size (0=Normal 8 bytes, 1=Extended 16 bytes)
              (DSi Launcher ..layout\cmn\launcher_u\.. uses 16-byte size)
  00Ch 4    Offset from KBEC+8 to Metatile Table? (18h)
  010h 4    Boundary Size (?)       (but is ZERO in layout\cmn\launcher_u\)
               "Specifies the area in which the image can be drawn,
               multiplied by 64, ie. 2 means that the area is 128x128 pixels."
  014h 0Ch  Zero
  020h ..   Metatile Table (8 bytes each) (or 16 bytes)
  ...  ..   OBJ Attribute Table (6-bytes each)
```

Metatile Table entries are (8-byte or 16-byte):

```
  000h 2    Number of OBJs
  002h 2    Unknown
  004h 4    OBJ Data Offset (from begin of OBJ Attr Table)
 (008h 2)   Unknown (can be 02h,10h,48h,74h)
 (00Ah 2)   Unknown (can be 08h)
 (00Ch 2)   Unknown (can be FFA0h..FFF0h) ;\maybe extra coordinate offsets?
 (00Eh 2)   Unknown (can be FFF0h..FFF9h) ;/
```

OBJ Attribute Table starts at Number of `Cells * 8` and each cell is made up of 6 bytes.

The 6-byte OBJ Attributes seem to be in normal OAM format, containing the coordinates, tile number, tile size, and other flags (however, the coordinates contain signed values; ie. one needs to add the current OBJ position to those values). For details on the OBJ Attributes, see: [LCD OBJ](../../lcd/sprite.md)

**LBAL and TEXU Chunks (if any)**

Same as in Animation files (see there). In fact, the content seems to be SAME as the corresponding Animation file (for pairs of filename.NANR and filename.NCER), and the number of labels must be obtained from the NANR file's KNBA chunk (as such, it's rather useless to have LBAL in NCER files, except perhaps for error checking that the correct file pair was loaded).

Note: DSi Launcher layout\cmn\logodemo.szs has KBEC Chunk Size 2B6h (although the filesize is padded as if it were 2B8h bytes) (the file has no LBAL chunk, so it's unclear if/how it were aligned if present).

## Nitro Unknown Files

DSi Deep Psyche has two extra file types:

- .NMAR file (with "RAMN" header ID, and "KNBA"+"LBAL" chunks)
- .NMCR file (with "RCMN" header ID, and "KBCM" chunk)

The purpose is unknown, but they are probably also animating something...

```
  OBJ with 16bit x/y (instead 9bit/8bit)?
  OBJ with fractional x/y-stepping (moving/motion)?
  OBJ rotation/scaling?
  BG scroll offsets?
  BG tile replacement?
```

Going by the filename (`a01_obj01.NMAR`) they seem to be OBJ related. Labels include things like "a01_upNN" and "a01_windowNN".

The chunks seem to resemble those in RNAN/RECN files (Animation+Cells). 

Except, the KBCM has 8-byte entries (unlike the 6-byte ones in KBEC):

```
  000h 2    Unknown (000xh..007Ah, maybe time or so?)
  002h 2    Unknown (signed 16bit?)
  004h 2    Unknown (signed 16bit?)
  006h 2    Unknown (0x21h, with x=0..8)
```

## Nitro More Unknown Files

Some 2D folders contain more unknown files (eg. DSi Camera `"rom:layout\cmn\fusion_camera.szs"`):

**JNBL (whatever, with .bnbl extension)**

```
  000h 4     ID "JNBL"
  004h 2     Zero
  006h 2     Number of 6-byte entries (01h or more)
  008h N*6   Unknown
```

**JNCL (whatever, with .bncl extension)**

```
  000h 4     ID "JNCL"
  004h 2     Zero (0000h)
  006h 2     Number of 8-byte entries (01h or more)
  008h N*8   Unknown (eg. 80h,10h,C0h,20h,00h,00h,00h,00h)
```

**JNLL (whatever, with .bnll extension)**

```
  000h 4     ID "JNLL"
  004h 2     Zero (0000h)
  006h 2     Number of 16-byte entries (01h or more)
  008h N*16  Unknown (eg. 80h,50h,60h,10h,7Ch,29h,FFh,FDh,0Dh,19h,0,0,0,0,0,0)
```

**BNGL (whatever, with .bngl extension)**

```
  000h 4   ID "BNGL"   ;this same as file extension (not JNGL)
  004h 2   Zero (0000h)
  006h 2   Number of ?-byte entries (01h or more)
  008h 2   Unknown (can be 02h,04h,06h,0Ah)
  00Ah 2   Number of ?-byte other entries maybe (01h or more)
  ...
  ...      Entries?
  ...      Other Entries?
  ...      Maybe More Other Entries?
```

Filesize can range from 18h bytes to 24Ah bytes (or maybe yet smaller/biggger).

## .ntft and .ntfp

**.ntft file**

Probably texture data (maybe for use as extra 2D layer), size is usually/always a power of 2 (ranging from 80h bytes to at least 64Kbytes (or even 512Kbytes?). Color depth can be 16bit or 8bit (and maybe less). Files with less than 16bit are bundled with a .ntfp palette file.

**.ntfp file**

Probably texture palette, with 16bit color numbers. The files can be quite small (eg. only 6 or 8 bytes).

## .wmif and .wmpf

DSi Sudoku `rom:\Textures\` has "Wild Magic" .wmif and .wmpf (image+palette) files.

**.wmif file**

```
  000h 1Bh  ID "Wild Magic Image File 3.00",00h
  01Bh 4    Palette Filename Length (eg. 0Dh)
  01Fh LEN  Palette Filename        (eg. "BG_Board.wmpf")
  ...  4    Texture Format (6=4bpp, 7=8bpp)
  ...  4    Texture Width in pixels
  ...  4    Texture Height in pixels
  ...  ..   Texture data
```

**.wmpf file**

```
  000h 1Dh  ID "Wild Magic Palette File 1.00",00h
  01Dh 4    Zero?
  021h 4    Number of Colors
  025h ..   Colors, 16bit (0000h..7FFFh)
```

