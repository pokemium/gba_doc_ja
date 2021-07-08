# 解凍

## BitUnPack: swi 0x10(GBA/NDS7/NDS9/DSi7/DSi9)

ビットマップやタイルデータの色深度を上げるために使用します。例えば、1bitのモノクロフォントを4bitや8bitのGBAタイルに変換する場合などに使用します。

Unpack Infoをr2で指定することで、同じソースデータを異なるフォーマットに変換することができます。

引数:

- r0: ソースアドレス(アラインメントはしなくてもよい)
- r1: ターゲットアドレス(必ず4byte単位でアラインメント)
- r2: Unpack Infoのポインタ
  - 16bit: ソースデータの長さ(バイト単位, 0x-0xFFFF)
  - 8bit: ソースデータの単位(bit単位, 1, 2, 4, 8)
  - 8bit: ターゲットデータの単位(bit単位, 1, 2, 4, 8, 16, 32)
  - 32bit: Data Offset (Bit 0-30), Zero Data Flag (Bit 31)

The Data Offset is always added to all non-zero source units. 

Zero Data Flagがセットされている場合、
If the Zero Data Flag was set, it is also added to zero units.

Data is written in 32bit units, Destination can be Wram or Vram. 

The size of unpacked data must be a multiple of 4 bytes. The width of source units (plus the offset) should not exceed the destination width.

返り値: なし。ターゲットアドレスにデータが書き込まれます。

## Diff8bitUnFilterWrite8bit (Wram): swi 0x16 (GBA/NDS9/DSi9)
## Diff8bitUnFilterWrite16bit (Vram): swi 0x17 (GBA)
## Diff16bitUnFilter: swi 0x18 (GBA/NDS9/DSi9)

## HuffUnCompReadNormal: swi 0x13(GBA)
## HuffUnCompReadByCallback: swi 0x13(NDS)

## LZ77UnCompReadNormalWrite8bit (Wram): swi 0x11 (GBA/NDS7/NDS9/DSi7/DSi9)
## LZ77UnCompReadNormalWrite16bit (Vram): swi 0x12 (GBA)
## LZ77UnCompReadByCallbackWrite8bit: swi 0x01 (DSi7/DSi9)
## LZ77UnCompReadByCallbackWrite16bit: swi 0x12 (NDS), swi 0x02 or swi 0x19 (DSi)

## RLUnCompReadNormalWrite8bit (Wram): swi 0x14 (GBA/NDS)
## RLUnCompReadNormalWrite16bit (Vram): swi 0x15 (GBA)
## RLUnCompReadByCallbackWrite16bit: swi 0x15 (NDS)
