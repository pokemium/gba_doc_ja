# バックアップメディア

セーブ機能のことです。

## GBA Cart Backup IDs

GBAはROMヘッダにバックアップタイプのエントリを入れていません。しかしバックアップタイプはROMイメージ内のID文字列から検出できます。

任天堂のツールでは、これらの文字列を（ライブラリ・ヘッダの一部として）自動的に挿入しています。他のツールを使用する場合は、手動でID文字列を挿入することができます。

**ID文字列**

ID文字列はワードアラインメントされたメモリアドレスに配置する必要があり、文字列の長さは4バイトの倍数でなければなりません。（余った分はゼロでパディングされます）

```
  EEPROM_Vnnn    EEPROM 512B or 8KB (4Kbit or 64Kbit)
  SRAM_Vnnn      SRAM 32KB (256Kbit)
  FLASH_Vnnn     FLASH 64KB (512Kbit) (ID used in older files)
  FLASH512_Vnnn  FLASH 64KB (512Kbit) (ID used in newer files)
  FLASH1M_Vnnn   FLASH 128KB (1Mbit)
```

任天堂のツールでは、"nnn "は3桁のライブラリバージョン番号です。他のツールを使用する場合は、数字を入れずに"nnn"にしておくとよいでしょう。

## GBA Cart Backup SRAM/FRAM

```
SRAM - 32KB (256Kbit) セーブ機能の寿命: バッテリー依存
FRAM - 32KB (256Kbit) セーブ機能の寿命: 10,000,000,000回の読み書き

Hyundai GM76V256CLLFW10 SRAM (Static RAM) (eg. F-Zero)
Fujitsu MB85R256 FRAM (Ferroelectric RAM) (eg. Warioware Twisted)
```

**アドレスとWaitstate**

SRAM/FRAM は `0xE000000-0xE007FFF`にマッピングされます。

またアクセスには 8 waitstates を要します。(WAITCNTのbit0-1が3の場合)

**バス幅**

SRAM/FRAM のバス幅は8bitで固定で、`LDRB`,`LDRSB`,`STRB`によってのみアクセス可能です。

**読み書き**

SRAM/FRAMからの読み出しは、WRAMで実行されたコードのみが行うことができます。（ROMで実行されたコードはできません）

書き込みについては、そのような制限はありません。

**データ損失の防止**

GBAのSRAM/FRAMカートリッジには、（古い8bitゲームボーイのカートとは異なり）ライトプロテクト機能が搭載されていません。

これが問題で、GBAの電源を入れたままカートリッジを抜き差しすると、データが失われることがあるようです。

私の理解では、これはハードウェアの問題というよりも、ソフトウェアの問題です。

つまり、理論的には何度でもカートリッジを抜き差しすることができますが、プログラムがクラッシュしないように（やみくもにメモリに書き込まないように）注意する必要があります。

**推奨される挙動**

（カートリッジを取り外すときにトリガされる可能性が高い）Gamepak Interruptを有効にして、割り込みハンドラがGamepak IRQを感知すると、GBAを無限ループでハングアップさせます。理由は明白で、割り込みハンドラは（取り外した）ROMカートリッジではなく、WRAMに配置する必要があります。ハンドラはGamepakのIRQを最優先で処理します。割り込みが禁止されている期間はできるだけ短くし、必要に応じてネストした割り込みを許可してください。

**上記の挙動を使うべきタイミング**

WRAMのコードとデータに完全に依存していて、ROMを取り外してもクラッシュしないプログラムは、上記のメカニズムを使用しなくても動作を続けることができます。カートリッジが挿入されていない状態で動作するプログラム（シングルゲームパック/マルチブートスレーブなど）や、ゲームパックのIRQ/DMAを他の目的で使用しているプログラムには、この回避策を使用しないでください。

それ以外のプログラムではこの回避策を使用してください。最終的には、SRAM/FRAMを使用しないプログラムにもこの回避策を含めることをお勧めします（例えば、SRAM/FRAMを使用しないカートリッジを取り外すとGBAがロックされ、SRAM/FRAMカートリッジを挿入するとバックアップデータが破壊される可能性があります）。

**SRAM と FRAM の比較**

FRAM(Ferroelectric RAM)は新しい技術で、最新のGBAカートに採用されています。SRAM(Static RAM)とは異なり、データを保持するための電池を必要としません。

しかもソフトウェア側では、SRAMと同じようにアクセスできます。つまり、EEPROM/FLASHとは異なり、書き込み/消去のコマンド/遅延 を必要としません。

**注意**

SRAM/FRAMカートリッジでは、/REQピン（ゲームパックバスの31番ピン）を他のピンよりも少し短くする必要があります。これにより、カートリッジを取り外す際に、他のピンが切断される前にゲームパックのIRQ信号がトリガーされることになります。

