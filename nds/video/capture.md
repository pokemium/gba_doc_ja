# キャプチャ

## 4000064h - NDS9 - DISPCAPCNT - 32bit - Display Capture Control Register (R/W)

キャプチャは 描画エンジンA でのみサポートされています。

ビット | 内容 | 備考
-- | -- | -- 
0-4   | EVA               | 0..16 = Blending Factor for Source A
5-7   | 未使用             | -- 
8-12  | EVB               | 0..16 = Blending Factor for Source B
13-15 | 未使用             | -- 
16-17 | VRAM書き込みブロック  | 0..3 = VRAM A..D (VRAM must be allocated to LCDC)
18-19 | VRAM書き込みオフセット | 0=00000h, 0=08000h, 0=10000h, 0=18000h
20-21 | キャプチャサイズ      | 0=128x128, 1=256x64, 2=256x128, 3=256x192 dots
22-23 | 未使用             | -- 
24    | ソースA          | 0=Graphics Screen BG+3D+OBJ, 1=3D Screen
25    | ソースB          | 0=VRAM, 1=Main Memory Display FIFO
26-27 | VRAM読み取りオフセット  | 0=00000h, 0=08000h, 0=10000h, 0=18000h
28    | 未使用             | -- 
29-30 | キャプチャソース    | 0=ソースA, 1=ソースB, 2/3=ソースABをブレンド
31    | キャプチャ有効化ビット   | 0=Disable/Ready, 1=Enable/Busy

Notes:

VRAM読み取りブロック (VRAM A..D)は DISPCNTのbit18-19で指定します。

VRAM読み取りブロックはLCDC(MST=0)に割り当てられている必要があります。

VRAM読み取りオフセットはVRAM Display Mode(DISPCNT.16-17)のとき、無視され、0として扱われます。

VRAM読み取りオフセット・VRAM書き込みオフセットは`1FFFFh`を超えた場合、`00000h`に戻ってきます。(最大 128K)

キャプチャサイズが`256x192`より小さい場合、画面の左上の部分がキャプチャされます。

ブレンドのパラメータである、EVAとEVBはbit29-30で`2 or 3`が選択されている時のみ利用されます。

キャプチャ有効化ビットをセットすると、次のスキャンライン0からキャプチャが始まります。そして、キャプチャサイズに関係なく、スキャンライン192でキャプチャ有効化ビットはクリアされます。

キャプチャデータは15bitの色深度を持っています。3Dイメージの場合は18bitです。

Capture A:  `Dest_Intensity = SrcA_Intensitity ; Dest_Alpha=SrcA_Alpha`.

Capture B:  `Dest_Intensity = SrcB_Intensitity ; Dest_Alpha=SrcB_Alpha`.

Capture A+B (blending):

```
 Dest_Intensity = ((SrcA_Intensitity * SrcA_Alpha * EVA) + (SrcB_Intensitity * SrcB_Alpha * EVB)) / 16
 Dest_Alpha = (SrcA_Alpha AND (EVA>0)) OR (SrcB_Alpha AND EVB>0)
```

キャプチャはいくつかの面白い使い方を提供します。

例えば、3Dエンジンの出力をソースA経由で（LCDC割り当てのVRAMへ）取り込み、次のフレームで描画エンジンAまたはBが、取り込んだ3D画像をBG2、BG3、またはOBJとしてVRAM画像に（BG/OBJ割り当てのVRAMから）表示することができます。この方法ではLCDC割り当てとBG/OBJ割り当てを切り替える必要があります。

また、エンジンAの出力をキャプチャし、キャプチャした画像をVRAM表示モードで次のフレームに表示すると同時に、新しいエンジンAの出力をキャプチャし、古いキャプチャした画像と混ぜ合わせることができます。このモードでは、移動したオブジェクトは画面上に痕跡を残します。この方法は、LCDCに割り当てられた1つのVRAMブロックで動作します。
