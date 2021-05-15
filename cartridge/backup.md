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

**When to use the above Workaround**

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
セーブ機能の寿命: 100,000回の読み書き
```

**Addressing and Waitstates**

The eeprom is connected to Bit0 of the data bus, and to the upper 1 bit (or upper 17 bits in case of large 32MB ROM) of the cartridge ROM address bus, communication with the chip takes place serially.
The eeprom must be used with 8 waitstates (set WAITCNT=X3XXh; 8,8 clks in WS2 area), the eeprom can be then addressed at DFFFF00h..DFFFFFFh.
Respectively, with eeprom, ROM is restricted to 8000000h-9FFFeFFh (max. 1FFFF00h bytes = 32MB minus 256 bytes). On carts with 16MB or smaller ROM, eeprom can be alternately accessed anywhere at D000000h-DFFFFFFh.

**Data and Address Width**

Data can be read from (or written to) the EEPROM in units of 64bits (8 bytes). Writing automatically erases the old 64bits of data. Addressing works in units of 64bits respectively, that is, for 512 Bytes EEPROMS: an address range of 0-3Fh, 6bit bus width; and for 8KByte EEPROMs: a range of 0-3FFh, 14bit bus width (only the lower 10 address bits are used, upper 4 bits should be zero).

**Set Address (For Reading)**

Prepare the following bitstream in memory:

```
  2 bits "11" (Read Request)
  n bits eeprom address (MSB first, 6 or 14 bits, depending on EEPROM)
  1 bit "0"
```

Then transfer the stream to eeprom by using DMA.

**Read Data**

Read a stream of 68 bits from EEPROM by using DMA, then decipher the received data as follows:

```
  4 bits - ignore these
 64 bits - data (conventionally MSB first)
```

**Write Data to Address**

Prepare the following bitstream in memory, then transfer the stream to eeprom by using DMA, it'll take ca. 108368 clock cycles (ca. 6.5ms) until the old data is erased and new data is programmed.

```
  2 bits "10" (Write Request)
  n bits eeprom address (MSB first, 6 or 14 bits, depending on EEPROM)
 64 bits data (conventionally MSB first)
  1 bit "0"
```

After the DMA, keep reading from the chip, by normal LDRH [DFFFF00h], until Bit 0 of the returned data becomes "1" (Ready). To prevent your program from locking up in case of malfunction, generate a timeout if the chip does not reply after 10ms or longer.

**Using DMA**

Transferring a bitstreams to/from the EEPROM must be done via DMA3 (manual transfers via LDRH/STRH won't work; probably because they don't keep /CS=LOW and A23=HIGH throughout the transfer).
For using DMA, a buffer in memory must be used (that buffer would be typically allocated temporarily on stack, one halfword for each bit, bit1-15 of the halfwords are don't care, only bit0 is used).
The buffer must be transfered as a whole to/from EEPROM by using DMA3 (DMA0-2 can't access external memory), use 16bit transfer mode, both source and destination address incrementing (ie. DMA3CNT=80000000h+length).
DMA channels of higher priority should be disabled during the transfer (ie. H/V-Blank or Sound FIFO DMAs). And, of course any interrupts that might mess with DMA registers should be disabled.

**Pin-Outs**

The EEPROM chips are having only 8 pins, these are connected, Pin 1..8, to ROMCS, RD, WR, AD0, GND, GND, A23, VDD of the GamePak bus. Carts with 32MB ROM must have A7..A22 logically ANDed with A23.

**注意**

There seems to be no autodection mechanism, so that a hardcoded bus width must be used.

## GBA Cart Backup Flash ROM

```
64 KB - 512Kbits Flash ROM - Lifetime: 10,000 writes per sector
128 KB - 1Mbit Flash ROM - Lifetime: ??? writes per sector
```

**Chip Identification (all device types)**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=90h  (enter ID mode)
  dev=[E000001h], man=[E000000h]                  (get device & manufacturer)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=F0h  (terminate ID mode)
```

Used to detect the type (and presence) of FLASH chips. See Device Types below.

**Reading Data Bytes (all device types)**

```
  dat=[E00xxxxh]                                  (read byte from address xxxx)
```

**Erase Entire Chip (all device types)**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=80h  (erase command)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=10h  (erase entire chip)
  wait until [E000000h]=FFh (or timeout)
