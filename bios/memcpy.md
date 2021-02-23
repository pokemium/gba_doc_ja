# メモリのコピー

CPUFastSet, CPUSet

## CPUFastSet: swi 0x0c

32**バイト**単位でのメモリコピー・メモリ埋めをおこないます。

メモリコピーは `LDMIA/STMIA [Rb]!,r2-r9` を繰り返し行うことでコピーをします。

メモリ埋めは `STMIA [Rb]!,r2-r9`を繰り返した後でLDRを1回します。

処理サイズはワード単位によって指定されています。つまり処理サイズのバイト数を4で割った値です。

GBAの場合、処理サイズは8ワード単位、つまりバイト数が32の倍数であることが望ましいです。そうでない場合は強制的に処理サイズを切り上げします。

引数:

- r0: ソースアドレス
- r1: ターゲットアドレス
- r2: サイズ/モード
  - Bit0-20: 処理サイズ(ワード単位)
  - Bit24: モード(0=Copy, 1=アドレスr0に格納された値でFill)

戻り値: なし

## CPUSet: swi 0x0b

2または4バイト単位でのメモリコピー・メモリ埋めをおこないます。

Memcopy is implemented as repeated `LDMIA/STMIA [Rb]!,r3` or `LDRH/STRH r3,[r0,r5]` instructions. 

Memfill as single LDMIA or LDRH followed by repeated `STMIA [Rb]!,r3` or `STRH r3,[r0,r5]`.

The length must be a multiple of 4 bytes (32bit mode) or 2 bytes (16bit mode). 

The (half)wordcount in r2 must be length/4 (32bit mode) or length/2 (16bit mode), ie. length in word/halfword units rather than byte units.

引数:

- r0: ソースアドレス
- r1: ターゲットアドレス
- r2: サイズ/モード/処理単位
  - Bit0-20: 処理サイズ(ワード/ハーフワード単位)
  - Bit24: モード(0=Copy, 1=アドレスr0に格納された値でFill)
  - Bit26: 処理単位(0=16bit, 1=32bit)

戻り値: なし

## 注意

CPUFastSetもCPUSetもコピー元、コピー先のどちらかがBIOS領域に到達すると特に通知せずに終了します。
