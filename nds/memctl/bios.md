# BIOS

## 4000308h - NDS7 - BIOSPROT - Bios-data-read-protection address

`BIOSPROT`レジスタはNDS7のBIOS領域の最初の数KBを`double-protect`する際にその制御を行うレジスタです。

BIOSの保護領域は2つに分けられます。1つは常にアクティブで、もう1つは`BIOSPROT`レジスタによって制御されます。

まとめると、BIOSのみがBIOS領域からの読み取りが可能で、他がBIOS領域からデータを読み出すと`0xff`となります。

 Opcodes at... | Can read from | Expl.
 0..\[BIOSPROT\]-1   | 0..3FFFh            | Double-protected (when BIOSPROT is set)
 \[BIOSPROT\]..3FFFh | \[BIOSPROT\]..3FFFh | Normal-protected (常にアクティブ)

The initial BIOSPROT setting on power-up is zero (disabled). Before starting the cartridge, the BIOS boot code sets the register to 1204h (actually 1205h, but the mis-aligned low-bit is ignored). Once when initialized, further writes to the register are ignored.

The double-protected region contains the exception vectors, some bytes of code, and the cartridge KEY1 encryption seed (about 4KBytes). As far as I know, it is impossible to unlock the memory once when it is locked, however, with some trickery, it is possible execute code before it gets locked. Also, the two THUMB opcodes at 05ECh can be used to read all memory at 0..3FFFh,

```
  05ECh  ldrb r3,[r3,12h]      ; requires incoming r3=src-12h
  05EEh  pop  r2,r4,r6,r7,r15  ; requires dummy values & THUMB retadr on stack
```

また、ほとんどのBIOS関数（`CpuSet`など）には、BIOS領域をソースアドレスとして使うのを拒否するソフトウェアベースの保護機能があります。唯一の例外は`GetCRC16`ですが、`BIOSPROT`の設定は無視することはできません。

## 注意

NDS9のBIOSには、ソフトウェアやハードウェアによる読み取り保護機能はありません。
