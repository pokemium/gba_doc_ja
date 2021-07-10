# 仕様

## ハードウェア

### プロセッサ

- 1x ARM946E-S 32bit RISC CPU, 66MHz (NDS9 video) (not used in GBA mode)
- 1x ARM7TDMI  32bit RISC CPU, 33MHz (NDS7 sound) (16MHz in GBA mode)

ARM946E-S: v5TE

### 内部メモリ

- 4096KB Main RAM (8192KB in debug version)
- 96KB   WRAM (64K mapped to NDS7, plus 32K mappable to NDS7 or NDS9)
- 60KB   TCM/Cache (TCM: 16K Data, 32K Code) (Cache: 4K Data, 8K Code)
- 656KB  VRAM (allocateable as BG/OBJ/2D/3D/Palette/Texture/WRAM memory)
- 4KB    OAM/PAL (2K OBJ Attribute Memory, 2K Standard Palette RAM)
- 248KB  Internal 3D Memory (104K Polygon RAM, 144K Vertex RAM)
- ?KB    Matrix Stack, 48 scanline cache
- 8KB    Wifi RAM
- 256KB  Firmware FLASH (512KB in iQue variant, with chinese charset)
- 36KB   BIOS ROM (4K NDS9, 16K NDS7, 16K GBA)

### グラフィック

- 2x LCD screens (each 256x192 pixel, 3 inch, 18bit color depth, backlight)
- 2x 2D video engines (extended variants of the GBA's video controller)
- 1x 3D video engine (can be assigned to upper or lower screen)
- 1x video capture (for effects, or for forwarding 3D to the 2nd 2D engine)

### サウンド

- 16 sound channels (16x PCM8/PCM16/IMA-ADPCM, 6x PSG-Wave, 2x PSG-Noise)
- 2 sound capture units (for echo effects, etc.)
- Output: Two built-in stereo speakers, and headphones socket
- Input:  One built-in microphone, and microphone socket

### 操作

- キー入力(4つの方向キー + 8つのボタン)
- タッチスクリーン

### 通信ポート

Wifi IEEE802.11b

### その他

- Built-in Real Time Clock
- Power Managment Device
- Hardware divide and square root functions
- CP15 System Control Coprocessor (cache, tcm, pu, bist, etc.)

### 外部メモリ

- NDS Slot (for NDS games) (encrypted 8bit data bus, and serial 1bit bus)
- GBA Slot (for NDS expansions, or for GBA games) (but not for DMG/CGB games)

### カートリッジ

- ROM: 16MB, 32MB, 64MB
- EEPROM/FLASH/FRAM: 0.5KB, 8KB, 64KB, 256KB, 512KB

### Can be booted from

- NDS Cartridge (NDS mode)
- Firmware FLASH (NDS mode) (eg. by patching firmware via ds-xboo cable)
- Wifi (NDS mode)
- GBA Cartridge (GBA mode) (without DMG/CGB support) (without SIO support)

### 電源

- 組み込みのリチウムイオン充電池 3.7V 1000mAh (DS-Lite)
- 外部電源: 5.2V DC

## DSLite

DSLiteは、初代NDSよりもわずかに小さく、見た目も洗練されています。

液晶ディスプレイはよりカラフルになりました。（そのため、古いNDSやGBAのゲームとの互換性はありません）

また液晶ディスプレイはより広い視野角をサポートするようになりました。

バックライトの明るさが選択可能、外部電源フラグの新設、オーディオアンプのミュートフラグの消失と、電源管理装置が若干異なります。

WifiコントローラもDSとは若干異なっています。具体的には、チップIDが異なる、無効なWifiポートや未使用のWifiメモリ領域にアクセスした際の汚れの影響が異なる、GAPDISPレジスタの動作が異なる、RF/BBチップが1つのチップに置き換えられています。

タッチスクリーンコントローラも新しい未使用の入力と、わずかに異なるパワーダウンビットを持つようになり、Wifiコントローラ同様に、DSとは異なったものになりました。

## 動作

- NDS9: ARM9プロセッサ + NDSモードのメモリとIOポート
- NDS7: ARM7プロセッサ + NDSモードのメモリとIOポート
- GBA: ARM7プロセッサ + GBAモードのメモリとIOポート

## 2つのプロセッサ

基本的に、ゲームのコードのほとんどはARM9プロセッサで実行されます。

実際、任天堂は、あらかじめ定義されたAPI関数を除いて、ARM7プロセッサを使用することを開発者に許可していないと言われています。最も非効率的なAPIコードを使用しても、ARM7の33MHzのマシンパワーのほとんどは使用されることはありません。ただし、ARM9だけを使うと、GBAゲームとの互換性に問題が生じる可能性があるので、33MHzのARM7を搭載していたようです。

一方の、ARM9の66MHzは、当然フルに利用することを想定されています。

任天堂は、33MHzのプロセッサでは3Dを使うゲームには遅すぎると考え、オリジナルのGBAハードウェアに追加のCPUをバッジしたようです。

しかし、**66MHzで動作するのはキャッシュと[TCM](./memctl/cache_tcm.md)においてのみ**で、それ以外のメモリやI/Oアクセスは33MHzのバスクロックに引っ張られて33MHzで動作します。

それでもかなり速いのですが、NDS9側ではノンシーケンシャルアクセスに本来1回でいいはずのWaitstateを3回も加えてしまうというハードウェアの不具合があるせいで、バスクロックが実質的に8MHz程度に落ちてしまい、33MHzのNDS7プロセッサよりもはるかに遅く、オリジナルの16MHzのGBAプロセッサよりもさらに遅いという不思議な状況に陥っています。

結局、バグのある66MHzと使われていない33MHzという2つのプロセッサのパフォーマンスはGBAの16MHzプロセッサとほとんど同じという悲しいことになりました。
