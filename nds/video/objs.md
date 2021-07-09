# OBJs

## DS OBJ Priority

The GBA has been assigning OBJ priority in respect to the 7bit OAM entry number, regardless of the OBJs 2bit BG-priority attribute (which allowed to specify invalid priority orders). That problem has been fixed in DS mode by combining the above two values into a 9bit priority value.

## OBJ Tile Mapping (DISPCNT.4,20-21)

 bit4 | bit20-21 | Dimension Boundary Total | Notes
---- | ---- | ---- | ---- | ---- 
0 | x | 2D | 32  | 32K  | Same as GBA 2D Mapping
1 | 0 | 1D | 32  | 32K  | Same as GBA 1D Mapping
1 | 1 | 1D | 64  | 64K  | --
1 | 2 | 1D | 128 | 128K | --
1 | 3 | 1D | 256 | 256K | Engine B: 128K max

TileVramAddress = `TileNumber * BoundaryValue`

Even if the boundary gets changed, OBJs are kept composed of 8x8 tiles.

## Bitmap OBJ Mapping (DISPCNT.6,5,22)

Bitmap OBJs are 15bit Direct Color data, plus 1bit Alpha flag (in bit15).

Bit6 | Bit5 | Bit22 | Dimension   | Boundary  | Total | Notes
0    | 0    | x     | 2D/128 dots | 8x8 dots  | 128K  | Source Bitmap width 128 dots
0    | 1    | x     | 2D/256 dots | 8x8 dots  | 128K  | Source Bitmap width 256 dots
1    | 0    | 0     | 1D          | 128 bytes | 128K  | Source Width = Target Width
1    | 0    | 1     | 1D          | 256 bytes | 256K  | Engine A only
1    | 1    | x     | Reserved    | --        | --    | -- 

In 1D mapping mode, the Tile Number is simply multiplied by the boundary value.

```
  1D_BitmapVramAddress = TileNumber(0..3FFh) * BoundaryValue(128..256)
  2D_BitmapVramAddress = (TileNo AND MaskX)*10h + (TileNo AND NOT MaskX)*80h
```

In 2D mode, the Tile Number is split into X and Y indices, the X index is located in the LSBs (ie. MaskX=0Fh, or MaskX=1Fh, depending on DISPCNT.5).

## OBJ Attribute 0 and 2

Setting the OBJ Mode bits (Attr 0, Bit10-11) to a value of 3 has been prohibited in GBA, however, in NDS it selects the new Bitmap OBJ mode; in that mode, the Color depth bit (Attr 0, Bit13) should be set to zero; also in that mode, the color bits (Attr 2, Bit 12-15) are used as Alpha-OAM value (instead of as palette setting).

## OBJ Vertical Wrap

On the GBA, a large OBJ (with 64pix height, scaled into double-size region of 128pix height) located near the bottom of the screen has been wrapped to the top of the screen (and was NOT displayed at the bottom of the screen).

This problem has been "corrected" in the NDS (except in GBA mode), that is, on the NDS, the OBJ appears BOTH at the top and bottom of the screen. That isn't necessarily better - the advantage is that one can manually enable/disable the OBJ in the desired screen-half on IRQ level; that'd be required only if the wrapped portion is non-transparent.
