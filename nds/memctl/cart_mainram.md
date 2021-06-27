# カートリッジとMain RAM

## 4000204h - NDS9 - EXMEMCNT - 16bit - External Memory Control (R/W)
## 4000204h - NDS7 - EXMEMSTAT - 16bit - External Memory Status (R/W..R)

```
  0-1   32-pin GBA Slot SRAM Access Time    (0-3 = 10, 8, 6, 18 cycles)
  2-3   32-pin GBA Slot ROM 1st Access Time (0-3 = 10, 8, 6, 18 cycles)
  4     32-pin GBA Slot ROM 2nd Access Time (0-1 = 6, 4 cycles)
  5-6   32-pin GBA Slot PHI-pin out   (0-3 = Low, 4.19MHz, 8.38MHz, 16.76MHz)
  7     32-pin GBA Slot Access Rights     (0=ARM9, 1=ARM7)
  8-10  不使用 (常に0)
  11    17-pin NDS Slot Access Rights     (0=ARM9, 1=ARM7)
  12    不使用 (常に0)
  13    NDS:Always set?  ;set/tested by DSi bootcode: Main RAM enable, CE2 pin?
  14    Main Memory Interface Mode Switch (0=Async/GBA/Reserved, 1=Synchronous)
  15    Main Memory Access Priority       (0=ARM9 Priority, 1=ARM7 Priority)
```

bit0-6はNDS9とNDS7の両方で変更可能です。これらのビットを変更すると、ローカルのEXMEMレジスタのみに影響し、他のCPUのレジスタには影響しません。

bit7-15はNDS9のみ変更可能で、これらのビットを変更すると、両方のEXMEMレジスタに影響を与えます。つまり、NDS9とNDS7の両方が現在のNDS9の設定を読み取ることができます。

bit14を0にするのはGBAモードのためのようですが、このbitへの書き込みは無視されている模様？

## GBA Slot (8000000h-AFFFFFFh)

GBAスロットはEXMEMCNTのbit7を通して、ARM9かARM7に対してマッピングされます。

マッピングされたCPUにとって、0x0800_0000-0x09FF_FFFFのメモリが`GBA ROM`領域、0x0A00_0000-0x0AFF_FFFFのメモリが`GBA SRAM`領域となります（64KBごとに繰り返されます）。

もしGBAスロットにカートリッジがない場合は、ROM/SRAM領域は次の値で埋められます。

- SRAM領域: 0xFF
- ROM領域: increasing 16bit valuesである、`Addr/2`(ROMへのアクセス時間に応じて次のように変動)

```
  6 clks   --> returns "Addr/2"
  8 clks   --> returns "Addr/2"
  10 clks  --> returns "Addr/2 OR FE08h" (or similar garbage)
  18 clks  --> returns "FFFFh" (High-Z)
```

マッピングされなかったCPUから見ると、`0x0800_0000-0x0AFF_FFFF`は0埋めされており、`Digimon Story: Super Xros Wars`のようにこの仕様をついてバグを生じさせるゲームもあるようです。