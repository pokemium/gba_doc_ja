# BG制御レジスタ

Mode0 では、タイルやマップ（タイルを並べて作った背景画面）の使い方を指定するため、BG制御レジスタを設定します。背景はBG0～BG3の4枚があり、それぞれに対応するBG制御レジスタがあります。

```
0x0400_0008 - BG0CNT - BG0制御レジスタ (R/W) (BG Modes 0,1 のみ)
0x0400_000a - BG1CNT - BG1制御レジスタ (R/W) (BG Modes 0,1 のみ)
0x0400_000c - BG2CNT - BG2制御レジスタ (R/W) (BG Modes 0,1,2 のみ)
0x0400_000e - BG3CNT - BG3制御レジスタ (R/W) (BG Modes 0,2 のみ)
```

 bit | 内容
---- | ----
0-1 | BG優先度(0~3, 0=最高)
2-3 | タイルデータ(16KB単位)を置くブロック (0-3)
4-5   | 未使用, 0
6     | モザイク (0=無効, 1=有効)
7     | カラーモード (0=16/16, 1=256/1)
8-12  | マップデータ(2KB単位)を置くブロック (0-31)
13    | BG0/BG1: 未使用, BG2/BG3: 画面オーバーフロー時の挙動 (0=Transparent, 1=Wraparound)
14-15 | 仮想画面サイズ (0-3)

優先度が同じBGが複数ある場合はBG0 -> BG3の順に優先されます(つまりBG0が最優先)。

## カラーモード(bit7)

カラーモード(bit7)は

- 0: 16色/16パレット
- 1: 256色/1パレット

## 画面サイズ(bit14-15)

画面サイズ と BGマップのサイズ(Byte単位)の関係は次のようになっています。

 bit14-15 | テキストモードのときの画面サイズ(BGマップのサイズ) | 伸縮回転モードのとき
---- | ---- | ---- 
0 | 256×256 (2KB) | 128×128   (256B)
1 | 512×256 (4KB) | 256×256   (1KB)
2 | 256×512 (4KB) | 512×512   (4KB)
3 | 512×512 (8KB) | 1024×1024 (16KB)

## タイルブロックとマップブロック

タイルモードでは、VRAMをタイルデータ(16KB単位)とマップデータ(2KB単位)で共有しますので、各データサイズを計算して、アドレスがが互いに重ならないように注意する必要があります。 たとえばタイルブロック番号2、マップ番号16のようにしてはいけません。

アドレス | タイルブロック番号 | マップブロック番号
-- | -- | --
0x0600_0000 | 0 | 0
0x0600_0800 | 0 | 1
0x0600_1000 | 0 | 2
0x0600_1800 | 0 | 3
0x0600_2000 | 0 | 4
0x0600_2800 | 0 | 5
0x0600_3000 | 0 | 6
0x0600_3800 | 0 | 7
0x0600_4000 | 1 | 8
0x0600_4800 | 1 | 9
0x0600_5000 | 1 | 10
0x0600_5800 | 1 | 11
0x0600_6000 | 1 | 12
0x0600_6800 | 1 | 13
0x0600_7000 | 1 | 14
0x0600_7800 | 1 | 15
0x0600_8000 | 2 | 16
0x0600_8800 | 2 | 17
0x0600_9000 | 2 | 18
0x0600_9800 | 2 | 19
0x0600_A000 | 2 | 20
0x0600_A800 | 2 | 21
0x0600_B000 | 2 | 22
0x0600_B800 | 2 | 23
0x0600_C000 | 3 | 24
0x0600_C800 | 3 | 25
0x0600_D000 | 3 | 26
0x0600_D800 | 3 | 27
0x0600_E000 | 3 | 28
0x0600_E800 | 3 | 29
0x0600_F000 | 3 | 30
0x0600_F800 | 3 | 31

## 例

 STATE | HEX | Binary | 内容 
---- | ---- | ---- | ---- 
Default | 0x0000 | 0b0000_0000_0000_0000 | 仮想スクリーン32x32tile, カラー16色
例1 | 0x1c80 | 0b0001_1100_1000_0000 | 仮想スクリーン32x32tile, カラー256色, タイルブロック0を使用, マップブロック28 (11100)を使用