## GBA Cart Backup EEPROM

```
9853 - EEPROM 512B (0200h) (4Kbit) (eg. used by Super Mario Advance)
9854 - EEPROM 8KB (2000h) (64Kbit) (eg. used by Boktai)
セーブ機能の寿命: アドレスあたり100,000回の読み書き
```

**Addressing and Waitstates**

eepromは、データバスのBit0と、カートリッジROMアドレスバスの上位1ビット(32MBの大容量ROMの場合は上位17ビット)に接続されており、チップとの通信はシリアル通信で行われます。

eepromのアクセスには8waitstatesが必要です。(set WAITCNT=X3XXh; 8,8 clks in WS2 area)

またeepromは`0xDFFFF00..0xDFFFFFF`にマッピングされます。

それぞれ、eepromの場合、ROMは`0x8000000-0x9FFFeFF`(最大で0x1FFFF00バイト = `32MB - 256B`)に制限されます。

16MB以下のROMを搭載したカートでは、`0xD000000-0xDFFFFFF`の任意の位置でeepromに交互にアクセスすることができます。

**Data and Address Width**

EEPROMのデータは、64ビット（8バイト）単位で読み書きできます。

書き込みを行うと、それまでの64ビットのデータは自動的に消去されます。すなわち，512バイトのEEPROMでは、`0x0-0x3F`のアドレス範囲、6ビットのバス幅となります。8KBのEEPROMでは、`0x0-0x3FF`のアドレス範囲，14ビットのバス幅となります。（アドレスの下位10ビットのみ使用し、上位4ビットは0とします）

**Set Address (For Reading)**

次のようなビットストリームをメモリ上に用意します。

```
  2 bits "11" (Read Request)
  n bits eeprom address (MSB first, 6 or 14 bits, depending on EEPROM)
  1 bit "0"
```

その後、DMAを使ってeepromにストリームを転送します。

**Read Data**

DMAを使ってEEPROMから68ビットのストリームを読み出し、受信したデータを以下のようにデコードします。

```
  4 bits - ignore these
 64 bits - data (conventionally MSB first)
```

**Write Data to Address**

次のようなビットストリームをメモリ上に用意し、DMAを使ってeepromに転送します。

古いデータが消去され、新しいデータがプログラムされるまで、約108368クロックサイクル（約6.5ms）かかります。

```
  2 bits "10" (Write Request)
  n bits eeprom address (MSB first, 6 or 14 bits, depending on EEPROM)
 64 bits data (conventionally MSB first)
  1 bit "0"
```

DMA終了後、通常の`LDRH [DFFFF00h]`により、戻ってきたデータのビット0が "1"(Ready)になるまで、チップからの読み出しを続けます。

誤動作時にプログラムがロックしないように、10ms以上経ってもチップが応答しない場合はタイムアウトを発生させてください。

**Using DMA**

ビットストリームのEEPROMへ(or から)の転送は、DMA3を介して行う必要があります。 転送中に/CS=LOWとA23=HIGHを維持しないため、LDRH/STRHによる手動転送は動作しません。

DMAを使用するためには、メモリ上のバッファを使用する必要があります。そのバッファは、通常、スタック上に一時的に割り当てられ、ハーフワードあたり1bitを使用します。つまりbit0のみが使用され、bit1-15は無視されます。

バッファはDMA3を使用してEEPROMへ(or から)全体として転送する必要があります（DMA0-2は外部メモリにアクセスできません）。転送モードは16ビットで、ソースとデスティネーションの両方のアドレスをインクリメントします。（例：DMA3CNT=`0x80000000+length`）

優先度の高いDMAチャンネルは、転送中は無効にしてください（例：H/V-BlankやSound FIFO DMA）。また、DMAレジスタに干渉する可能性のある割り込みは、当然ながら無効にしてください。

**Pin-Outs**

EEPROMチップには8つのピンしかなく、これらのピン1～8は、GamePakバスのROMCS、RD、WR、AD0、GND、A23、VDDに接続されています。

32MBのROMを搭載したカートでは、A7～A22とA23を論理的にANDする必要があります。

**注意**

自動検出の仕組みがないため、ハードコードされたバス幅を使用しなければならないようです。

## GBA Cart Backup Flash ROM

```
64 KB - 512Kbits Flash ROM - セーブ機能の寿命: セクタあたり10,000回の書き込み
128 KB - 1Mbit Flash ROM - セーブ機能の寿命: セクタあたりの書き込み回数上限があるはずだが、具体的な回数は不明
```

**チップの検出**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=90h  (enter ID mode)
  dev=[E000001h], man=[E000000h]                  (get device & manufacturer)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=F0h  (terminate ID mode)
```

FLASHチップの種類（および存在）を検出するために使用します。下記のデバイスの種類を参照してください。

**(複数バイト単位の)データの読み出し**

```
  dat=[E00xxxxh]                                  (read byte from address xxxx)