```

Erases all memory in chip, erased memory is FFh-filled.

**Erase 4Kbyte Sector (all device types, except Atmel)**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=80h  (erase command)
  [E005555h]=AAh, [E002AAAh]=55h, [E00n000h]=30h  (erase sector n)
  wait until [E00n000h]=FFh (or timeout)
```

Erases memory at E00n000h..E00nFFFh, erased memory is FFh-filled.

**Erase-and-Write 128 Bytes Sector (only Atmel devices)**

```
  old=IME, IME=0                                  (disable interrupts)
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=A0h  (erase/write sector command)
  [E00xxxxh+00h..7Fh]=dat[00h..7Fh]               (write 128 bytes)
  IME=old                                         (restore old IME state)
  wait until [E00xxxxh+7Fh]=dat[7Fh] (or timeout)
```

Interrupts (and DMAs) should be disabled during command/write phase. Target address must be a multiple of 80h.

**Write Single Data Byte (all device types, except Atmel)**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=A0h  (write byte command)
  [E00xxxxh]=dat                                  (write byte to address xxxx)
  wait until [E00xxxxh]=dat (or timeout)
```

The target memory location must have been previously erased.

**Terminate Command after Timeout (only Macronix devices, ID=1CC2h)**

```
  [E005555h]=F0h                            (force end of write/erase command)
```

Use if timeout occurred during "wait until" periods, for Macronix devices only.

**Bank Switching (devices bigger than 64K only)**

```
  [E005555h]=AAh, [E002AAAh]=55h, [E005555h]=B0h  (select bank command)
  [E000000h]=bnk                                  (write bank number 0..1)
```

Specifies 64K bank number for read/write/erase operations.

Required because gamepak flash/sram addressbus is limited to 16bit width.

**Device Types**
Nintendo puts different FLASH chips in commercial game cartridges. Developers should thus detect & support all chip types. For Atmel chips it'd be recommended to simulate 4K sectors by software, though reportedly Nintendo doesn't use Atmel chips in newer games anymore. Also mind that different timings should not disturb compatibility and performance.

```
  ID     Name       Size  Sectors  AverageTimings  Timeouts/ms   Waits
  D4BFh  SST        64K   16x4K    20us?,?,?       10,  40, 200  3,2
  1CC2h  Macronix   64K   16x4K    ?,?,?           10,2000,2000  8,3
  1B32h  Panasonic  64K   16x4K    ?,?,?           10, 500, 500  4,2
  3D1Fh  Atmel      64K   512x128  ?,?,?           ...40..,  40  8,8
  1362h  Sanyo      128K  ?        ?,?,?           ?    ?    ?    ?
  09C2h  Macronix   128K  ?        ?,?,?           ?    ?    ?    ?
```

Identification Codes MSB=Device Type, LSB=Manufacturer.

Size in bytes, and numbers of sectors * sector size in bytes.

Average medium Write, Erase Sector, Erase Chips timings are unknown?

Timeouts in milliseconds for Write, Erase Sector, Erase Chips.

Waitstates for Writes, and Reads in clock cycles.

**Accessing FLASH Memory**
FLASH memory is located in the "SRAM" area at E000000h..E00FFFFh, which is restricted to 16bit address and 8bit data buswidths. Respectively, the memory can be accessed only by 8bit read/write LDRB/STRB opcodes.

Also, reading anything (data or status/busy information) can be done only by opcodes executed in WRAM (not from opcodes in ROM) (there's no such restriction for writing).

**FLASH Waitstates**

Use 8 clk waitstates for initial detection (WAITCNT Bits 0,1 both set). After detection of certain device types smaller wait values may be used for write/erase, and even smaller wait values for raw reading, see Device Types table.

In practice, games seem to use smaller values only for write/erase (even though those operations are slow anyways), whilst raw reads are always done at 8 clk waits (even though reads could actually benefit slightly from smaller wait values).

**Verify Write/Erase and Retry**

Even though device signalizes the completion of write/erase operations, it'd be recommended to read/confirm the content of the changed memory area by software. In practice, Nintendo's "erase-write-verify-retry" function typically repeats the operation up to three times in case of errors.

Also, for SST devices only, the "erase-write" and "erase-write-verify-retry" functions repeat the erase command up to 80 times, additionally followed by one further erase command if no retries were needed, otherwise followed by six further erase commands.

**注意**

FLASH (64Kbytes) is used by the game Sonic Advance, and possibly others.
