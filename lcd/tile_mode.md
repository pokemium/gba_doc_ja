# タイルモードについて

ここではタイルモードでの画面描画に必要な、

- パレット
- タイルデータ
- マップデータ

について見ていきます。

## 🎨 パレット

### Color Palette RAM

BGパレットとOBJパレットはそれぞれ独立したメモリ領域を持っています。

- `0x0500_0000-0500_01ff` - BGパレット (512バイト, 256色)
- `0x0500_0200-0500_03ff` - OBJパレット (512バイト, 256色)

BGとOBJのパレットは、それぞれ16色×16枚のパレットとしても使えますし、256色の単一パレットとして使うこともできます。

OBJの一部が16色モードとしてパレットを使用している間、他のOBJは256色モードとしてパレットを使用することがあることに注意してください。 これはOBJに限らずBG0-BG3レイヤについても同様です。

### 透明色

すべてのBGおよびOBJパレットの色0は透明色として扱います。

よってパレットが16(256)色のパレットとして記述されていても、実際に使えるのは15(255)色だけです。

### 背景色

BGパレット0の色0は背景色として扱います。背景色は画面上で(非透明色の)BGかOBJが描画されなかったピクセルに表示されます。

### 色の定義

1色を表すのに2バイト使用します。RGBで32段階です。

前述のようにパレットのメモリ領域は512バイトなので256色 or 16色/16パレット を表現可能です。

また32768色を使用可能なBGモード(3, 5)でも色の表現は同じです。

 bit  |  内容
---- | ----
0-4   | 赤   (0-31)
5-9   | 緑 (0-31)
10-14 | 青  (0-31)
15    | 未使用

**注意**

基本的に、各色の強度は32段階ありますが、0-14にした場合全て真っ黒になってしまいます。なので実際に色を表す場合は15-31の間で表していくことになります。

色の強度が上がるにつれて画面は明るくなっていきます。

## 🦊 タイルデータ

**キャラクタデータ** や BGタイルデータ とも呼ばれます。 タイルモードにおける描画単位です。

各タイルは8×8pxです。 

色深度(色を表すのに利用するbit数)は 4bit か 8bit のどちらかを[BG制御レジスタ](./bg_control.md)で選択できます。

### 4bit (16色/16パレット)

1pxあたり4bit(0.5バイト)なので `1タイル = 64px = 64×0.5 = 32バイト`です。

最初の4バイトはタイルの最上段の行のためのものです。その次の4バイトは次の行となり、以後同様です。

各バイトは2つのドットを表し、下位4ビットが**左ドット**の色、上位4ビットが**右ドット**の色を定義します。

### 8bit (256色/1パレット)

`1タイル = 64バイト`です。

4bitと同様に最初の8バイトが最初の行に対応し、その後の行も同様です。

各バイトは、各ドットのパレットエントリを選択します。

## 🗺 マップデータ

上記のタイルデータを画面上にどのように配置していくかを表すのがマップデータ(BGマップ)です。

## テキストモードでのBGマップ

各エントリ(1タイルの配置を担当)あたり2バイトです。

Specifies the tile number and attributes. Note that BG tile numbers are always specified in steps of 1 (unlike OBJ tile numbers which are using steps of two in 256 color/1 palette mode).

 bit  |  内容 
---- | ---- 
0-9   | タイル番号 (0-1023) (a bit less in 256 color mode, because there'd be otherwise no room for the bg map)
10    | X反転 (0=しない, 1=反転)
11    | Y反転 (0=しない, 1=反転)
12-15 | パレット番号 (0-15) (256色/1パレットのときは使用しない)

A Text BG Map always consists of 32x32 entries (256x256 pixels), 400h entries = 800h bytes. 

However, depending on the BG Size, one, two, or four of these Maps may be used together, allowing to create backgrounds of 256x256, 512x256, 256x512, or 512x512 pixels, if so, the first map (SC0) is located at base+0, the next map (SC1) at base+800h, and so on.

## Rotation/Scaling BG Screen (1 byte per entry)

In this mode, only 256 tiles can be used. There are no x/y-flip attributes, the color depth is always 256 colors/1 palette.

  Bit   Expl.
  0-7   Tile Number     (0-255)

The dimensions of Rotation/Scaling BG Maps depend on the BG size. For size 0-3 that are: 16x16 tiles (128x128 pixels), 32x32 tiles (256x256 pixels), 64x64 tiles (512x512 pixels), or 128x128 tiles (1024x1024 pixels).

The size and VRAM base address of the separate BG maps for BG0-3 are set up by BG0CNT-BG3CNT registers.
