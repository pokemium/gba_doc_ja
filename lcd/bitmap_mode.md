# ビットマップモード

BGモード3-5では背景の描画はタイル形式ではなく、ビットマップ形式でおこないます。

この形式はメモリ領域に値を書き込むと、それがそのまま画面に反映されます。

ビットマップ形式ではVRAMのうち80KBを背景描画に使うので、OBJタイルデータの格納に使えるのは残りの16KBだけになります。

## BGモード3 - 240×160px, 32768色

- `0x0600_0000-0x0601_3fff`:  80KBのフレームバッファ0 (使えるのは75KB)
- `0x0601_4000-0x0601_7fff`:  16KBでOBJタイルデータを格納

各ピクセルを表すのに2バイト使用し、パレットデータを介さず直接32768色の中から色を指定します。透明色はありません。

 bit  |  内容
---- | ----
0-4   | 赤   (0-31)
5-9   | 緑 (0-31)
10-14 | 青  (0-31)
15    | 使用しない

最初の480バイトは最初の行(1行=240pxなので480バイト)で、次の480バイトは次の行です。以後も同様です。

画面全体を描画するのに必要なメモリ領域は76800バイト(=75KB: `0x0600_0000`-`0x0601_2bff`)で、あらかじめ用意された80KBのフレームバッファのほとんどを使用します。

## BGモード4 - 240×160px, (32768色のうちの)256色

- `0x0600_0000-0x0600_9fff`: 37.5KBのフレームバッファ0
- `0x0600_a000-0x0601_3fff`: 37.5KBのフレームバッファ1
- `0x0601_4000-0x0601_7fff`: 16KBでOBJタイルデータを格納

1バイトが1ピクセルと対応していて、256個のパレットのうちの1つを選択します。カラー0（背景）は透明色として扱われ、OBJをビットマップの後ろに表示することも可能です。

最初の240バイトが1行目、次の240バイトが2行目となっており以後も同様です。

このモードでは画面全体をカバーするのに必要なフレームバッファが37.5KBだけで済み、VRAM全体で2つのフレームバッファを使用することができます。

## BGモード5 - 160×128px, 32768色

- `0x0600_0000-0x0600_9fff`: 40KBのフレームバッファ0
- `0x0600_a000-0x0601_3fff`: 40KBのフレームバッファ1
- `0x0601_4000-0x0601_7fff`: 16KBでOBJタイルデータを格納

基本的にBGモード3と同じですが、フレームバッファと対応する画面のサイズがBGモード3では`240px×160px`でしたがBGモード5では`160px×128px`と小さくなっています。

このとき画面全体をカバーするのに必要なフレームバッファは40KBとなり、VRAMは2つのフレームバッファを持つことができます。

BGモード4,5では[LCD制御レジスタ](./lcd_control.md)のbit4でフレーム0とフレーム1のうち、どちらを画面に反映させるか選択します。表示されなかった片方のフレームは再描画される可能性があります。
