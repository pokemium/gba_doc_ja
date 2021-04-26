# メモリマップ

## メモリマップ

**内部メモリ**

空いている領域は未使用領域です。

アドレス | サイズ | バス幅 | 名称 | 用途
---- | ---- | ---- | ---- | ----
00000000-00003FFF | 16KB | 32bit | BIOS | 起動プログラムなどを実装したROM
02000000-0203FFFF | 256KB | 16bit | WRAM(オンボード) | プログラムを入れる外部メモリ EWRAMとも
03000000-03007FFF | 32KB | 32bit | WRAM(チップ) | ARM内蔵の高速メモリ IWRAMとも
04000000-040003FE | 1KB | 32bit | IOレジスタ | IO機能を制御するレジスタ

**内部メモリ(画面描画用)**

空いている領域は未使用領域です。

アドレス | サイズ | バス幅 | 名称 | 用途
---- | ---- | ---- | ---- | ----
05000000-050003FF | 1KB | 16bit | パレット | 256色分色を記憶可能
06000000-06017FFF | 96KB | 16bit | VRAM | LCD に表示するデータを入れる
07000000-070003FF | 1KB | 32bit | OAM | オブジェクトと背景を重ねる機能

**外部メモリ (Game Pak)**

Game Pak = カートリッジのことです。

アドレス | サイズ | バス幅 | 名称 | 備考
---- | ---- | ---- | ---- | ----
08000000-09FFFFFF | 32MB | 16bit | Game Pak ROM/FlashROM | WaitState 0
0A000000-0BFFFFFF | 32MB | 16bit | Game Pak ROM/FlashROM | WaitState 1
0C000000-0DFFFFFF | 32MB | 16bit | Game Pak ROM/FlashROM | WaitState 2
0E000000-0E00FFFF | 64KB | 8bit | SRAM | --
0E010000-0FFFFFFF | -- | -- | 未使用 | -- 

`10000000-FFFFFFFF`は使用できません。

## WRAMについて

デフォルトでは、WRAMの`0x0300_7F00-0x0300_7FFF`の256バイトは、割り込みベクタ、割り込みスタック、BIOSコールスタック用に予約されています。

残りのWRAMは、`0x0300_7F00`から配置されているユーザースタックを含めてどのような用途にも自由に使用できます。

## アクセス

バス幅とCPUが一度に読み書き可能なbit数、アクセスに要するサイクルの一覧です。

メモリ領域 | バス幅 | Read幅 | Write幅 | サイクル(8/16/32bit) | 備考(後述)
---- | ---- | ---- | ---- | ---- | ---- 
BIOS      | 32  |  8/16/32  | -       |  1/1/1 | --
WRAM(オンボード) | 16  |  8/16/32  | 8/16/32 |  3/3/6 | ②
WRAM(チップ)  | 32  |  8/16/32  | 8/16/32 |  1/1/1 | --
IOレジスタ           | 32  |  8/16/32  | 8/16/32 |  1/1/1 | --
パレット   | 16  |  8/16/32  | 16/32   |  1/1/2 | ①
VRAM          | 16  |  8/16/32  | 16/32   |  1/1/2 | ①
OAM           | 32  |  8/16/32  | 16/32   |  1/1/1 | ①
GamePak ROM   | 16  |  8/16/32  | -       |  5/5/8 | ②,③
GamePak Flash | 16  |  8/16/32  | 16/32   |  5/5/8 | ②,③
GamePak SRAM  | 8   |  8        | 8       |  5     | ②

これ以外にも16bitごとまたは32bitごとのDMA転送がSRAMをのぞいた全てのメモリに可能です。

備考について

- ①: GBAが同時にビデオメモリにアクセスする場合はプラス1サイクル。
- ②: サイクルはwaitstateの設定によって変わってきます。
- ③: シーケンシャルアクセスとノンシーケンシャルアクセスのタイミングを分けます。

## VRAM、OAM、パレット

VRAM、パレットは、**HBlank中またはVBlank中のみアクセス可能**です。 DISPCNTレジスタの強制白塗りビットで表示が無効化されているときはそれ以外のタイミングでもアクセスできます。

OAMはさらにアクセス条件が厳しいです。 [DISPCNT](lcd/lcd_control.md#0x0400_0000---dispcnt---lcd%E5%88%B6%E5%BE%A1%E3%83%AC%E3%82%B8%E3%82%B9%E3%82%BF-rw)のbit5が1の場合にのみ、HBlank中のアクセスが許可されます。

## GamePak(カートリッジ)

CPUとDMA3のみがGamePak ROMにアクセスできます。

SRAMにはCPUしかアクセスできず一度に8bitごとしかアクセスできません。 SRAMは電池形式とフラッシュメモリ形式があります。

GamePak ROMのバスは16bitに制限されているため、GamePak ROMの内部からARM命令（32bit）を実行すると、あまり良いパフォーマンスが得られません。よってTHUMB命令(16bit)を使うことをお勧めします。

ただし、どうしてもARM命令を使いたい時はGamePak ROMから内部のWRAMにコードをコピーすることで、パフォーマンスを落とさず実行が可能です。

## データフォーマット

GBAでは常にデータはリトルエンディアン形式です。

## Reading from Unused Memory (00004000-01FFFFFF,10000000-FFFFFFFF)

Accessing unused memory at 00004000h-01FFFFFFh, and 10000000h-FFFFFFFFh (and 02000000h-03FFFFFFh when RAM is disabled via Port 4000800h) returns the recently pre-fetched opcode. 

For ARM code this is simply:

```
  WORD = [$+8]
```

For THUMB code the result consists of two 16bit fragments and depends on the address area and alignment where the opcode was stored.

For THUMB code in Main RAM, Palette Memory, VRAM, and Cartridge ROM this is:

```
  LSW = [$+4], MSW = [$+4]
```

For THUMB code in BIOS or OAM (and in 32K-WRAM on Original-NDS (in GBA mode)):

```
  LSW = [$+4], MSW = [$+6]   ; for opcodes at 4-byte aligned locations
  LSW = [$+2], MSW = [$+4]   ; for opcodes at non-4-byte aligned locations
```

For THUMB code in 32K-WRAM on GBA, GBA SP, GBA Micro, NDS-Lite (but not NDS):

```
  LSW = [$+4], MSW = OldHI   ; for opcodes at 4-byte aligned locations
  LSW = OldLO, MSW = [$+4]   ; for opcodes at non-4-byte aligned locations
```

Whereas OldLO/OldHI are usually:

```
  OldLO=[$+2], OldHI=[$+2]
```

Unless the previous opcode's prefetch was overwritten; that can happen if the previous opcode was itself an LDR opcode, ie. if it was itself reading data:

```
  OldLO=LSW(data), OldHI=MSW(data)
```

Theoretically, this might also change if a DMA transfer occurs.

Note: Additionally, as usually, the 32bit data value will be rotated if the data address wasn't 4-byte aligned, and the upper bits of the 32bit value will be masked in case of LDRB/LDRH reads.

Note: The opcode prefetch is caused by the prefetch pipeline in the CPU itself, not by the external gamepak prefetch, ie. it works for code in ROM and RAM as well.
