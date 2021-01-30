# ビットマップモード

BGモード3-5では背景の描画はタイル形式ではなく、ビットマップ形式でおこないます。

Bitmaps are implemented as BG2, with Rotation/Scaling support. 

As bitmap modes are occupying 80KBytes of BG memory, only 16KBytes of VRAM can be used for OBJ tiles.

## BGモード3 - 240×160px, 32768色

各ピクセルを表すのに2バイト使用し、パレットデータを介さず直接32768色の中から色を指定します。透明色はありません。

 bit  |  内容
---- | ----
0-4   | 赤   (0-31)
5-9   | 緑 (0-31)
10-14 | 青  (0-31)
15    | 使用しない

最初の480バイトは最初の行(1行=240pxなので480バイト)で、次の480バイトは次の行です。以後も同様です。

画面全体を描画するのに必要なメモリ領域は76800バイト(=75KB: `0x0600_0000`-`0x0601_2bff`)で、あらかじめ用意された80KBのフレームバッファのほとんどを使用します。

## BGモード4 - 240×160px, (32768色のうちの)256色

One byte is associated to each pixel, selecting one of the 256 palette entries. Color 0 (backdrop) is transparent, and OBJs may be displayed behind the bitmap.

The first 240 bytes define the topmost line, the next 240 the next line, and so on. 

The background occupies 37.5 KBytes, allowing two frames to be used (06000000-060095FF for Frame 0, and 0600A000-060135FF for Frame 1).

## BGモード5 - 160×128px, 32768色

Colors are defined as for Mode 3 (see above), but horizontal and vertical size are cut down to 160x128 pixels only - smaller than the physical dimensions of the LCD screen.

The background occupies exactly 40 KBytes, so that BG VRAM may be split into two frames (06000000-06009FFF for Frame 0, and 0600A000-06013FFF for Frame 1).

In BG modes 4,5, one Frame may be displayed (selected by DISPCNT Bit 4), the other Frame is invisible and may be redrawn in background.
