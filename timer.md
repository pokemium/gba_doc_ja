# タイマー

GBAは16bitのタイマーを4つ持っています。カウントアップにはCPUのクロック数が基準になっています。

タイマー0とタイマー1はDMAサウンドチャネルA/Bのサンプルレートのために使われます。

## タイマーとCPUの関係

GBAのCPUクロック数は16.78Mhzで、Hzの単位にすると16,780,000になります。

この数を1秒で割ると1サイクル時間がわかるという寸法です。

タイマーは1, 64, 256, 1024クロックごとにインクリメント（カウントアップ）する設定が可能です。この関係を表にすると以下のようになります。

CPU(クロック) | 1サイクル時間 | 16bitカウント時間
---- | ---- | ----
1 | 60ns | 3.9ms
64 | 3.8us | 250ms
256 | 15.3us | 1s
1024 | 61us | 4s

## タイマーレジスタ

CNT_Lがタイマーのカウンタ、CNT_Hが制御レジスタと分かれています。大きさはそれぞれ16bit単位です。

### カウンタ

```
4000100h - TM0CNT_L - タイマーカウンタ/リロード0 (R/W)
4000104h - TM1CNT_L - タイマーカウンタ/リロード1 (R/W)
4000108h - TM2CNT_L - タイマーカウンタ/リロード2 (R/W)
400010Ch - TM3CNT_L - タイマーカウンタ/リロード3 (R/W)
```

- 読み取り処理: 現在のカウンタの値を読み取る
- 書き込み処理: リロード値を初期化(ただし現在のカウンタの値には直接影響しない)

リロード値は

- タイマーオーバーフロー時
- タイマースタートビットが0から1に変更された場合

にカウンタにセットされます。

注：スタートビットを0から1に変更すると同時にリロード値を設定した場合（1回の32ビットI/O操作で）、新たに書き込まれたリロード値は新しいカウンタ値として認識されます。

### 制御レジスタ

```
4000102h - TM0CNT_H - タイマー制御0 (R/W)
4000106h - TM1CNT_H - タイマー制御1 (R/W)
400010Ah - TM2CNT_H - タイマー制御2 (R/W)
400010Eh - TM3CNT_H - タイマー制御3 (R/W)
```

 bit | 内容
---- | ----
0-1 | カウント周期 (0=1クロックごとに, 1=64, 2=256, 3=1024)
2 | カスケード機能 (0=Off, 1=On) 
3-5 | 未使用
6 | タイマー割り込み有効無効  (0=無効, 1=オーバーフロー時にIRQ)
7 | タイマー有効無効  (0=無効, 1=有効)
8-15 | 未使用

カスケード機能は下位のタイマー(タイマー1なら0、タイマー2なら1)がオーバーフローしたときに、上位のタイマーの値がインクリメントされるという機能です。

カスケード機能を使うと上位のタイマー自身は周期(bit0-1で設定)によってインクリメントされないようになっています。

タイマー0は最も下位のタイマーなので、この機能はタイマー0には使用できません。