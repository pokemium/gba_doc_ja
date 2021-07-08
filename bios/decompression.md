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


Data Offsetは解凍対象のデータが0でないときに、解凍対象のデータに加えられる値です。ただし、Zero Data Flagがセットされている場合、解凍対象のデータが0であってもData Offsetが加えられます。

解凍したデータは32bit単位でWRAMかVRAMのターゲットアドレスで指定したメモリに書き込まれます。

解凍データのサイズは必ず4バイトの倍数である必要があります。ソースデータの単位はターゲットデータの単位を超えてはいけません。

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
