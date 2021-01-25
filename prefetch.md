# カートリッジのプリフェッチ

カートリッジのプリフェッチは WAITCNTレジスタで有効にすることができます。プリフェッチバッファが有効になっている場合、GBAはCPUがバスを使用していないタイミングにカートリッジROMから（もしあれば）オペコードを読み出そうとします。

CPU が既にバッファに保存されているデータを要求した場合、メモリアクセスは 0Waits で実行されます。プリフェッチバッファには、最大8つの16ビット値が格納されます。

## カートリッジROMのオペコード

プリフェッチ機能はカートリッジROMのオペコードに対してのみ有効です。RAMやBIOSに配置されたオペコードはプリフェッチ機能を使うことはできません。

## 有効な場面

プリフェッチは次のようなオペコードで発生します。

- Iサイクルで実行されるオペコードでR15を変更しないもの (shift/rotate register-by-register, load opcodes (ldr,ldm,pop,swp), multiply opcodes)
- ロードストア命令 (ldr,str,ldm,stm,etc.)

## プリフェッチ無効バグ

When Prefetch is disabled, the Prefetch Disable Bug will occur for all "Opcodes in GamePak ROM with Internal Cycles which do not change R15" for those opcodes, the bug changes the opcode fetch time from 1S to 1N.

Note: Affected opcodes (with I cycles) are: Shift/rotate register-by-register opcodes, multiply opcodes, and load opcodes (ldr,ldm,pop,swp).
