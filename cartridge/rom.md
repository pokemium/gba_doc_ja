# カートリッジROM

## ROM Size

基本的に8MBや16MBは主流です。

FZeroとスーパーマリオアドバンスは4MBです。

## ROM Waitstates

GBAは起動時にカートリッジのWaitStateをN=4, S=2にしプリフェッチは無効にします。

WAITCNTを変更することで、これらの設定を変えることが可能です。FZeroとスーパーマリオアドバンスはWaitStateをN=3,S=1にしプリフェッチを有効にしています。

サードパーティ製のフラッシュカードは、これらの設定では動作が不安定になると報告されています。また、プリフェッチと短いウェイトステートは、ROMからより多くのデータとオペコードを読み取ることができ、その時間も少ないですが、欠点は、それが消費電力を増加させることです。

## ROMチップ

Because of how 24bit addresses are squeezed through the Gampak bus, the cartridge must include a circuit that latches the lower 16 address bits on non-sequential access, and that increments these bits on sequential access. 

Nintendo includes this circuit directly in the ROM chip.

Also, the ROM must have 16bit data bus (or a circuit which converts two 8bit data units into one 16bit unit - by not exceeding the waitstate timings).
