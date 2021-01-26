# gba_doc_ja

## コンテンツ一覧

### GBA

- [仕様](spec.md)
- [メモリマップ](memory.md)
- [DMA転送](dma.md)
- [割り込み](interrupt.md)
- [システム制御](system.md)
- [タイマー](timer.md)
- [キー入力](keypad.md)
- [カートリッジのプリフェッチ](prefetch.md)

### カートリッジ

- [カートリッジヘッダ](./cartridge/header.md)
- [カートリッジROM](./cartridge/rom.md)

### CPU

- [レジスタ](./arm7tdmi/register.md)
- [命令一覧](arm7tdmi/instruction.md)
  - [ALU](./arm7tdmi/alu.md)
  - [分岐命令](arm7tdmi/branch.md)
  - [乗算](./arm7tdmi/multiply.md)
  - [プロセッサ状態](./arm7tdmi/psr.md)
- [条件](arm7tdmi/cond.md)
- [例外](arm7tdmi/exception.md)

### 画面描画

- [LCD制御レジスタ](./lcd/lcd_control.md)
- [BGモード](./lcd/bg_mode.md)
- [BG制御レジスタ](./lcd/bg_control.md)
- [BGスクロール](./lcd/bg_scrolling.md)
- [BG伸縮回転](./lcd/bg_rotation_scaling.md)
- [ウィンドウ](./lcd/window.md)
- [割り込み](./lcd/interrupt_and_status.md)

## References

- [GBATEK](https://web.archive.org/web/20210108175702/https://problemkaputt.de/gbatek.htm)
- [GBA develop Wiki](http://akkera102.sakura.ne.jp/gbadev/)
- [情報システム設計演習－携帯型ゲームプログラミング－](http://jaco.ec.t.kanazawa-u.ac.jp/edu/GBA/)
- [ultimate-tutorial-2](https://tutorial.feuniverse.us/)
