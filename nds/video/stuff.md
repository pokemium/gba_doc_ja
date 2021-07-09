# 概要

## DS Display Dimensions / Timings

Dot clock = 5.585664 MHz (=33.513982 MHz / 6)

H-Timing: 256 dots visible, 99 dots blanking, 355 dots total (15.7343KHz)
V-Timing: 192 lines visible, 71 lines blanking, 263 lines total (59.8261 Hz)

The V-Blank cycle for the 3D Engine consists of the 23 lines, 191..213.

Screen size 62.5mm x 47.0mm (each) (256x192 pixels)

Vertical space between screens 22mm (equivalent to 90 pixels)

## 400006Ch - NDS9 - MASTER_BRIGHT - 16bit - Master Brightness Up/Down

画面の明るさを決めるレジスタ

ビット | 内容
-- | -- 
0-4 | Factor(後述, 0-16で16より大きい場合は16として扱う)
5-13 | 不使用
14-15 | モード(0=無効, 1=Up, 2=Down, 3=Reserved)
16-31 | 不使用

明るさは次のように決定します。

モード(bit14-15)が

- 1(Up)のとき: `New = Old + ((63-Old) * Factor/16)`
- 2(Down)のとき: `New = Old - (Old * Factor/16)`

## DISPSTAT/VCOUNT

TODO