```

**データの消去**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=80h  (erase command)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=10h  (erase entire chip)
  wait until [E000000h]=FFh (or timeout)
```

チップ内の全メモリを消去し、消去されたメモリは0xFFで埋められます。

**4KBセクタ単位のデータ消去 (Atmel以外)**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=80h  (erase command)
  [E005555h]=AAh, [E002AAAh]=55h, [E00n000h]=30h  (erase sector n)
  wait until [E00n000h]=FFh (or timeout)
```

`E00n000h...E00nFFFh`のメモリを消去し、消去されたメモリは0xFFで埋められます。

**128Bセクタ単位の消去と書き込み (Atmelのみ)**

```
  old=IME, IME=0                                  (disable interrupts)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=A0h  (erase/write sector command)
  [E00xxxxh+00h..7Fh]=dat[00h..7Fh]               (write 128 bytes)
  IME=old                                         (restore old IME state)
  wait until [E00xxxxh+7Fh]=dat[7Fh] (or timeout)
```

コマンドフェーズ・ライトフェーズでは、割り込み（およびDMA）を無効にする必要があります。ターゲットアドレスは、0x80の倍数でなければなりません。

**1バイトの書き込み (Atmel以外)**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=A0h  (write byte command)
  [E00xxxxh]=dat                                  (write byte to address xxxx)
  wait until [E00xxxxh]=dat (or timeout)
```

対象となるメモリアドレスは、事前に消去されている必要があります。

**Terminate Command after Timeout (only Macronix devices, ID=1CC2h)**

```
  [E005555h]=F0h                            (force end of write/erase command)
```

Macronixデバイスのみ、"wait until"期間中にタイムアウトが発生した場合に使用します。

**バンクの切り替え (64KBより大きいチップのみ)**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=B0h  (select bank command)
  [E000000h]=bnk                                  (write bank number 0..1)
```

読み出し/書き込み/消去 操作の対象の64Kバンク番号を指定します。

カートリッジのflash/sramのアドレスバスが16bit幅に制限されているため必要です。

**デバイスの種類**

任天堂は市販のゲームカートリッジに様々な種類のFLASHチップを搭載しています。

そのため、開発者はすべてのチップタイプを検出し、サポートする必要があります。

Atmel製チップの場合、ソフトウェアで4Kセクタをシミュレートすることが推奨されますが、任天堂は後期のゲームではAtmelチップを使用していないと言われています。

また、タイミングの違いが互換性やパフォーマンスを妨げないように注意してください。

```
  ID     Name       Size  Sectors  AverageTimings  Timeouts/ms   Waits(w,r)
  D4BFh  SST        64K   16x4K    20us?,?,?       10,  40, 200  3,2
  1CC2h  Macronix   64K   16x4K    ?,?,?           10,2000,2000  8,3
  1B32h  Panasonic  64K   16x4K    ?,?,?           10, 500, 500  4,2
  3D1Fh  Atmel      64K   512x128  ?,?,?           ...40..,  40  8,8
  1362h  Sanyo      128K  ?        ?,?,?           ?    ?    ?    ?
  09C2h  Macronix   128K  ?        ?,?,?           ?    ?    ?    ?
```

IDの MSBはデバイスの種類、LSBは製造元を表しています。

**FLASHメモリへのアクセス**

FLASHメモリは，`0xE000000-0xE00FFFF`の SRAMエリアに配置されており，そのバス幅はアドレスが16ビット，データが8ビットに制限されています。このメモリは，8ビット単位で読み書きを行う`LDRB/STRB`オペコードでしかアクセスできません。

また、データやステータス、ビジー情報などを読み出すには、WRAMで実行されるオペコードでのみ可能です。ROM内のオペコードからは読み出せません。（書き込みにはそのような制限はありません）

**FLASH Waitstates**

WAITCNTのbit0-1が0b11の場合、最初の検出のために8waitstatesの時間がかかります

特定のデバイスタイプが検出された場合、書き込み・消去には、短い待機時間ですみ、生の読み取りにはさらに小さい待機時間で済むことがあります。

**Verify Write/Erase and Retry**

書き込み・消去の完了を知らせる信号が出ていても、変更されたメモリ領域の内容をソフトウェアで読み込んで確認することが推奨されています。

実際には、任天堂の "erase-write-verify-retry" 機能は、エラーが発生した場合に最大3回まで動作を繰り返すのが一般的です。

また、SSTデバイスのみ、"erase-write" および "erase-write-verify-retry" 機能は、消去コマンドを最大80回繰り返し、リトライが不要な場合はさらに1回、それ以外の場合はさらに6回消去コマンドを繰り返します。

**注意**

FLASH（64KB）は、ゲーム「ソニックアドバンス」などで使用されています。
