# メモリアクセス時間

## システムクロック

```
  Bus clock  = 33MHz (33.513982 MHz)
  NDS7 clock = 33MHz (busと同じクロック)
  NDS9 clock = 66MHz (busの2倍のクロック)
```

このドキュメントに記載されているほとんどのタイミングは、33MHzのクロックで指定されています。（66MHzのクロックではありません）

また、NDS9のタイミングは「ハーフ」サイクルでカウントされます。

## メモリアクセスにかかる時間

以下の表は、ARM7/ARM9 CPUにおけるコードフェッチ・データフェッチのアクセス時間の違いを、シーケンシャル/ノンシーケンシャルの32bit/16bitアクセスで測定したものです。

すべてのタイミングは33MHz単位でカウントされます。つまり 1サイクル = 1/33M秒 です。

注：8bitデータのアクセスにかかる時間は、16bitデータと同じ時間です。

### コードフェッチ

**NDS7**

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
Main RAM (read) (cache off) | 9 | 2 | 8 | 1 | 16 
WRAM,BIOS,I/O,OAM | 1 | 1 | 1 | 1 | 32 
VRAM,Palette RAM | 2 | 2 | 1 | 1 | 16 
GBA ROM (example 10,6 access) | 16 | 12 | 10 | 6 | 16 

**NDS9**

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
Main RAM (read) (cache off) | 9 | 9 | 4.5 | 4.5 | 16 
WRAM,BIOS,I/O,OAM | 4 | 4 | 2 | 2 | 32 
VRAM,Palette RAM | 5 | 5 | 2.5 | 2.5 | 16 
GBA ROM (example 10,6 access) | 19 | 19 | 9.5 | 9.5 | 16 
TCM, Cache_Hit | 0.5 | 0.5 | 0.5 | 0.5 | 32 

### データフェッチ

**NDS7**

NDS7のコードフェッチと全く同じです。ただし、メインRAMの非連続アクセスは1サイクル遅く、GBAスロットの非連続アクセスは1サイクル速いという不思議なことになっています。

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
Main RAM (read) (cache off) | 10 | 2 | 9 | 1 | 16 
WRAM,BIOS,I/O,OAM | 1 | 1 | 1 | 1 | 32 
VRAM,Palette RAM | 1 | 2 | 1 | 1 | 16 
GBA ROM (example 10,6 access) | 15 | 12 | 9 | 6 | 16 
GBA ROM (example 10 access) | 9 | 10 | 9 | 10 | 8 

**NDS9**

Region | N32 | S32 | N16 | S16 | Bus幅 
-- | -- | -- | -- | -- | -- 
Main RAM (read) (cache off) | 10 | 2 | 9 | 1 | 16 
WRAM,BIOS,I/O,OAM | 4 | 1 | 4 | 1 | 32 
VRAM,Palette RAM | 5 | 2 | 4 | 1 | 16 
GBA ROM (example 10,6 access) | 19 | 12 | 13 | 6 | 16 
GBA ROM (example 10 access) | 13 | 10 | 13 | 10 | 8 
TCM, Cache_Hit | 0.5 | 0.5 | 0.5 | -- | 32 
Cache_Miss (BIOS) | 11 | 11 | 11 | -- | 32 
Cache_Miss (Main RAM) | 23 | 23 | 23 | -- | 16 

これは最も厄介なタイミングです。

全ての非連続アクセス（キャッシュ、Tcm、Main RAMを除く）に3サイクルの悪名高いPENALTYが加算されます。

また、すべてのオペコードフェッチは強制的に32ビットの非連続アクセスにされます。(NDS9は単に高速なシーケンシャルオペコードフェッチをサポートしていません)

これはTHUMBコードにも当てはまります。つまり2つの16bitオペコードを1回のノンシーケンシャルな32bitアクセスでフェッチします。そのため、16bitオペコードあたりの時間は32bitフェッチの半分になります。
