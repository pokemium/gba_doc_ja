# カートリッジROM

## ROM Size

基本的に8MBや16MBが主流です。

FZeroとスーパーマリオアドバンスは4MBです。

## ROM Waitstates

GBAは起動時にカートリッジのWaitStateをN=4, S=2に設定しプリフェッチは無効にします。これらの設定は[WAITCNT](../system.md#0x0400_0204---waitcnt---waitstate制御レジスタ-rw)を介して変更可能です。

例えば、FZeroとスーパーマリオアドバンスはWaitStateをN=3,S=1にしプリフェッチを有効にしています。

プリフェッチ機能を有効にしてWaitStateを短く設定すると、ROMから多くのデータとオペコードを高速に読み取ることができますが、消費電力が増加することに注意してください。

## ROMチップ

Because of how 24bit addresses are squeezed through the Gampak bus, the cartridge must include a circuit that latches the lower 16 address bits on non-sequential access, and that increments these bits on sequential access. 

Nintendo includes this circuit directly in the ROM chip.

Also, the ROM must have 16bit data bus (or a circuit which converts two 8bit data units into one 16bit unit - by not exceeding the waitstate timings).
